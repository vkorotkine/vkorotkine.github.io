---
layout: distill
title: noise in batch models that enters nonlinearly
description: linearization everywhere
tags: 
giscus_comments: true
date: 2021-05-22
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

bibliography: 2025-01-14-identities.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
# toc:
#   - name: Lie Groups
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  # - name: Citations
  # - name: Footnotes
  # - name: Code Blocks
  # - name: Interactive Plots
  # - name: Mermaid
  # - name: Diff2Html
  # - name: Leaflet
  # - name: Chartjs, Echarts and Vega-Lite
  # - name: TikZ
  # - name: Typograms
  # - name: Layouts
  # - name: Other Typography?

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

Batch optimization methods are ubiquitous in robotics.
We would like to solve for some robot states given some measurements.
To do this, we create an error that
quantifies the difference between **_the measurement we expect_** (that depends on the state)
and the **_the measurement we receive_** (a given value from a sensor, and does not depend on the robot state).

Consider a single received measurement $\mathbf{y}$,
the sensor model for which is $\mathbf{y}=\mathbf{g}(\mathbf{x})+\mathbf{v}$, where
$\mathbf{v}\sim \mathcal{N}(\mathbf{0}, \mathbf{R})$ is Gaussian distributed noise
with covariance $\mathbf{R}$.
The robot state estimate is obtained by solving the Max A Posteriori problem
\begin{equation}
\hat{\mathbf{x}}=\text{argmax}_{\mathbf{x}} p(\mathbf{x}|\mathbf{y}),
\end{equation}

Since there are no priors, and discarding the normalization constant, Bayes' rule leaves us with
\begin{equation}
\hat{\mathbf{x}}=\text{argmax}_{\mathbf{x}} p(\mathbf{y}|\mathbf{x}).
\end{equation}

which is equivalent to minimizing the negative log-likelihood as
\begin{equation}
\hat{\mathbf{x}}=\text{argmin}_{\mathbf{x}} -\log p(\mathbf{y}|\mathbf{x}).
\end{equation}

The form for $p(\mathbf{y}|\mathbf{x})$ is Gaussian, meaning that
\begin{equation}
p(\mathbf{y}|\mathbf{x}) = \alpha \exp(-(\mathbf{y}-\mathbf{g}(\mathbf{x}))\mathbf{R}^{-1}(\mathbf{y}-\mathbf{g}(\mathbf{x}))),
\end{equation}

where $\alpha$ is a normalization constant. By defining $\mathbf{e}(\mathbf{x})=\mathbf{y}-\mathbf{g}(\mathbf{x})$, 
the negative log-likelihood minimization can be written 
\begin{equation}
\hat{\mathbf{x}}=\text{argmin}_{\mathbf{x}} \mathbf{e}(\mathbf{x}) \mathbf{R}^{-1} \mathbf{e}(\mathbf{x}). 
\end{equation}

This is pretty much in the nonlinear-least-squares form that is used by solvers such as Ceres.
We linearize the error $\mathbf{e}(\mathbf{x})$ to construct successive Taylor series approximations to our loss
and (hopefully) reach the minimum we want. 


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/blogs/linearization_everywhere.jpg" class="img-fluid rounded z-depth-1" style="width: 50%;" %}
    </div>
</div>
<div class="caption">
    A favorite engineering pastime. 
</div>
However, what happens when the noise enters nonlinearly into the measurement model? Formally, 
\begin{equation}
\mathbf{y}=\mathbf{g}(\mathbf{x}, \mathbf{v}) \quad
\mathbf{v}\sim \mathcal{N}(\mathbf{0}, \mathbf{R}).
\end{equation}

The measurement likelihood $p(\mathbf{y}|\mathbf{x})$ is non-Gaussian and forming the error
\begin{equation}
\mathbf{e}(\mathbf{x})=\mathbf{y}-\mathbf{g}(\mathbf{x}, \mathbf{v}),
\end{equation}

gets us nowhere. So we linearize and write
\begin{equation}
\mathbf{y}=\mathbf{g}(\mathbf{x}) + \mathbf{v}\quad
\mathbf{v}\sim \mathcal{N}(\mathbf{0}, \mathbf{L} \mathbf{R} \mathbf{L}^{\text{T}}), 
\end{equation}

where $\mathbf{L}$ is the Jacobian of the measurement model with respect to the noise variable evaluated at the current state estimate $\bar{\mathbf{x}}$, 
\begin{equation}
\mathbf{L} = \left. \frac{\partial \mathbf{g}}{\partial \mathbf{x}}\right|_{\bar{\mathbf{x}}}.
\end{equation}

This is different from the linearization approximation that we use in methods like Gauss-Newton, where
we first pose the problem then use iterative methods to solve it.
*_before even going into the optimizer_* we make a linearization approximation to the loss.
Whether this causes issues depends on the application. We can get funny situations where $\mathbf{L}$ is not full rank,
causing a singular covariance. 
<!-- \begin{align}
    \hat{\mathbf{x}}&=\text{argmax}_{\mathbf{x}} p(\mathbf{x}|\mathbf{y}) \\\ &= 
    \begin{bmatrix}
        0 & -v\\\ v & 0
    \end{bmatrix}
    \begin{bmatrix}
        u_1 \\\ u_2 
    \end{bmatrix} \\\ 
    &= 
    \begin{bmatrix}
        0 & -1 \\\ 1 & 0
    \end{bmatrix}
    \begin{bmatrix}
        u_1 \\\ u_2 
    \end{bmatrix} 
    v \\\ 
    &= 
    \mathbf{P} \mathbf{u}v,
\end{align} -->
<!-- 
Fact 1: For the $SO(2)$ group of 2D rotations, the following holds, 
\begin{align}
    v^\wedge \mathbf{u} &= 
    \begin{bmatrix}
        0 & -v\\\ v & 0
    \end{bmatrix}
    \begin{bmatrix}
        u_1 \\\ u_2 
    \end{bmatrix} \\\ 
    &= 
    \begin{bmatrix}
        0 & -1 \\\ 1 & 0
    \end{bmatrix}
    \begin{bmatrix}
        u_1 \\\ u_2 
    \end{bmatrix} 
    v \\\ 
    &= 
    \mathbf{P} \mathbf{u}v,
\end{align}
Where $$\mathbf{P}=\begin{bmatrix} 0 & -1 \\\ 0 & 1\end{bmatrix}$$. This contrasts with the $SO(3)$ case, where $$\mathbf{v}^\wedge \mathbf{u} =  -\mathbf{u}^\wedge \mathbf{v}$$.
This ties into the $\ \odot\ $ operator as described in <d-cite key="barfoot2024state"></d-cite> in the section covering homogeneous points. -->
