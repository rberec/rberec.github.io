---
layout: post
title: Test
author: rberec
category:  random numbers, thrust, transform_reduce, cuda, monte carlo, sobol, mersenne twister
---

We have already shown before how to set up *CUDA* in both Windows and Linux environment and how to create a simple project. Today we will demonstrate how easy it is to perform a simple Monte Carlo simulation using [Thrust](http://thrust.github.io/) library. We will do so by estimating $\pi$ with random numbers.

From the project's homepage we see that
>Thrust is a parallel algorithms library which resembles the C++ Standard Template Library (STL). Thrust's high-level interface greatly enhances programmer productivity while enabling performance portability between GPUs and multicore CPUs.

And this is very true. When you use *Thrust* it is almost identical to using the *STL*. While it might not be the fastest GPU implementation for your particular problem, it is easy to use and provides nice tools for quick development. It takes care of many things behind the scene (`cudaMalloc`, `cudaFree`, etc.) and makes the code cleaner and pleasant to read.

We will illustrate the $\pi$ estimation with three different approaches. Firstly, we will implement the algorithm outlined in [this presentation](http://www.nvidia.com/docs/IO/116711/sc11-montecarlo.pdf). Secondly, we will implement a modified code from *Thrust* examples (with *CuRand* in kernel). Finally, we will implement the standard *CuRand* example utilizing *Thrust*.

## Approach 1
The first version of $\pi$ estimation is using random number generator implemented in *Thrust* library. The code is very strightforward and has three main components. The functor `randomPoint` responsible for generating pair of random numbers is

{% highlight c++ %}
struct randomPoint{
    __device__ float2 operator() (const unsigned int n) {
        thrust::default_random_engine rng;
        rng.discard(2 * n);
        return make_float2(
            (float)rng() / thrust::default_random_engine::max,
            (float)rng() / thrust::default_random_engine::max);
    }
};
{% endhighlight %}

Note, that the original presentation is missing and important line of code `rng.discard(2 * n)`. We would geerate the same random numbers again and again without it.

Following functor is responsible for deciding whether the point is inside a unit circle or not

{% highlight c++ %}
struct insideCircle {
    __device__ unsigned int operator() (float2 p) const {
        return (sqrtf(p.x*p.x + p.y*p.y) < 1.0f) ? 1 : 0;
    }
};
{% endhighlight %}

And the final piece of code for gluing these parts together is

{% highlight c++ %}
// DEVICE: Generate random points within a unit square
thrust::device_vector<float2> d_random(N);
thrust::counting_iterator<unsigned int> d_indexSequence(0);
thrust::transform(d_indexSequence, d_indexSequence + N,
			      d_random.begin(), randomPoint());

// DEVICE: Flags to mark points lying inside or outside the circle
thrust::device_vector<unsigned int> d_inside(N);

// DEVICE: Function evaluation. Mark points as inside or outside and aggregate
unsigned int total = thrust::transform_reduce(
	d_random.begin(),
    d_random.end(),
    insideCircle(),
    0,
    thrust::plus<unsigned int>());

return 4.0f * (float)total / (float)N;
{% endhighlight %}

The original presentation uses `transform` and `reduce` functions separately. For a small performance improvement we combined these two functions into one - `transform_reduce`.

## Approach 2
The second version follows an example from *Thrust* library but instead of using *Thrust* random number generator we use *CuRand* random number generator in kernel function.

The heavy-weighting is done by following functor

{% highlight c++ %}
struct estimatePiHelper :
    public thrust::unary_function<unsigned int, float>
{
    __device__
        float operator()(unsigned int thread_id)
    {
        float sum(0);
        unsigned int N(100000); // samples per thread

        unsigned int seed = thread_id;

        curandState s;

        // seed a random number generator
        curand_init(seed, 0, 0, &s);

        // take N samples in a quarter circle
        for (unsigned int i = 0; i < N; ++i)
        {
            // draw a sample from the unit square
            float x = curand_uniform(&s);
            float y = curand_uniform(&s);

            // measure distance from the origin
            float dist = sqrtf(x*x + y*y);

            // add 1.0f if (u0,u1) is inside the quarter circle
            if (dist <= 1.0f)
                sum += 1.0f;
        }

        // multiply by 4 to get the area of the whole circle
        sum *= 4.0f;

        // divide by N
        return sum / N;
    }
};
{% endhighlight %}

The main difference between first and second approach is that here we generate *100,000* random numbers within each thread, i.e. we perform more computations within each kernel function.

Following is code which calls the code above and estimates $\pi$

{% highlight c++ %}
float estimate = thrust::transform_reduce(
        thrust::counting_iterator<unsigned int>(0),
        thrust::counting_iterator<unsigned int>(N),
        estimatePiHelper(),
        0.0f,
        thrust::plus<float>());

return estimate / N;
{% endhighlight %}

The only downside is that we found it rather cumbersome to specify a different random number generator, e.g. Sobol or Mersenne Twister.

## Approach 3
The last version utilizes *CuRand* function called from host rather than device. The helper functor responsible for deciding whether point is within unit circle is

{% highlight c++ %}
struct insideCircle {
    __device__ unsigned int operator() (thrust::tuple<float, float> p) const {
        float x = thrust::get<0>(p);
        float y = thrust::get<1>(p);
        return (sqrtf(x*x + y*y) < 1.0f) ? 1 : 0;
    }
};
{% endhighlight %}

The core part of code of this version is

{% highlight c++ %}
curandGenerator_t qrng;
thrust::device_vector<float> d_points(2 * N);

// Set type of random number generator
curandCreateGenerator(&qrng, CURAND_RNG_QUASI_SOBOL32);
//curandCreateGenerator(&qrng, CURAND_RNG_PSEUDO_DEFAULT);

// Set dimension of random number generator
curandSetQuasiRandomGeneratorDimensions(qrng, 2);

// Set default ordering
curandSetGeneratorOrdering(qrng, CURAND_ORDERING_QUASI_DEFAULT);

// Generate N numbers in 2 dimensions
curandGenerateUniform(qrng, thrust::raw_pointer_cast(&d_points[0]), 2 * N);

unsigned int total = thrust::transform_reduce(
	thrust::make_zip_iterator(
    	thrust::make_tuple(
        	d_points.begin(),
            d_points.begin() + N + 1)),
	thrust::make_zip_iterator(
    	thrust::make_tuple(
        	d_points.begin() + N + 1,
        	d_points.end())),
	insideCircle(),
	0,
	thrust::plus<unsigned int>()
);

// Destroy random number generator
curandDestroyGenerator(qrng);

return 4.0f * (float)total / (float)N;
{% endhighlight %}

Note, that we use Sobol random number generator in this case. Also note, that we use structure of arrays (SoA) rather than array of structures (AoS) as described in [this presentation](http://on-demand.gputechconf.com/gtc-express/2011/presentations/introductiontothrust.pdf).

We hope you enjoyed this post and do not forget to leave us a comment. Thank you.
