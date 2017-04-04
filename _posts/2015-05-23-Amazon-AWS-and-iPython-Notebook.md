---
layout: post
title: Amazon AWS and iPython Notebook
author: rberec
---

Amazon Web Services ([AWS](http://aws.amazon.com/)) offer easy-to-use and cheap cloud computing on demand - the elastic cloud computing ([EC2](http://aws.amazon.com/ec2/)). I've been using it for several of my projects and I found that using spot instances is an extremely useful and effective way to test/experiment with your ideas.

In this post, I will show you how to set up iPython Notebook which you can then access from your web-browser. Please note, that I will be installing nVidia's drivers and *CUDA* so that you can use *Theano* to speed up your calculations using *GPU*.

# Installing Ubuntu Packages

After you start your GPU large instance and connect to it using SSH just install these packages

{% highlight bash %}
sudo apt-get -y install gfortran
sudo apt-get -y install libblas-dev libatlas-dev liblapack-dev
sudo apt-get -y install freetype*
sudo apt-get -y install pkg-config 
sudo apt-get -y install gcc
sudo apt-get -y install g++
sudo apt-get -y install build-essential
sudo apt-get -y install git
sudo apt-get -y install wget
sudo apt-get -y install linux-image-generic
sudo apt-get -y install cmake

# install python stuff
sudo apt-get -y install python-dev python-pip

# install virtual environments
sudo pip install virtualenvwrapper

# install CUDA
sudo wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1404/x86_64/cuda-repo-ubuntu1404_6.5-14_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1404_6.5-14_amd64.deb
sudo apt-get update 
sudo apt-get -y install cuda
{% endhighlight %}

Then, we can add CUDA's lib and bin to bashrc

{% highlight bash %}
echo -e "\nexport PATH=/usr/local/cuda-6.5/bin:$PATH\n\nexport LD_LIBRARY_PATH=/usr/local/cuda-6.5/lib64" >> .bashrc
{% endhighlight %}

Reboot

{% highlight bash %}
sudo reboot
{% endhighlight %}

And then test if CUDA works as it is supposed to be

{% highlight bash %}
#test CUDA
cuda-install-samples-6.5.sh ~/
cd NVIDIA_CUDA-6.5_Samples/1_Utilities/deviceQuery
make
./deviceQuery
{% endhighlight %}

# Setting-up Python

I prefer to use Python's virtual environments to keep each python environment separate.

{% highlight bash %}
mkdir ~/.virtualenvs
cd .virtualenvs/

mkvirtualenv --no-site-packages quantTest
workon quantTest

pip install nose
pip install numpy
pip install scipy
pip install pygments
pip install mysql-python
pip install matplotlib
pip install pandas

pip install feedparser
pip install lxml
pip install beautifulsoup4
pip install requests
pip install cython

pip install pyzmq

pip install tornado
pip install jinja2
pip install PySide

pip install ipython

pip install scikit-learn
pip install scikit-image

pip install joblib
pip install pyyaml
pip install --upgrade --no-deps git+git://github.com/Theano/Theano.git
pip install --upgrade --no-deps git+git://github.com/benanne/Lasagne.git
pip install --upgrade --no-deps git+git://github.com/dnouri/nolearn.git

git clone git://github.com/lisa-lab/pylearn2.git
cd pylearn2
python setup.py develop
{% endhighlight %}

# Setting-up iPython

First of all, create a password in python

{% highlight bash %}
from IPython.lib import passwd
passwd()
Enter password:
Verify password:
sha1:def70021a545:7b799f5ecfXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
{% endhighlight %}

Then, create a certificate for notebook

{% highlight bash %}
mkdir .ssh
cd .ssh
openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem
{% endhighlight %}

Followed by creating a iPython profile

{% highlight bash %}
ipython profile create quantServer
{% endhighlight %}

Then, modify the profile `.ipython/profile_quantServer`

{% highlight python %}
c = get_config()

# Kernel config
c.IPKernelApp.pylab = 'inline'  # if you want plotting support always

# Notebook config
c.NotebookApp.certfile = u'/home/ubuntu/.ssh/mycert.pem'
c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.password = u'sha1:def70021a545:7b799f5ecfXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'
# Put it on a fixed port
c.NotebookApp.port = 9999
{% endhighlight %}

# Run iPython Notebook

After all this, we can finally run iPython Notebook on our server by simply calling

{% highlight bash %}
ipython notebook --profile=quantServer
{% endhighlight %}

And that's it! After this, you can connect to your server on port 9999 and enjoy iPython. Once finished, you can create an amazon image (AMI) and use it in future to set up your spot instance and avoid running all these scripts over and over again.

