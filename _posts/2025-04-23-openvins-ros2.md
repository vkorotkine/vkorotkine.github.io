---
layout: distill
title: getting openvins up and running with ros2
description: notes on getting stuff running
tags: 
giscus_comments: true
date: 2025-04-23
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

bibliography: 2025-03-20-discretization.bib

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

# Quickstart
Following the installation instructions
<a href="https://docs.openvins.com/gs-installing.html">for installing OpenVINS with ROS 2 </a>. 
Following the ROS 2
<a href="https://docs.openvins.com/gs-tutorial.html#gs-tutorial-ros2">for installing OpenVINS with ROS 2 </a>. 
OpenVINS tutorial for running on EuRoC dataset. 
Running RViz using ROS 2 instead of ROS1. Note that simply running ```rviz``` from the command
line like for ROS 1 will not work. Instead, do not install anything extra and run

Specifically for the EuRoC example from OpenVINS, 
```
ros2 run rviz2 rviz2 -d /home/vassili/projects/openvins_workspace/catkin_ws_ov/src/open_vins/ov_msckf/launch/display_ros2.rviz
```
where the global path is used (for now), since relative paths didn't seem to be able to find the display file. 
Checking the
<a href="https://docs.ros.org/en/humble/Tutorials/Intermediate/RViz/RViz-User-Guide/RViz-User-Guide.html#install-or-build-rviz"> ROS 2 guide for RViz </a> is handy.  
Then, as per the OpenVINS documentation, 
```
ros2 launch ov_msckf subscribe.launch.py config:=euroc_mav
ros2 bag play V1_01_easy.db3
```
which are to be run in different terminals. 

# VS Code Intellisense for ROS
To have the IntelliSense working for Python ROS packages, such as launch files, make sure to run VS Code as ```code``` from a terminal where
the ROS setup has been sourced. 
<a href="https://robotics.stackexchange.com/questions/97707/how-can-i-make-vs-code-recognize-ros2-python-packages
"> Source. </a>
Furthermore, even though ROS installs its own versions of Eigen and OpenCV, I found that I had to install them separately to be able to point VS Code's ```includePath``` to their libraries and have IntelliSense work properly.
```
sudo apt install libeigen3-dev
sudo apt install libopencv-dev
```
After which the following ```cpp_properties.json``` file does the trick, 
```
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**", 
                "/usr/local/include/Eigen/**", 
                "/usr/include/opencv4/**", 
                ""/opt/ros/humble/**"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "c17",
            "cppStandard": "gnu++17",
            "intelliSenseMode": "linux-gcc-x64"
        }
    ],
    "version": 4
}
```
There is also the following
<a href="https://docs.ros.org/en/jazzy/How-To-Guides/Setup-ROS-2-with-VSCode-and-Docker-Container.html
"> guide for using VS Code with ROS 2 and Docker. </a>



# Evaluation
We now run the EuRoC dataset such that we can do the full algorithm evaluation. 
```
colcon build
source install/setup.bash # from workspace directory
ros2 launch ov_msckf subscribe.launch.py config:=euroc_mav # from workspace directory
ros2 bag play V1_01_easy.db3 # from data bag directory
ros2 launch ov_eval record.launch.py  # from workspace directory
```

Debugging: 
```
ros2 node info /pose_to_file # check that subscription is actually created 
```


