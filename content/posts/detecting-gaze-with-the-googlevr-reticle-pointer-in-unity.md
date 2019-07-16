---
title: "Detecting Gaze with the GoogleVR Reticle Pointer in Unity"
date: 2019-07-16T08:07:54+01:00
draft: true
---
One of the situations I often encounter when developing VR software in Unity is detecting gaze. With the GoogleVR SDK for Cardboard and Daydream, there are prefabs that provide gaze based pointers. When combined with an event system and wired up with one or more events, I can detect when a particular game object is being gazed at (providing it has a collider component).

When configured, these components work harmoneously to provide most of the features needed to interact with virtual worlds. However, I regularly find that I have a game object for which I either can't detect gaze or detect gaze at unwanted times.

A recent example is a game of Whack-a-Mole that I was working on for Google Cardboard. When the mole popped its head up out of the ground I was expecting the Reticle Pointer to dilate. But there was nothing. When the mole hid below ground and I looked lower than its mole hole, the pointer dilated. This behaviour was the complete opposite of what I actually wanted.

The solution? Well, the first is relatively simple. I had forgotten to turn off or remove the colliders for the rest of my scene. My world is a simple cylinder coloured green and scaled so that it looks like a circular plane. The mole holes are similarly devised. But they both had colliders extending 2m vertically upwards. Once disabled, I had no problem detecting when the mole's were being gazed at.

A solution to the second unwanted behaviour is perhaps a little tricker. For simplicity, I settled on re-enabling the collider for the mole hole and setting its position to below my scene but covering the hidden mole. Effectively, its the inverse of problem one but now working for me rather than against me.

I don't particularly like the solution to problem two, however. It feels like a bit of a hack. Plus, Unity has fantastic layer and tagging systems that can be combined effectively with raycasting. Layers are particularly effective given that they can be used to build a layer mask which is passed as an argument to the raycast.

Since I'm pretty certain that the `GvrReticlePointer` uses raycasting underneath, I'm going to investigate whether I can specify a layer mask.