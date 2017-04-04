---
layout: post
title: HJM No-Arbitrage Drift Condition
author: rberec
---

In late 1980s David Heath, Robert Jarrow and Andrew Morton worked on interest rate framework which would unify the theory for valuing contingent claims under a stochastic term structure of interest rates. The paper, *Bond pricing and the term structure of interest rates: A new methodology for contingent claims valuation*, was published in 1992 in Econometrica.

In this post we will quickly describe the model and derive the no-arbitrage condition in the risk-neutral measure in a single economy.

## Definitions and Relationships

Let $f(t,T)$ be a *nominal instantaneous forward rate* at time $$t$$ with expiry $$T$$, $$Z(t,T)$$ be a *nominal zero coupon bond price* at time $$t$$ with maturity $$T$$. The relationship between the two is

$$ Z(t,T) = \exp \left( \int_{t}^{T} f(t,s) {d} s \right) \quad \textrm{for all } T \in [0,\tau],t \in [0,T], $$

where $$\tau$$ is the ending point in the trading interval. The *nominal spot rate*  at time $$t$$, $$r(t)$$, is the nominal instantaneous forward rate at time $$t$$ for date $$t$$

$$ r(t) = f(t,t) \quad \textrm{for all } t \in [0,\tau]. $$

The *risk-free nominal money-market account* (accumulation factor), $$B(t)$$ initialized at time 0 with a one unit of currency investment is

$$ B(t) = \exp \left( \int_{0}^{t} r(s) {d} s \right) \quad \textrm{for all } t \in [0,\tau] $$

## No-Arbitrage Conditions

Under the nominal risk neutral measure the dynamics of the nominal risk-free zero coupon bond is

$$ \frac{dZ(t,T)}{Z(t)} = r(t)dt + \sigma_Z(t,T)'dW(t), $$

where $$W(t)$$ is a $$d$$-dimensional standard $$Q$$-Brownian motion. Assuming following dynamics for the $$f(t,T)$$, the question now is to identify the drift term $$\mu(t,T)$$

$$ df(t,T) = \mu(t,T)dt + \sigma_f(t,T)'dW(t) $$

Let $$X(t) = -\int_t^T f(t,s) ds$$, then

\begin{align}
dX(t) &= f(t,t)dt -\int_t^T df(t,s) ds \\\\
&= r(t)dt - \int^T\_t \left[ \mu(t,s)dt + \sigma\_f(t,s)'dW(t) \right] ds
\end{align}

Next, by applying standard and stochastic Fubini theorem we get

\begin{align}
dX(t) &= r(t) dt - \int^T\_t \mu(t,s) ds\ dt - \int^T\_t \sigma\_f(t,s)' ds\ dW(t) \\\\
\\\\
&= r(t)dt - \alpha(t,T)dt - \sigma\_B(t,T)' dW(t)
\end{align} 

where $$\alpha(t,T) = \int^T\_t \mu(t,s) ds$$ and $$\sigma\_B(t,T)' = \int^T\_t \sigma\_f(t,s)' ds$$. We are almost there, now we can put all things together and using the very first equation and Ito's lemma

\begin{align}
 d Z(t,T) &= \exp ( X(t) ) dX(t) + \frac{1}{2} \exp ( X(t) ) (dX(t))^2 \\\\
\\\\
 \frac{dZ(t,T)}{Z(t,T)}&= \left[ r(t) - \alpha(t,T) + \frac{1}{2} \sigma\_B(t,T)' \sigma\_B(t,T) \right] dt - \sigma\_B(t,T)' dW(t)
\end{align}

Please note that under the risk-neutral measure the drift term for $$Z(t,T)$$ must be equal to $$r(t) Z(t,T)$$. We conclude following no-arbitrage condition which must hold for all $$T$$

$$ \alpha(t,T) = \frac{1}{2} \sigma\_B(t,T)' \sigma\_B(t,T) $$

By differentiating both sides with respect to $$T$$ we obtain

\begin{align}
\mu(t,T) &= \sigma\_f(t,T)'\sigma\_B(t,T) \\
\\
\mu(t,T) &= \sigma\_f(t,T)' \int^T\_t \sigma\_f(t,s) ds
\end{align}

So the final stochastic differential equation for the instantaneous forward rate under the risk-free measure is of the following form

$$ df(t,T) = \left[ \sigma_f(t,T)' \int^T_t \sigma_f(t,s)ds \right] dt + \sigma_f(t,T)'dW(t) $$


