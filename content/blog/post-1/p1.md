+++
title = "Evidence Lower Bound as Estimating Normalization Constant"
date = "2023-04-13T17:12:35-05:00"
description = "We re-examine the Evidence Lower Bound as a form of Monte Carlo estimation for normalizing a distribution."

tags = []
math = true
draft = true
+++

I am quite forgetful and it doesn't help that many Bayesian methods look too similar, making it easy to get confused. This is especially true when said algorithms use make slightly different assumptions of the distributions accessible, what is conditioned on what, etc.

The ELBO is just one of those formulas where you can swap a couple of variables here and there and get another similar looking formula describing something completely different.[^1] Recall the ELBO bound is defined as
$$p(y)=E_q\left[\log\frac{p(x,y)}{q(x)}\right]+D(q(x)||p(x|y)) \label{elbo}$$

where $y$ are observations and $x$ are latents. Recall that $p_{x|y}$ is interesting because it represents some posterior model which in general is hard to calculate. 

The way ELBO is usually presented [^2] is via an argument of bounding the *evidence*, $p(y)$, by some lower bound defined between the target distribution $p_{x|y}$ and an approximate distribution $q$. By minimizing $D(q || p(z|x))$, we get better lower bounds on $p_{x|y}$. The presentation of the proof is succinct and to the point, but to be honest I always forget the setup and by proxy the ELBO formula. 

Instead I want to share another perspective on what the ELBO derivation is doing, by solving a slightly (not really) different problem. The proof will end up being exactly the same, but with a different motivation.

#### Estimating Normalization Constant

Given a non-negative function $\tilde{p}(x)$, it is always possible to turn it into a distribution by summing (integrating) over the space to obtain a constant $Z$. The constatn $Z$ is refered to as the normalization constant.[^3] The distribution is then defined as $p(x)=\tilde{p}(x)/Z$. The problem is that finding $Z$ is often intractable in high-dimensional settings. For example, if $x\in \mathbb{R}^3$, then three sums, one for each axis, are required. The way we will estimate $Z$ is, you guessed it, by lower bounding it. The better the lower bound $\hat{Z}$ is, the closer our "true distribution" $\tilde{p}/\hat{Z}$ will be to a true distribution. The first step of the proof is the most crucial to remember, but thankfully the most obvious:

\begin{align*}
Z = \frac{\tilde{p}(x)}{p(x)}
\end{align*}
Now we repeat the ELBO derviation. Taking the log on both sides and taking an expectation with respect to $q$ (recall $q$ is the approximate distribution),
\begin{align*}
E_q\left[\log Z\right] = E_q\left[\log \frac{\tilde{p}(x)}{p(x)}\right]
\end{align*}
Since $Z$ is constant, the expectation on the left side disappears. Now we massage the right side by adding and subtracting a $\log q(x)$ term to extract out a divergence term.
\begin{align}
\log{Z} &= E_q\left[\log\frac{\tilde{p}(x)q(x)}{p(x)q(x)}\right] \\
&=E_q\left[\log\frac{\tilde{p}(x)}{q(x)}\right]+E_q\left[\log\frac{q(x)}{p(x)}\right]\\
&=E_q\left[\log\frac{\tilde{p}(x)}{q(x)}\right]+D(q(x)||p(x)) \label{Z}
\end{align}

Note that this is virtually the same lower bound. Minimizing the divergence term means $q$ gets closer to $p$ and thus the first term gets closer to $\log Z$. 

Look at the form of \eqref{Z} and compare it with Equation \eqref{elbo}. Now the connection is more apparent. In the first setting, we have a distribution $p(x,y)$ but wish to find $p(x|y)$. How do we do it? Well we need to apply Bayes' rule and divide by $p(y)$. But what is $p(y)$ exactly? It's the normalization constant of $p(x,y)$! By replacing $\tilde{p}$ with $p(x,y)$, $p(x)$ with $p(x|y)$, and $Z=p(y)$ in Equation \eqref{Z}, we obtain Equation \eqref{elbo}.

With this slight perspective change, we see that there is nothing really special about using $p(x,y)$ to start of with. If we have *any* constant multiple of $p(x|y)$ to start, then we can estimate the normalization constant to reobtain $p(x|y)$. Sure, it is typical to have $p(x,y)$ on hand, but it is not the only scenario. Imagine going somewhat in reverse and defining $p(x|y)$ based of an energy function $E(x,y)$. We can define a distribution $\tilde{p}(x|y)=E(x,y)$ and claim that $p(x|y)\propto E(x,y)$. Repeating the same steps above gives us
\begin{align*}
Z_E(y)=E_q\left[\log\frac{E(x,y)}{q(x;y)}\right]+D(q(x;y)||p(x|y))
\end{align*}
Now we can happily use this formula to approximate a good $q$, and we didn't have to know a distribution $p(x,y)$ to begin with.


Perhaps ELBO should be instead called the NCLO - the normalization constant lower bound 🤔.

[^1]: Some changes to the modeling assumptions do not fundamentally change the ELBO. For example adding another latent doesn't change what ELBO can't already handle.
[^2]: Refer to [ELBO reference](https://www.cs.princeton.edu/courses/archive/fall11/cos597C/lectures/variational-inference-i.pdf). Page 4 for derivation.
[^3]: Sometimes referred to as "partition function" or "marginal likelihood."
