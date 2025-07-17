---
layout: distill
title: outlier rejection in nonlinear least squares
description: an aesthetic derivation
tags: 
giscus_comments: true
date: 2025-03-19
featured: true
mermaid:
  enabled: true
  zoomable: true
code_diff: true
map: true
chart:
  chartjs: true
  echarts: true
  vega_lite: true
tikzjax: true
typograms: true

authors:
  # - name: Albert Einstein
  #   url: "https://en.wikipedia.org/wiki/Albert_Einstein"
  #   affiliations:
  #     name: IAS, Princeton
  # - name: Boris Podolsky
  #   url: "https://en.wikipedia.org/wiki/Boris_Podolsky"
  #   affiliations:
  #     name: IAS, Princeton
  # - name: Nathan Rosen
  #   url: "https://en.wikipedia.org/wiki/Nathan_Rosen"
  #   affiliations:
  #     name: IAS, Princeton

bibliography: 2025-05-02-robust.bib

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---
<!-- Physical systems are continuous. However, we work with discrete-time models that are amenable
to digital computers. Therefore, we create continuous models of the physical systems that we then discretize. 

This becomes particularly interesting in the case of sensor noise, since there is a randomness that needs to be modeled. Typically we use white noise models. However, while the discrete-time white noise model is fairly straightforward (a Gaussian distribution for every noise sample), the continuous-time model is really not, and deriving the transition from continuous to discrete time for white noise is nuanced. 

A key aspect in this dilemma is that it's hard to decouple the dynamics of the system we are considering from the random process noise properties. For instance, a gyroscope may have some continuous noise characteristics of its own. 
Yet, when we consider discrete-time properties, we need to look at the effect of time, and thus the overall system dynamics. This is only really tractable for linear systems (and thus, linearizations). 

The post is split into three parts. 
The first part on random processes essentially covers terminology. 
We go over why the term power spectral density is used interchangeably with continuous-time covariance. 
The second part covers the "Linearize then Discretize" method, which obtains discretizations by lumping in the effects of system dynamics with effect of random process noise. 
The first and second parts are essentially 
summaries of relevant parts of
Sec. 4.3.2, 4.4, 4.7 of <d-cite key=farrell2008navigation></d-cite>.
The third part is where things get really interesting. We consider 
continuous to discrete time noise "in a vacuum"
and try to decouple the effect of system dynamics from the
random process noise. There is a commonly used formula, which yields good results in practice,
but as far there is no completely satisfying way to derive it from first principles. 
We go through some approaches that have been proposed and discuss them.  -->
<!-- For instance,
white noise is not physically possible <d-cite key=farrell2008navigation></d-cite>, Sec. 4.4.2.1 as it has infinite power, and is an assumption that works for cases where the noise spectral constant interval is much larger than the bandwidth of interest.  -->

In robotics, before anything else, the robot must estimate its **state**. An aerial vehicle must know its position, velocity, and heading
before planning its path and creating a control law to follow it. An autonomous vehicle must know its velocity
and the position of surrounding vehicles. 
Even a Roomba vacuum cleaner must know its location relative to the household cat in order to avoid an early demise
at the hands of General Mittens.
The **state** is thus whatever quantity of interest must be estimated for subsequent safe, reliable robot operation.

Furthermore, the state is never known perfectly but rather **estimated** from sensor measurements, which are
themselves imperfect. An accelerometer is corrupted by noise; a camera can suffer from motion blur;
feature points on images may be incorrectly associated across frames to yield outliers.
The problem is thus statistical in nature, and is often solved in a Maximum A Posteriori optimization framework,
which takes the form of a nonlinear least squares problem. For "well-behaved" problems, where there are no outliers, Gaussian sensor models are used. In cases where outliers are present, robust losses are used to address the effect of the outliers' high residuals on the solution quality. 

# Nonlinear Least Squares
For Gaussian sensor models, the Maximum A Posteriori problem takes the nonlinear least squares form
<p>
\begin{align}
\hat{\mathbf{x}}&=\text{argmin}_{\mathbf{x}} J(\mathbf{x}) = \text{argmin}_{\mathbf{x}}  \sum_{i=1}^N \frac{1}{2} \mathbf{e}_i^T(\mathbf{x}) \mathbf{e}_i(\mathbf{x}),
\end{align}
</p>
where each $\mathbf{e}_i(\mathbf{x})$ is a nonlinear, vector function of the state $\mathbf{x}$, and usually corresponds to
the difference between **_expected_** (based on the sensor model) and **_actual_** value of the $i$'th received measurement. 

The Newton optimization update at an iterate $\mathbf{x}^k$ is derived by minimizing a quadratic expansion expanded around the current iterate to yield
<p>
\begin{align}
\frac{\partial J}{\partial \mathbf{x} \partial \mathbf{x}^T} \Delta \mathbf{x} =
-\frac{\partial J}{\partial \mathbf{x}^T}.
\end{align}
</p>
<!--Should add matrix derivative identities to useful idienties sectio-->
For the nonlinear-least-squares form of the loss $J(\mathbf{x})$, the Jacobian and Hessian become
<p>
\begin{align}
\frac{\partial J}{\partial \mathbf{x}^T} &= \sum_{i=1}^N \frac{\partial J}{\partial \mathbf{e}_i} 
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}  = \sum_{i=1}^N \mathbf{e}_i^T \frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}, \\ 
\frac{\partial J}{\partial \mathbf{x} \partial \mathbf{x}^T} &= 
\sum_{i=1}^N 
\left(\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T}  \frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}
+\sum_{j=1}^{n_{e_i}} e_{i,j} \frac{\partial ^2 e_{i,j}}{\partial \mathbf{x} \partial \mathbf{x}^T}
\right).
\end{align}
</p>
The second term in the Hessian sum is neglected to yield the Gauss-Newton update, 
<p>
\begin{align}
\left(\sum_{i=1}^N \frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T}  \frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}\right)
\Delta \mathbf{x}
&=
\sum_{i=1}^N \mathbf{e}_i^T \frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}.
\end{align}
</p>
Typically we see all the error terms stacked into one big error, which yields a cleaner expression. However,
for the sake of the robust loss derivation later on, we will keep them separated. 


As an aside, the convenient way to derive/check vector derivative identities is through expanding the matrix operation
as a sum, carrying out the differentiation, and reassembling the output as a matrix expression. 
For example, the Jacobian of the quadratic form $\mathbf{e}^T \mathbf{e}$ may be derived by writing
the definition of the Jacobian, and keeping in mind that we are working with a scalar such that it only has one row, 
<p>
\begin{align}
\left . \frac{\partial \mathbf{e}^T \mathbf{e}}{\partial \mathbf{e}} \right|_{1j} &=
\frac{\partial}{\partial e_j} \sum_{i=1}^N e_i^2 = 2e_j, 
\end{align}
</p>
which may be reassembled back into matrix form as
<p>
\begin{align}
\frac{\partial \mathbf{e}^T \mathbf{e}}{\partial \mathbf{e}} &= 2\mathbf{e}^T.
<!-- \sum_{i=1}^N \frac{\partial J}{\partial \mathbf{e}} \frac{\partial \mathbf{e}}{\partial \mathbf{x}}  = \mathbf{e}^T \frac{\partial \mathbf{e}}{\partial \mathbf{x}}.  -->
\end{align}
</p>

# Robust Loss
In the case of robust losses, a robust
loss $\rho_i: \mathbb{R}\rightarrow \mathbb{R}$ is assigned to each quadratic form $f_i=\frac{1}{2}\mathbf{e}_i^T\mathbf{e}_i$, which
downweighs the effects of very high residuals. See <d-cite key=Barron17></d-cite> for examples of robust loss functions. 
The Maximum A Posteriori problem is
<p>
\begin{align}
\hat{\mathbf{x}}&=\text{argmin}_{\mathbf{x}} J(\mathbf{x}) =
\text{argmin}_{\mathbf{x}} \sum_{i=1}^N \rho_i\left(\frac{1}{2}\mathbf{e}_i^T(\mathbf{x}) \mathbf{e}_i(\mathbf{x})\right)
\end{align}
</p>

The Jacobian of the loss is given by
<p>
\begin{align}
\frac{\partial J}{\partial \mathbf{x}^T} &= \sum_{i=1}^N \frac{\partial J}{\partial \mathbf{e}_i} 
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}  = \sum_{i=1}^N\frac{\partial \rho_i}{\partial f_i}\mathbf{e}_i^T \frac{\partial \mathbf{e}_i}{\partial \mathbf{x}},
\end{align}
</p>
while the Hessian of the loss is given by
<p>
\begin{align}
\frac{\partial J}{\partial \mathbf{x} \partial \mathbf{x}^T} &= 
\left(
\frac{\partial}{\partial \mathbf{x}} 
\frac{\partial J}{\partial \mathbf{x}^T}\right)^T \\ 
&= 
\left(
\frac{\partial}{\partial \mathbf{x}}
\sum_{i=1}^N 
\left(
\frac{\partial \rho_i}{\partial f_i} \frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T} \mathbf{e}_i 
\right)
\right)^T.
\end{align}
</p>
The derivative of the summand is obtained by 
a double application of the product rule,
such that
<p>
\begin{align}
\frac{\partial}{\partial \mathbf{x}}
\left(
\frac{\partial \rho_i}{\partial f_i}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T} \mathbf{e}_i 
\right)
&= 
\frac{\partial}{\partial \mathbf{x}}
\left(\frac{\partial \rho_i}{\partial f_i}\right)
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T} \mathbf{e}_i 
+
\frac{\partial \rho_i}{\partial f_i}
\left(
\left(
\frac{\partial}{\partial \mathbf{x}}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T}
\right)
\mathbf{e}_i
+
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}
\right).
\end{align}
</p>
The key step in the Gauss-Newton iteration, which is carried over to the robust loss case, 
is in neglecting the second order derivatives of $\mathbf{e}_i$, such that 
$\frac{\partial}{\partial \mathbf{x}}
 \frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T}\approx \mathbf{0}$.
 Furthermore, the chain rule must be applied since
since $\frac{\partial \rho_i}{\partial f_i}$ is itself a function of $\mathbf{x}$.
This yields
<p>
\begin{align}
\frac{\partial}{\partial \mathbf{x}}
\left(
\frac{\partial \rho_i}{\partial f_i}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T} \mathbf{e}_i 
\right)
&= 
\frac{\partial}{\partial \mathbf{x}}
\left(\frac{\partial \rho_i}{\partial f_i}\right)
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T} \mathbf{e}_i 
+
\frac{\partial \rho_i}{\partial f_i}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}} \\ 
&= 
\frac{\partial^2 \rho_i}{\partial f_i^2}\mathbf{e}_i^T 
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T} \mathbf{e}_i 
+
\frac{\partial \rho_i}{\partial f_i}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}.
\end{align}
</p>
The overall Hessian is obtained by substituting the summand expression we just derived back into the sum, 
<p>
\begin{align}
\frac{\partial J}{\partial \mathbf{x} \partial \mathbf{x}^T} &= 
\left(
\frac{\partial}{\partial \mathbf{x}} 
\frac{\partial J}{\partial \mathbf{x}^T}\right)^T \\ 
&= 
\sum_{i=1}^N 
  \frac{\partial^2 \rho_i}{\partial f_i^2}\mathbf{e}_i^T 
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T} \mathbf{e}_i 
+
\frac{\partial \rho_i}{\partial f_i}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}.
\end{align}
</p>

The first term in the sum is called the __Triggs correction__ <d-cite key=triggs1999bundle></d-cite>.
Neglecting it yields the iteratively reweighted least squares approach, where the Newton step is given by
<p>
\begin{align}
 \sum_{i=1}^N
\frac{\partial \rho_i}{\partial f_i}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}^T}
\frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}
\Delta \mathbf{x}
&= 
 \sum_{i=1}^N\frac{\partial \rho_i}{\partial f_i}\mathbf{e}_i^T \frac{\partial \mathbf{e}_i}{\partial \mathbf{x}}
\end{align}
</p>
Comparing this expression to the Gauss-Newton case clarifies why it is called iteratively reweighted least squares. 
For each factor with a robust loss, we just consider the *non-weighted* error, and then
just weigh it by $\sqrt{\frac{\partial \rho_i}{\partial f_i}}$, which yields exactly the nonlinear-least-squares
update for the robust loss case we just derived. 
This is done for every factor separately. We can notice that if only one factor is used, the
$\sqrt{\frac{\partial \rho_i}{\partial f_i}}$'s cancel out.

# Remarks
Different robust losses may be used, and correspond to specific heavy-tailed distributions if we work "backward"
to the MAP problem from the negative log-likelihood. A Cauchy loss corresponds to a Cauchy distribution assumed on sensor noise. 
However, the motivation for using robust losses is usually not any kind of rigorous statistical argument. We do not consider the statistical properties of erroneous feature associations across camera frames to determine the "right"
robust loss. Rather, we use trial and error. It works because it works, and because we have nice methods for solving the resulting problem. 
This somewhat echoes a theme I've seen on Ben Recht's substack. We often work with given models not because they are the most accurate or rigorous ones. Rather, we work with them because we are able to solve them.
Afterward, we can try to justify why these models correspond to reality. 