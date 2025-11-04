---
title: Approximation Techniques for Hardware Implementation of Image Procssing - Part 1
description: Talking about some approximation techniques suitable for hardware
author:
date: 2019-07-19 00:00:00 +0100
categories: [FPGA, MATLAB]
tags: [FPGA]
pin: true
math: true
mermaid: true
--- 

A major setback! I originally planned to complete the Canny edge detection algorithm tutorial tonight, but couldn't because the MATLAB in the lab doesn't have the `DSP Toolbox`, which prevented me from using the `atan2` HDL module. I'll have to wait until Monday to work on it at the office. So today, let's talk about some approximation techniques commonly used in image processing hardware implementation. This will serve as a general summary, and if you have related knowledge, feel free to share in the comments!

---

## 1. The Obsession with 255

As we all know, RGB images are typically represented using the `uint8` data type, with a value range of [0, 255]. Therefore, when normalization is required (e.g., in RGB to HSV conversion), we often need to divide by the awkward number 255, instead of 256, which could be easily achieved by simply shifting right by 8 bits. In practice, however, dividing by 256 generally does not introduce significant errors in most digital image systems, and this is indeed what we do in engineering applications. This approach saves one DSP unit.

For applications requiring higher precision, consider the following approximation formula:  
`(x + (x >> 8)) >> 8`  
This uses a 16-bit adder instead of a multiplier, saving on-chip hard core resources.

---

## 2. Separable Convolution

For a 3x3 Gaussian filter, directly processing a 3x3 patch would require 9 multipliers and a bunch of adders. However, due to the properties of the 2D Gaussian matrix, we know it can be decomposed into the outer product of two 3x1 vectors. This allows us to perform multiplication and addition separately for each vector. In this case, only 6 multipliers and several adders are needed. The resource savings are even more significant for larger Gaussian kernels. You can refer to the example on the [MathWorks website](https://www.mathworks.com/help/visionhdl/examples/using-line-buffer-create-separable-filter.html)

This method can also be applied to other algorithms that can process rows and columns separately, such as image resizing. Related examples will be available in `MATLAB 2020a`.

---

## 3. Calculating Euclidean Distance

In the RANSAC algorithm, we often need to calculate the Euclidean distance between selected points and the center point. The square root operation involved consumes substantial hardware resources. Inspired by the Sobel operator, we approximate  
`sqrt((x - x_bar)^2 + (y - y_bar)^2)`  
as the Manhattan distance:  
`abs(x - x_bar) + abs(y - y_bar)`  
This avoids the square root computation entirely.

---

## Recommended Books

[1] Design for Embedded Image Processing on FPGAs
[2] Hacker's Delight (2nd Edition)