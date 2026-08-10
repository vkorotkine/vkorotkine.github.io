---
layout: distill
title: filtering updates on lie groups
description: moving gaussians around curves 
tags: 
giscus_comments: true
date: 2025-07-16
featured: false
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

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
# toc:
# #   - name: Lie Groups
#     # if a section has subsections, you can add them as follows:
#     # subsections:
#     #   - name: Example Child Subsection 1
#     #   - name: Example Child Subsection 2
#   - name: Random Processes
#   - name: From Continuous-Time Covariances to Discrete Time
#     subsections:
#       - name: Linearize Then Discretize
#       - name: Continuous to Discrete Time Noise "In A Vacuum"
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

$\require{boldsymbol}$
Robot navigation is the task of determining a robot state from noisy sensor measurements,
and therefore involves statistics and errors. However, our robot states often involve rotations, which raises questions
on how we add and subtract rotations. We have to reconsider addition, subtraction, integration, and so on. We need Lie groups (some <a href="{{ site.baseurl }}/assets/pdf/notes/lie_group_doc.pdf">notes</a> on Lie groups,
and they are also covered in <d-cite key=barfoot2024state></d-cite> and <d-cite key=sola2021microlietheorystate></d-cite>). 

In this post I specifically focus on how we move gaussians around on Lie groups. 
Our robot belief is typically a gaussian. In cases where there are no rotations,
and our robot state lives in a vectorspace where everything is nice. 
The robot state $\mathbf{x}$ is updated, typically through a Kalman filter correction step
where the belief is conditioned on a received measurement (
  see <d-cite key=sarkka2023bayesian></d-cite> for detailed expressions
). The initial robot belief is a gaussian $\mathcal{N}(\mathbf{0}, \check{\mathbf{P}})$, that is then updated
to $\mathcal{N}(\hat{\mathbf{x}}, \hat{\mathbf{P}})$.
 <!-- robot state, that contains rotations, is denoted $\mathcal{X}$.  -->
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/blogs/vectorspace_distribution_update.png" class="img-fluid rounded z-depth-1" style="width: 50%;" %}
    </div>
</div>
<div class="caption">
    The nice vectorspace case. 
</div>

For the Lie group case, the state now lives on a manifold and is denoted $\mathcal{X}$. 
Notions of addition and subtraction are allowed in the Lie algebra, and in the __tangent__ spaces
defined at each $\mathcal{X}$.
State beliefs are now represented as __concentrated__ gaussians,
where $\mathcal{X}=\bar{\mathcal{X}} \oplus \boldsymbol{\xi}, \quad \boldsymbol{\xi} \sim \mathcal{N}(\mathbf{0}, \mathbf{P})$. The gaussian lives in the tangent space and the distribution over the manifold is defined by its relationship to the tangent space. There are now two steps that must be done to update the robot state. 
First, in the initial state belief's tangent space,
the gaussian  $\check{\boldsymbol{\xi}}$ distributed as
$\mathcal{N}(\mathbf{0}, \check{\mathbf{P}})$
is updated to  $\mathcal{N}(\bar{\boldsymbol{\xi}}, \bar{\mathbf{P}})$. This is done by writing
the measurement model in terms of $\boldsymbol{\xi}$ and carrying out the standard Kalman filter updates
on $\boldsymbol{\xi}$. This yields a distribution over $\boldsymbol{\xi}$ with a nonzero mean $\bar{\boldsymbol{\xi}}$. 
However, when we use concentrated gaussians, we want 
a distribution of the form 
$\mathcal{X}=\hat{\mathcal{X}} \oplus \boldsymbol{\xi}, \quad \boldsymbol{\xi} \sim \mathcal{N}(\mathbf{0}, \hat{\mathbf{P}})$,
that has a zero mean for $\boldsymbol{\xi}$.
We want our robot state belief to be at a given updated state $\hat{\mathcal{X}}$, without any strange tangent space additions. 

This whole process is illustrated in the following picture. 
<!-- \oplus \boldsymbol{\xi}$ -->
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/blogs/manifold_distribution_update.png" class="img-fluid rounded z-depth-1" style="width: 50%;" %}
    </div>
</div>
<div class="caption">
    The more complicated Lie group case. 
</div>

The question is then, what are $\hat{\mathcal{X}}$ and $\hat{\mathbf{P}}$?
The key point is that the distributions given by
$\mathcal{X}=\check{\mathcal{X}}\oplus \hat{\boldsymbol{\xi}}, \quad \hat{\boldsymbol{\xi}}\sim
\mathcal{N}(\bar{\boldsymbol{\xi}}, \bar{\mathbf{P}})$ and 
$\mathcal{X}=\hat{\mathcal{X}}\oplus \boldsymbol{\xi}, \quad \boldsymbol{\xi}\sim
\mathcal{N}(\mathbf{0}, \hat{\mathbf{P}})$ are the __same__ distribution, just expressed in different tangent spaces
of the manifold. 

We can therefore do a bit of math to manipulate
$\mathcal{X}=\check{\mathcal{X}}\oplus \hat{\boldsymbol{\xi}}, \quad \hat{\boldsymbol{\xi}}\sim
\mathcal{N}(\bar{\boldsymbol{\xi}}, \bar{\mathbf{P}})$. 
<p>
\begin{align*}
    \mathcal{X} &= \check{\mathcal{X}} \oplus \check{\boldsymbol{\xi}}, \quad \check{\boldsymbol{\xi}} \sim \mathcal{N}(\bar{\boldsymbol{\xi}}, \bar{\mathbf{P}}) \\
    &= \check{\mathcal{X}} \oplus (\bar{\boldsymbol{\xi}} + \tilde{\boldsymbol{\xi}}), \quad \tilde{\boldsymbol{\xi}} \sim \mathcal{N}(0, \bar{\mathbf{P}}) \\
    &\approx \left( \check{\mathcal{X}} \oplus \bar{\boldsymbol{\xi}} \right) \oplus 
    \underbrace{ \left. \frac{D(\mathcal{X} \oplus \boldsymbol{\tau})}{D\boldsymbol{\tau}} \right|_{\bar{\boldsymbol{\xi}}} }_{\mathbf{J}_{\tilde{\boldsymbol{\xi}}}} 
    \tilde{\boldsymbol{\xi}}, \quad \tilde{\boldsymbol{\xi}} \sim \mathcal{N}(0, \bar{\mathbf{P}}) \\
    &= \hat{\mathcal{X}} \oplus \boldsymbol{\xi}, \quad \boldsymbol{\xi} \sim \mathcal{N}(0, \mathbf{J}_{\tilde{\boldsymbol{\xi}}} \bar{\mathbf{P}} \mathbf{J}_{\tilde{\boldsymbol{\xi}}}^\top)
\end{align*}
</p>

The crucial step comes in the third line, where we use the Lie group Jacobian of the plus operation,
which is the same as the group Jacobian <d-cite key=sola2021microlietheorystate></d-cite>. 
This Jacobian accounts for the difference in the tangent spaces at $\check{\mathcal{X}}$
and $\hat{\mathcal{X}}$. 
This explains the presence of the Jacobian in Lie group filter estimation codes such as in 
<a href="https://github.com/decargroup/navlie/blob/main/navlie/filters.py"> our lab's navlie library. </a>
For a small update, where $\tilde{\boldsymbol{\xi}}$ is small, 
this Jacobian is close to identity. However, for larger updates, neglecting it can cause
inaccuracies in the updated covariance $\hat{\mathbf{P}}$. 
