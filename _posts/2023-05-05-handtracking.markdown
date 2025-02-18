---
layout: post
title:  "Hand tracking stuff"
date:   2023-05-05 00:00:00 +0100
categories: update
---

I'm posting this publicly a) so my coworkers can easily get to it and b) so that I have some record of what's already been "invented" before I start work at my new employer.


- Things I tried:
	- Optimizer
		- in late 2021, I was thinking about using some kind of kalman filter to fit 3D hands to 2D observations. I realized that the non-rigidity of hands made this really challenging - you'd need a whole bunch of iterations per frame. Slowly started to learn about
		- in late 2021, I started writing an implementation from scratch using finite differences to estimate gradients fitting a 3D hand skeleton to unconstrained triangulated points. This was a really cool way to learn things but definitely wasn't ever gonna be performant
		- In early 2022, I used Cyclic Coordinate Descent IK to fit a constrained 3D skeleton to unconstrained triangulated points. CCDIK is basically an off-the-shelf IK method to do IK with specific joint constraints (so as to not let all the joints bend in random directions.) The main innovation I made was figuring out how to do IK with a root bone that can move and more than one "end effector." Basically regular IK can't move the root bone and just rotates the joints so that the last bone in the chain matches a target, but you can do it using hacks.
			- The way it works is this:
				- for i in range(idk 30):
					- Use Umeyama to move the root bone to bring the model joints as close as possible to the observed joints
					- From root to tip, do one CCDIK iteration (adding a rotation and constraining it) where the end effector and target is not the *tip* of that bone, but the centroid of all the joints "downstream" of this bone
			- Some links that really helped me:
				- http://number-none.com/product/IK%20with%20Quaternion%20Joint%20Limits/
				- https://zalo.github.io/blog/inverse-kinematics/
			- This code exists in Monado's history - git grep for `ccdik`, you'll find it.
		- In mid 2022, I wrote the Ceres optimizer that's stuck around for a long time.
			- This one doesn't explicitly enforce hard constraints for joint rotations, it just uses LMToModel/ModelToLM to ensure that the only *possible* values are within constraints. We could have not done that and instead had an extra residual for "one of ur joints is exceeding rotation limits" but that would have induced some instability and overshoot in the LM implementation
			- One tip: simplify your problem space! I started with Eigen's finite diff and Eigen's LM implementation, and tried dumb simple things like the 2D links example or aligning two pointclouds before attempting the real problem.
	- Hand detector:
		- I started with trying to use YOLO at Marcus's recommendation. I don't recommend it.
			- Small rant about YOLO detectors:
				- the whole thing's been a fucking sham since YOLO v3. Joseph Redmon developed YOLO/Darknet, then stopped because he realized general-purpose box detectors were useless for basically everything other than government surveillance. Then a bunch of the worst AI grifters realized that YOLO wasn't trademarked or anything, so they made YOLO v4-v8 (!!!), none of which are any good. No iteration made a fundamental change - they all only eked out tiny improvements on the official ImageNet test set that aren't meaningful in the real world.
				- Why box detectors are only useful for surveillance:
					- For hand tracking, you only have two objects in total you're trying to detect.
					- For other VR tasks, box detection is useless (except for I guess very primitive "let's detect if one of your pets has stepped into your playspace" models which actually wouldn't ever run embedded because they'd take compute away from more important tasks.) For VR scene understanding, 2D segmentation is pretty useful, otherwise you want a 3D-aware model that can make a semantic mesh of your room, which boxes don't help with.
					- For robots/cars you basically need 3D semantics and boxes are very unhelpful except for as a very primitive safety mechanism.
					- HOWEVER, box detectors are pretty great at tracking lots of people or cars on security footage.
			- Anyway. I had lots of trouble getting the official YOLO repositories to work with grayscale images and I realized I didn't need a box detector, so I gave up on that.
			- Then I tried applying the model architectures behind SimDR https://github.com/leeyegy/SimCC to hand detection and that worked good. (I modded the model architectures to do 2D heatmaps again, not just 1D.) For a while we just had two heatmaps for hand center and two for hand size, that worked great.
			- That model was pretty slow though, so I ended up switching to something that does direct regression, actually exactly the same model arch that FB uses: https://github.com/milkcat0904/MegaTrack-pytorch. We're still using that, that's great. It just regresses 8 numbers: cx, cy, size, "whether the hand is visible" for the left and right hand.
	- Keypoint estimator *architecture*:
		- At first I tried some SimDR-y things, they worked OK but were slow. It was kind of interesting to see that the architecture itself was inefficient - the main problem is that we had a bunch of blocks running in parallel ..that we wanted to run in a single thread. In general for embedded stuff this fast you don't need/want parallel blocks, you just want your backbone to be a bunch of linear inverted residual blocks.
		- Eventually I also just used the backbone from https://github.com/milkcat0904/MegaTrack-pytorch with some minor changes to let us use a different input-predicted-keypoints size and extra scalar outputs.
- Stuff I want to try next:
	- Estimate variances in the optimizer so we can reduce jitter
	- Right now our keypoint estimator model can only see one view at a time - we have one instance per view. This is dumb because:
		- This makes us redo a lot of the computation per view - the views aren't *that* different and if we could make a model that could "think" about both views at once it shouldn't need to do as much work
		- This means that each instance of the model has less information about parts of the hand that are occluded in one camera but not in the other, and their estimates are worse than they could be
	- Recurrent architectures. Your hand's pose doesn't change much between frames, so you can save on computation by having your model already know about what's going on. (We do this sorta by feeding the last two frames' predicted keypoints, but it's not a good solution and you can do way better)
		- Also, what if we could make a metric of how "interesting" the last N frames were, and throw out boring ones so that we can save compute?
	- Quantized Mixture of Experts! Have several instances of the model backbone, and early-on in the model decide which instance of the model we're going to use. During inference we use only one, but during training all of them run and you do the loss function between each instance's output and ground truth, *multiplied by* that model's weight. That way you don't get crazy interference patterns in the model output, but just the one that happens to be used less during that run doesn't have to be as accurate.
		- Some things:
			- You can use something similar to batch-norm to force the expert-selector part of the model to always pick each one an even amount, so that you don't just end up with one model that's really good that always gets picked, then the other ones never get good.
			- Also, MoE will make the model *way* better at overfitting to artificial data, so you should also add a greeble to make sure that it doesn't select one expert more often for artificial data and another more often for real data.
- Hardest tech problem: datasets!
	- A lot of the stuff I want to try next model-wise is not possible yet because models can overfit real bad to artificial data and we don't have (really any) sequential real data. You can fix this either by making more realistic artificial data, or collecting+annotating some sequential real data.
- Principles/Glossary:
	- "Swing" rotation - a rotation that rotates one vector to point in the same direction as another vector while taking the shortest path across the unit circle and not inducing any twist. For our tracking, this is equivalent to an angle-axis rotation where the Z axis is 0. This was useful in the CCDIK optimizer as a way to explicitly enforce constraints (we'd take an affine matrix, convert it to swing/twist, constrain that value, turn it back into affine), and it's useful in the LM optimizer info (I explained this to Ryan, ask him)
	- "Swing-twist" rotation - a "swing" rotation, then you apply a "twist" afterwards along (usually) the Z axis of the new orientation
	- We want our keypoint estimator to be multimodal - able to track hands in all configurations, but we want it to be *unimodal* across tracking synthetic-data hands and real-data hands. Basically everywhere we can we do lots of interesting hacks to make sure it has a hard time distinguishing real data from synthetic data.