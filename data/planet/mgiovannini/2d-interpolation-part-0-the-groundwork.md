---
title: '2D Interpolation, Part 0: The Groundwork'
description: (First of a series)  My process for going from a textbook implementation
  of an algorithm to an efficient production-grade version of it is t...
url: https://alaska-kamtchatka.blogspot.com/2012/06/2d-interpolation-part-0-groundwork.html
date: 2012-06-22T17:51:00-00:00
preview_image: https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgL1sKv8sVOBhWvq9S1wLJx3JuJbcv12oSi6Kdryro5mFLFJRVpAjy9yfixWC6RYacM6WsIv4EZwo3W5O7yByJkHYVFa8A5lRtjFKew0_AXDERkU9helcYTw-D-tLiKZdRnhEGImmE7o64/w1200-h630-p-k-no-nu/interp0.png
authors:
- "Mat\xEDas Giovannini"
source:
---

(First of a series) My process for going from a textbook implementation of an algorithm to an efficient production-grade version of it is to methodically apply meaning-preserving transformations that make the code progressively tighter. In this case I'll massage a naïve 2D interpolator that selectively uses nearest-neighbor, bilinear and bicubic interpolation. I started by studying a basic 
