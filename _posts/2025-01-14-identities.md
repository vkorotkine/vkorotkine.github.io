---
layout: distill
title: useful identities
description: quick mathematical facts I rederive every so often
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
toc:
  - name: Lie Groups
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

The following is a random collection of identities and facts I encounter semi-regularly in my research. 
## Lie Groups
Lie groups are ubiquitous in robotics due to the need to handle rotations. 
If you are looking for a comprehensive introduction to the subject, <d-cite key="solà2021microlietheorystate"></d-cite> and <d-cite key="barfoot2024state"></d-cite>
are classic resources. 

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
This ties into the $\ \odot\ $ operator as described in <d-cite key="barfoot2024state"></d-cite> in the section covering homogeneous points.
