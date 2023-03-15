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

This is by far the coolest thing I've done so far. This is a competent, reasonably low-latency, accurate and smooth optical hand tracking system, built from scratch (not on top of Mediapipe, although [I tried that too](https://www.collabora.com/news-and-blog/blog/2022/05/31/monado-hand-tracking-hand-waving-our-way-towards-a-first-attempt/)) with a completely open-source implementation and dataset. 
It also works on pretty much any VR headset with calibrated cameras.

Many considered this problem way too hard and best left to the likes of Ultraleap and Oculus, but we said no way to that!

At Collabora I've done maybe 90% of the relevant work: scraping, collecting and generating training data, writing and wrangling machine learning training infrastructure, writing realtime inference code in C++, a realtime nonlinear optimizer to get a smooth 3D hand pose trajectory, and various infrastructure to deal with camera streaming, camera calibration, distortion/undistortion, etc.

This took me about a year, starting right after I graduated college.
I was a pretty huge novice to computer vision, machine learning and algorithms, but I made it happen through hard work and being completely unafraid to learn new things!


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

- Two key components: hand detector and keypoint estimator
- Hand detector doesn't always run every single frame and it's a little bit inaccurate - this is *on purpose* to lower compute
- Keypoint estimator runs in every frame in every view for every hand. 


<html>
      <img src="/assets/images/architecture.jpg" alt="steam" width="100%">
</html>

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

- Jakob Bornecrantz for helping with general Monado infrastructure stuff and always reviewing my code.
- Ryan Pavlik for pointing me to the right resources for learning C++, C++ template metaprogramming, and Eigen3. And for helping me learn various tracking algorithm fundamentals.
- Mateo de Mayo for several key good ideas, writing a lot of Monado's shared HT/SLAM camera pipelining and dataset recording infrastructure, and encouraging me to try realtime nonlinear optimization.
- Marcus Edel and Jakub Piotr Cłapa for helping with the very early ML stuff and encouraging me to take the time to really learn PyTorch.
- Nikitha Garlapati and Hampton Moseley for giving me a hand with training data.
- Christoph Haag, Lubosz Sarnecki, Pete Black and all the other Monado contributors for giving me such a solid base of hardware drivers and infrastructure to build on top of.

---------------------

# StereoKit

I've made a lot of contributions to StereoKit over the years, using it all the time for internal tools/demos.
You can see my code contributions [here](https://github.com/StereoKit/StereoKit/pulls?q=is%3Apr+author%3Amoshimeow) :D

---------------------

# Project North Star

## Designed and built a custom North Star variant

As of 2023, this is the only North Star headset that does all tracking on a single stereo pair.

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

For this project, I did all the 
## Created a custom optical calibration method for North Star HMDs

I iterated a _lot_ on this, picking up traditional camera calibration methods, edge detection, thresholding, marker detection, etc. along the way.

The main focus of my work here was _empirical_ calibration, where you place a calibrated stereo camera inside of a HMD where the user's eyes would be, draw a pattern on the display, and just save the mapping from display UV to tanangles as a mesh.

If I recall correctly, I tried (sequential order):

- Canny edge detection - had big issues with bloom
- first, switched to thresholding (drawing one line at a time on the vertical and horizontal axes)

(right before I discovered nonlinear optimization - this could have been so much better)

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
    </video>  </div>
 </div>
</html>

<html>
    <video autoplay controls loop muted width="100%">
      <source src="/assets/videos/ns_calibration/lines.webm" type="video/webm">
      Your browser doesn't seem to support video! If you're using a modern browser, please contact me using the contact info at the bottom of this page so I can fix it!
    </video>
</html>

The code is [here!](https://github.com/moshimeow/calibrate-hmd-v3) It works, although it's pretty game-jam quality!

TODO You should also backlink to your old repos
## Created almost all of Monado's North Star driver and Ultraleap driver

TODO Backlink to u_distortion_mesh d/ns d/ulv2

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

# Various SLAM stuff

This is near the bottom of the list not because I think SLAM/head tracking is unimportant, but simply because fate hasn't crossed my paths with SLAM work very often.



Okay, so I... couldn't really find the reciepts for all of this this - they're buried in the depths of some Discord server and I don't really have time to go look. But yes, in 2021 I helped out a bunch with searching for and evaluating

---------------------
# Uncategorized stuff

At Collabora, I have contributed a lot to Monado! Apparently I'm up to 122 merge requests as of March 12 2023! You can see them [here](https://gitlab.freedesktop.org/monado/monado/-/merge_requests?author_username=slitcch&scope=all&state=merged)
as well as a cool graph which has deemed me the fourth-biggest contributor [here](https://gitlab.freedesktop.org/monado/monado/-/graphs/main?ref_type=heads)
