---
title: "FOC"
date: 2026-08-25
draft: false
showToc: true
TocOpen: false
tags: ["motors", "electronics", "engineering"]
categories: ["blog"]
description: "A quick, chill rundown of field oriented control (FOC): current sensing, the Clarke and Park transforms, and closing the torque loop on a BLDC motor."
---

Field oriented control is one of the predominant ways to maximize torque out of your BLDC motor. Here's what I learned:

Inputs to the control loop are obtained from each inverter stage. The inverters set a voltage that the motor coils respond to, and the response is sensed by shunts placed below each inverter stage's lower leg. This results in 3 currents (Ia, Ib, Ic). The voltage across the shunts is measured and sent to the MCU to be discretized, transformed, and controlled.

In addition to gathering current inputs, a magnet was placed at the shaft of the motor, and a magnet encoder was placed nearby, which senses the magnet's orientation, corresponding to the mechanical angle of the rotor. Through a calibration routine, the encoder is able to track the mechanical angle and send it to the MCU. This mechanical angle then needs to be multiplied by the motor's pole pair count to get the electrical angle, which is what actually gets used in the Park transform. There are also sensorless motor controllers that use more advanced models to compute the angle without sensors or encoders.

The currents sensed from the shunts are AC values from the 3 phases present in the stator. As such, you can sum up all 3 currents to be 0. By knowing 2 currents, you can always figure out the third. The Clarke transform removes this redundant current so that we can deal with a reduced dimension and make calculation easier.

Coming out of the Clarke transform, we send the 2 AC values into the Park transform. Using the encoder output, we can shift the axis given in the Clarke transform by the movement of the rotor. This allows us to capture the rotation and synchronize the frame with the rotor. This essentially converts the AC values to DC, which are easier to deal with. We now have 2 currents: Iq and Id. Iq is proportional to the torque and needs to be optimized to hit the reference. Id needs to be minimized.

Before beginning the loop, you should already have an idea of how much current you want in the motor (`iq_reference`). If starting from rest, it will take time for the current to reach your desired level. That is why we use PI error correction to correct Iq towards the reference and Id towards 0.

Once we have performed error correction, we send the DC values into an inverse Park and then an inverse Clarke to redo all our transformations and send it to the power stage so that the motor can respond to our new PWMs. This basically repeats many times per second until our motor reaches the speed we desired. If using space vector PWM instead, the last inverse Clarke can be omitted.
