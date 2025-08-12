---
layout: post
title: portfolio!
permalink: /portfolio/
---

Hi!

This portfolio is something I use to showcase work I've done in the past.

You'll notice some lack of polish on this - this is due to time pressure and my expectation of a mostly-engineer audience who will focus on the technical aspects instead of the presentation. And I'm certainly not trying to show off my skill as a web designer here 😅

This is ordered by "significance of technical achievement" - the things I'm most proud of are at the top, and the honorable mentions are at the bottom :)

---------------------

# Monado's hand tracking

This is by far the coolest thing I've worked on so far. This is a low-latency, accurate and smooth optical hand tracking system, built from scratch (not on top of Mediapipe, although [I tried that too](https://www.collabora.com/news-and-blog/blog/2022/05/31/monado-hand-tracking-hand-waving-our-way-towards-a-first-attempt/)) with a completely open-source implementation and dataset.

Many considered this problem one best left to the big leagues of Ultraleap and Oculus. I thought differently, and we ended up with a really good system that basically works on any VR headset with calibrated cameras.

At Collabora I've done maybe 90% of the relevant work:

* Scraping, collecting, and generating training data
* Writing and wrangling machine learning training infustructure
* Writing realtime inference code in C++
* Creating a realtime nonlinear optimizer to get smooth 3D hand pose trajectories
* Writing various infrastructure to deal with camera streaming, camera calibration, distortion/undistortion, etc.

This took me about a year, right out of college. I was a pretty huge novice to computer vision, machine learning, and algorithms, but through hard work and waging holy war against impostor syndrome, we made it happen!

## Full-system demo

<html>
    <video autoplay controls loop muted width="100%">
      <source src="/assets/videos/mercury_demo_3.webm#t=47" type="video/webm">
      Your browser doesn't seem to support video! If you're using a modern browser, please contact me using the contact info at the bottom of this page so I can fix it!
    </video>
</html>

([Twitter link](https://t.co/zitKhc66Yp), [Collabora blog post](https://www.collabora.com/news-and-blog/news-and-events/monados-mercury-hand-tracking-now-ready-for-use.html))

These are first-person recordings on my custom Project North Star headset, Valve Index and Reverb G2 showing off our hand tracking. These were recorded live, nothing added in post!

## Artificial dataset generator

<html>
    <video autoplay controls loop muted width="100%">
      <source src="/assets/videos/hand_grid_loop_long.webm" type="video/webm">
      Your browser doesn't seem to support video! If you're using a modern browser, please contact me using the contact info at the bottom of this page so I can fix it!
    </video>
</html>

<!-- 
* Got some hand scans from various asset stores
* (The releasable ones are available [here](https://gitlab.freedesktop.org/monado/utilities/hand-tracking-playground/hand_scans))
* Retopologized some of them (maybe 10%)
* Rigged all of them from scratch
* In Blender: Wrote a Python script that talks to a C++ manager, gets mocap data, animates the hand to follow that data, and renders images centered on the hand to files.
* In C++: Wrote a multithreaded manager that communicates over sockets to 8-16 Blender instances, retrieves mocap data (actually by randomly combining head-to-hand offset sequences and finger-to-wrist sequences for diversity), uses nonlinear optimization to retarget the mocap data to the proportions of that blender instance's model. -->

I collected various hand scans from asset stores, releasing the open-licensed ones [here](https://gitlab.freedesktop.org/monado/utilities/hand-tracking-playground/hand_scans). I retopologized the ones that needed it and rigged all of them myself to ensure accurate deformation across the range of human hand poses.

To automate hand animation and rendering, I developed a Blender Python script that communicates with a manager process written in C++, receiving mocap data with which it animates the hand and renders annotated images.

To manage multiple Blender instances, I built a multithreaded C++ manager program that communicates with several concurrent Blender instances over sockets. It retrieves mocap data by combining randomly chosen head/wrist pose mocap data collected from Lighthouse tracking with finger pose data either collected from an earlier version of our hand tracking pipeline or procedurally generated. Then it retargets the mocap data using nonlinear optimization to fit each hand scan exactly and sends it to the Blender instance.

This work was pretty challenging, requiring me to learn a new-to-me part of Blender, a fair bit about orchestrating headless Blender instances and reusing our nonlinear optimizer in a novel way to create diverse, smooth hand trajectories.

## Neural nets

Our hand tracking pipeline currently has two main ML components: the hand detector and the keypoint estimator. The hand detector is very small and doesn't run every frame - this is intentional to reduce compute load. The keypoint estimator is a bit bigger and runs every frame in each camera view for each hand.

The keypoint estimator does a few things:

* Estimates hand landmarks as 2D heatmaps
* Estimates the distance of each hand landmark from the camera, relative to the middle-proximal joint, as a 1D heatmap
* Estimates each finger's curl value
* Tries to predict whether or not the input image contains a hand, which we use as one of the termination conditions to stop tracking the hand under high occlusion or in the case of erroneous detections

Here's some imagery of the debug view of our optical hand tracking pipeline. The four cropped views are the input to the keypoint estimator models. The number to the right is the confidence that these are hands (you can see it's working! the probability is one, and they're definitely all hands!), the dots to the right are the 2D heatmaps, the lines below and to the right are the 1D heatmaps, and the curling lines at the bottom are visualization of the finger curl values.

<html>
      <img src="/assets/images/DEBUG.png" alt="steam" width="100%">
</html>

The hand detector is simple: it takes full camera views cropped down to 160x160 and outputs a 8D vector corresponding to the center, radius and confidence of the left and right hand bounding boxes.

<html>
      <img src="/assets/images/kitty.png" alt="steam" width="33%">
</html>

Both of these models were pretty heavily inspired by [Facebook's MeGATrack paper](https://research.facebook.com/publications/megatrack-monochrome-egocentric-articulated-hand-tracking-for-virtual-reality/) and [milkcat0904's PyTorch reimplementation of the model architectures](https://github.com/milkcat0904/MegaTrack-pytorch) although we've iterated a bit on the base model architectures by now.

Also, to be clear, we don't have any of Facebook's weights or dataset - we did train these from scratch! Shoutout to facebook for doing quite actually-open research (publish your datasets next pls) and milkcat0904 for their very solid reimplementation!

<!-- TODO Add visualization picture -->
<!-- 
- Two key components: hand detector and keypoint estimator
- Hand detector doesn't always run every single frame and it's a little bit inaccurate - this is *on purpose* to lower compute
- Keypoint estimator runs in every frame in every view for every hand. It's a bit smarter.
- It:
  - Estimates hand landmarks as 2D heatmaps
  - Estimates hand landmarks' distance from the camera relative to the middle-proximal joint
  - Estimates each finger's curl value
  - Estimates the likelihood that the input image actually contains a hand, to help us stop tracking hands under occlusion or erroneous detections -->

## Nonlinear optimizer

This is what we use to turn model predictions into a smooth 6DOF hand trajectory. We're using what I know now to be a fairly normal architecture, also inspired by [Facebook's MeGATrack paper.](https://research.facebook.com/publications/megatrack-monochrome-egocentric-articulated-hand-tracking-for-virtual-reality/)

The story of how we got there was pretty interesting, though.

* Late 2021: Started by implementing very simple SGD from finite differences, from scratch in C++. I did end up getting it working, but it was very slow
* Late 2021-early 2022: Read a bunch about kalman filtering, figured out that we'd _definitely_ need something iterative - I remember Iterative Extended Kalman Filters and single constraint at a time looking somewhat promising
* Early 2022: Wrote a very weird optimizer that iteratively ran Cyclic Coordinate Descent IK on all hand joints, using basically the centerpoint of all the downstream joints as end effector/target joints, and used Umeyama within the loop to position the root joint. This worked but it was jittery and we had very little control over it
* Early-mid 2022: Realized Levenberg-Marquardt was the way to go, especially after reading FB paper. Started by using Eigen's [unsupported LM implementation](https://gitlab.com/libeigen/eigen/-/blob/master/unsupported/Eigen/src/NonLinearOptimization/LevenbergMarquardt.h) with finite differences. This worked but was pretty slow. Messed around with Ceres, Enoki and sympy - ended up using Ceres's tinysolver and packaging it [here.](https://gitlab.freedesktop.org/monado/utilities/hand-tracking-playground/tinyceres) We're still using tinyceres, pretty happy with it!

(I really wanted to include visuals here, but then realized I don't quite know what a nonlinear optimizer looks like. If you know how to take a picture of one, please contact me!)

<!-- <html>
      <img src="/assets/images/architecture.jpg" alt="steam" width="100%">
</html> -->

## Steam launch

This is still ongoing! The store page is finally live, but we have a waiting period where the page has to be on "coming soon" before we can publish binaries.

<!-- ![steam](/assets/images/store_page_screenshot.png) -->

<html>
      <img src="/assets/images/store_page_screenshot.png" alt="steam" width="100%">
</html>

I did all the graphic design, copywriting, and filling-out-Steamworks-forms for this store page. Definitely not groundbreaking - I've seen way too many sunset-colored gradients in VR software branding - but somebody had to do it and I stepped up.

<!-- For more practical considerations it's also worth mentioning that I created the store page in my spare time and own it in my name -->

The store page is [here](https://store.steampowered.com/app/2317150/Monado_Hand_Tracking/)
and all the code is [here.](https://github.com/moshimeow/mercury_steamvr_driver)

Note that that codebase submodules Monado for the heavy lifting and only handles camera communication and being an OpenVR driver.

## Some general other notes about Monado's hand tracking

I did most of the work, but at the same time there's no chance I could have done it without a few key peoples' mentorship, ideas and friendship.

Many thanks to:

* Jakob Bornecrantz for helping with general Monado infrastructure stuff and always reviewing my code.
* Rylie Pavlik for pointing me to the right resources for learning C++, C++ template metaprogramming, and Eigen3. And for helping me learn various tracking algorithm fundamentals.
* Mateo de Mayo for several key good ideas, writing a lot of Monado's shared HT/SLAM camera pipelining and dataset recording infrastructure, and encouraging me to try realtime nonlinear optimization.
* Marcus Edel and Jakub Piotr Cłapa for helping with the very early ML stuff and encouraging me to take the time to really learn PyTorch.
* Nikitha Garlapati and Hampton Moseley for giving me a hand with training data.
* Christoph Haag, Lubosz Sarnecki, Pete Black and all the other Monado contributors for giving me such a solid base of hardware drivers and infrastructure to build on top of.

The code is a little bit spread out, so here are some links:
* [Main inference code](https://gitlab.freedesktop.org/monado/monado/-/tree/main/src/xrt/tracking/hand/mercury)
* [Repo hosting final trained models](https://gitlab.freedesktop.org/monado/utilities/hand-tracking-models)
* [Repo for data generation, annotation, model training and pipeline evaluation](https://gitlab.freedesktop.org/monado/utilities/hand-tracking-playground/mercury_train/)
* [SteamVR driver](https://github.com/moshimeow/mercury_steamvr_driver)

---------------------

# Project North Star

## Designed and built a custom North Star variant

As of 2023, this is the only North Star headset that does all tracking on a single stereo pair - the more common design still uses a Ultraleap tracker paired with some off-the-shelf SLAM tracker, is heavier and has a less-comfortable headgear.

![meow](/assets/images/quitegood20230313_000426.jpg){:width="100%"}

 <details>
 <summary>More images</summary>
 <img src="/assets/images/20230315_010610.jpg" alt="Top view" width="100%">
 <img src="/assets/images/20230315_010619.jpg" alt="Front view" width="100%">
 <img src="/assets/images/ns_screenshot.png" alt="Blender screenshot" width="100%">

 I'm also very happy about this magnet snap-on mount for a through-the-lens camera. Makes demo recording very easy :)
<img src="/assets/images/magnets.jpg" alt="Front view" width="100%">
<img src="/assets/images/magnets_camera.jpg" alt="Front view" width="100%">

 </details>

For this project, I did all the design, printing, assembly and calibration. I think I iterated about 40 times (ie. updating the design, waiting 7 hours for my 3D printer to make it, reassembling it with the new part and testing the feature.) This was really fun and a big part of my 2020 and 2021!

## Created a custom optical calibration method for North Star HMDs

I iterated a _lot_ on this, picking up traditional camera calibration methods, edge detection, thresholding, marker detection, etc. along the way.

The main focus of my work here was _empirical_ calibration, where you place a calibrated stereo camera inside of a HMD where the user's eyes would be, draw a pattern on the display, and just save the mapping from display UV to tanangles as a mesh.

This was my very first "real" computer vision project, and I ended up trying a lot of things before I learned enough to make something that actually worked! In sequential order, this is what I remember doing:

* Canny edge detection - didn't work well because of bloom
* Used thresholding - drawing one line at a time on the vertical and horizontal axes, keeping track of which pixels in the camera image were above a certain threshold for that line, then for each grid point take the intesection of active pixels for the vertical and horizontal line that point corresponded to. This worked but was very unreliable and needed a lot of tuning:

  <html>
      <video autoplay controls loop muted width="100%">
        <source src="/assets/videos/ns_calibration/lines.webm" type="video/webm">
        Your browser doesn't seem to support video! If you're using a modern browser, please contact me using the contact info at the bottom of this page so I can fix it!
      </video>
  </html>

* Just drew Aruco markers on the display and recorded the average position of each corner. This was pretty reliable and is where I stopped.

  I also wrote a force-directed-graph-based system of smoothing the mapping out and coming up with believable grid points for areas that were obscured. If I were doing this today I'd just use nonlinear optimization, and I'd probably use known lens geometry as a prior instead of assuming literally nothing, but hey this way did work!

  <html>
  <div class="half_split_row">
    <div class="half_split_column">
    <video autoplay controls loop muted width="100%">
        <source src="/assets/videos/ns_calibration/aruco.webm" type="video/webm">
        Your browser doesn't seem to support video! If you're using a modern browser, please contact me using the contact info at the bottom of this page so I can fix it!
      </video>
    </div>

    <div class="half_split_column">
      <video autoplay controls loop muted width="100%">
        <source src="/assets/videos/ns_calibration/fdg.webm" type="video/webm">
        Your browser doesn't seem to support video! If you're using a modern browser, please contact me using the contact info at the bottom of this page so I can fix it!
      </video>  
    </div>
  </div>
  </html>

The most recent code is [here](https://github.com/moshimeow/calibrate-hmd-v3), and I think I have some older stuff [here](https://gitlab.com/slitcch/ns-vipd-calibrator) and [here](https://gitlab.com/slitcch/usb2calibrationtests). All of this stuff was pretty game-jam-quality - I did write it, but it was a long time ago and it's not representative of the kind of work I can do now.

## Created almost all of Monado's North Star driver and Ultraleap driver

The code is here:
* [North Star driver](https://gitlab.freedesktop.org/monado/monado/-/tree/main/src/xrt/drivers/north_star)
* [Optical undistortion evaluation code for North Star](https://gitlab.freedesktop.org/monado/monado/-/blob/main/src/xrt/auxiliary/util/u_distortion_mesh.c)
* [Ultraleap driver (written before I joined Collabora)](https://gitlab.freedesktop.org/monado/monado/-/tree/main/src/xrt/drivers/ultraleap_v2)

## Various other demos
<!-- <details>
<summary>Expand me!</summary> -->

<div class="half_split_row">
  <div class="half_split_column">
    <!-- xrdesktop -->
    <blockquote class="twitter-tweet" data-theme="dark"><p lang="en" dir="ltr">xrdesktop is fun too. Needs a lot of work to be practical for day-to-day use, but it looks cool! <a href="https://t.co/yMtxar766A">pic.twitter.com/yMtxar766A</a></p>&mdash; moshi turner (@moshimeowshiVR) <a href="https://twitter.com/moshimeowshiVR/status/1381562529521020930?ref_src=twsrc%5Etfw">April 12, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
  </div>
  <div class="half_split_column">
    <!-- intel d435  -->
    <blockquote class="twitter-tweet" data-theme="dark"><p lang="en" dir="ltr">WebXR running on <a href="https://twitter.com/MortimerGoro?ref_src=twsrc%5Etfw">@mortimergoro</a> <a href="https://twitter.com/webkit?ref_src=twsrc%5Etfw">@webkit</a> on Linux with <a href="https://twitter.com/MonadoXR?ref_src=twsrc%5Etfw">@MonadoXR</a> runtime, <a href="https://twitter.com/ultraleap_devs?ref_src=twsrc%5Etfw">@ultraleap_devs</a> hand tracking and <a href="https://twitter.com/NorthStarXR?ref_src=twsrc%5Etfw">@NorthStarXR</a>&#39;s headset! <a href="https://t.co/tzGfdSbDrC">pic.twitter.com/tzGfdSbDrC</a></p>&mdash; moshi turner (@moshimeowshiVR) <a href="https://twitter.com/moshimeowshiVR/status/1386899099522326529?ref_src=twsrc%5Etfw">April 27, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>  
  </div>
</div>


<!-- WebXR -->

<div class="half_split_row">
  <div class="half_split_column">
    <!-- WebXR -->
    <blockquote class="twitter-tweet" data-theme="dark"><p lang="en" dir="ltr">Fun with the Intel Realsense D435 <a href="https://t.co/6i0mt9rGf0">pic.twitter.com/6i0mt9rGf0</a></p>&mdash; moshi turner (@moshimeowshiVR) <a href="https://twitter.com/moshimeowshiVR/status/1422186274987859970?ref_src=twsrc%5Etfw">August 2, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 
  </div>
  <div class="half_split_column">
    <!-- intel d435  -->
  </div>
</div>


<!-- </details> -->
---------------------

# Various other contributions to Monado

At Collabora, I have contributed a lot to Monado! Apparently I'm up to 122 merge requests as of March 2023! You can see them [here](https://gitlab.freedesktop.org/monado/monado/-/merge_requests?author_username=slitcch&scope=all&state=merged)
as well as a cool graph which has deemed me the fourth-biggest contributor [here.](https://gitlab.freedesktop.org/monado/monado/-/graphs/main?ref_type=heads)

Off the top of my head I've

* Written a frameserver driver for DepthAI cameras
* Added various improvements to the Vive/Index and WMR drivers, adding support for hand tracking and more config options
* Worked a ton on Monado's device setup code
* Worked on Monado's camera calibration and frame streaming infrastructure
* Created a hardware+undistortion driver for SimulaVR's upcoming HMD
* Worked on Monado's debug UI
* Worked on Monado's internal auxiliary library and math code
* Added basic euro filtering as a Monado util
* Added a pose history util to let us correctly interpolate through past poses as a runtime
* Helped review various OpenXR extensions

---------------------

# StereoKit

I've made a lot of contributions to StereoKit over the years, using it all the time for internal tools/demos.
You can see my code contributions [here](https://github.com/StereoKit/StereoKit/pulls?q=is%3Apr+author%3Amoshimeow) :D

---------------------

# Google Summer of Code 2022 mentor and OpenGloves

During the summer of 2022, I mentored Dan Willmott of LucidVR/OpenGloves as he added OpenGloves support to Monado.

You can see his blog post about it [here](https://blog.dan-w.com/blog_post/2022/09/09/gsoc-2022-monado.html) and his Monado merge requests are [here.](https://gitlab.freedesktop.org/monado/monado/-/merge_requests?scope=all&state=all&author_username=danwillm)

We also met up in the UK and built a pair of gloves for myself:

![gloves](../assets/images/dan.jpg){:height="400"}

This project as well as his earlier work landed him an internship at Valve where he still works, which has been pretty cool to see :)

---------------------

# OpenComposite

I've contributed a few times to OpenComposite, a translation layer that lets you run OpenVR games on OpenXR runtimes without needing SteamVR in the loop. My opinions about SteamVR have changed over the years, but OC is a great tool and at least VRChat and Beat Saber work great with it.

A while ago I wrote a blog post about some of that work [here.]({% post_url 2022-07-16-got-opencomposite-working %}) and you can see all my MRs so far [here](https://gitlab.com/znixian/OpenOVR/-/merge_requests?scope=all&state=all&author_username=slitcch) :)

---------------------
