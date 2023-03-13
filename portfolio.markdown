---
layout: post
title: portfolio!
permalink: /portfolio/
---

THIS IS A DRAFT. IF YOU ARE IMPORTANT DON'T LOOK AT THIS YET.

Hi!

This portfolio is something I use to showcase work I've done in the past.

You'll notice some lack of polish on this - this is due to time pressure and my expectation of a mostly-engineer audience who will focus on the technical aspects instead of the presentation. And I'm certainly not trying to show off my skill as a web designer here 😅

This is ordered by "significance of technical achievement" - the things I'm most proud of are at the top, and the honorable mentions are at the bottom :)

---------------------

# Monado's hand tracking

This is by far the coolest thing I've done so far.

## Full-system demo

<html>
    <video autoplay controls loop muted width="100%">
      <source src="/assets/videos/mercury_demo_3.webm" type="video/webm">
      Your browser doesn't seem to support video! If you're using a modern browser, please contact me using the contact info at the bottom of this page so I can fix it!
    </video>
</html>

([Twitter link](https://t.co/zitKhc66Yp))


These are through-the-lens (or through-the-virtual-left-eye for VR headsets) recordings

## Artificial dataset generator

<html>
    <video autoplay controls loop muted width="100%">
      <source src="/assets/videos/hand_grid_loop_long.webm" type="video/webm">
      Your browser doesn't seem to support video! If you're using a modern browser, please contact me using the contact info at the bottom of this page so I can fix it!
    </video>
</html>

I got some hand scans from a well-known photogrammetry asset store,

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

---------------------

## StereoKit

I've made a lot of contributions to StereoKit over the years, using it all the time for internal tools/demos.
You can see my code contributions [here](https://github.com/StereoKit/StereoKit/pulls?q=is%3Apr+author%3Amoshimeow) :D

---------------------

## Project North Star

### Built a custom North Star variant, designed from scratch in Blender, 40 iterations

### Built a custom optical calibration method (right before I discovered nonlinear optimization - this could have been so much better)

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

### Worked on SLAM stuff

--------------

## GSoC '22 mentor and OpenGloves

During the summer of 2022, I mentored Dan Willmott of LucidVR/OpenGloves as he added OpenGloves support to Monado.

You can see his blog post about it [here.](https://blog.dan-w.com/blog_post/2022/09/09/gsoc-2022-monado.html)

We also met up in the UK and built a pair of gloves for myself:

![gloves](../assets/images/dan.jpg)

## OpenComposite

I've contributed a few times to OpenComposite, a translation layer that lets you run OpenVR games on OpenXR runtimes without needing SteamVR in the loop. My opinions about SteamVR have changed over the years, but OC is a great tool and at least VRChat and Beat Saber work great with it.

A while ago I wrote a blog post about some of that work [here.]({% post_url 2022-07-16-got-opencomposite-working %}) and you can see all my MRs so far [here](https://gitlab.com/znixian/OpenOVR/-/merge_requests?scope=all&state=all&author_username=slitcch) :)

## Uncategorized stuff

At Collabora, I have contributed a lot to Monado! Apparently I'm up to 122 merge requests as of March 12 2023! You can see them [here](https://gitlab.freedesktop.org/monado/monado/-/merge_requests?author_username=slitcch&scope=all&state=merged)
as well as a cool graph which has deemed me the fourth-biggest contributor [here](https://gitlab.freedesktop.org/monado/monado/-/graphs/main?ref_type=heads)

---------------------
---------------------
---------------------
---------------------
---------------------
---------------------
---------------------
---------------------
---------------------
---------------------
---------------------
---------------------
---------------------
---------------------
---------------------
