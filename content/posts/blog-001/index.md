---
title: "Ramblings on Heat and Efficiency"
date: 2026-08-01
draft: false
slug: "bldc-motor-heat-losses-efficiency"
aliases: ["/posts/blog-001/"]
math: true
showToc: true
TocOpen: false
tags: ["motors", "electronics", "engineering"]
categories: ["blog"]
description: "Breaking down every source of heat in a BLDC motor — copper, core, and mechanical losses — using the ODrive D5065 as a model, and building toward an efficiency map."
---

Today, I'll be showing all the sources of heat within a BLDC motor. We'll be using the ODrive D5065 motor as our model.

To start, there are 3 types of heat present in a motor at any time: core losses, copper losses, and mechanical losses.

![Taxonomy of the three heat sources in a BLDC motor: iron, copper, and mechanical losses](figures/loss_taxonomy.png)

Starting with copper losses, you can have heat loss due to the motor demanding current from the electronic speed controller (ESC), which would represent DC current loss as described by $P = I^2 R$. Additionally, as a result of the motor requiring 3-phase power to work, we also have skin effect and proximity effect, the effects of which increase greatly at higher frequencies.

Core losses primarily occur within the stator. Heat is generated here by way of hysteresis loss, which is the heat generated from all the alternating magnetic waves present in a motor, and eddy current losses, which are small current loops created within the stator that cause heat.

Lastly, mechanical heat is the easiest to imagine. The rotor and shaft spin as one body; the stator is bonded to a stationary hub, and two bearings bridge the two. Nothing slides, but rolling contact isn't free: ball-race contact friction, grease shear, and seal drag all extract torque, and grease drag grows with speed. Separately, windage is the aerodynamic loss from air being dragged through the gap and around the spinning bell.

Our final goal here is to simulate and graph all these individual sources of heat and see how they change as we adjust torque and RPM. By plotting the combined influence of power losses, and calculating power output as a function of torque and speed with $P = T\omega$ where $\omega = \text{RPM} \cdot 2\pi / 60$, we can determine input power as a function of torque and speed with $P_{in} = P_{out} + P_{loss}$.

Ultimately, the goal is to take input power and output power graphs to determine the efficiency of the motor when driven at a particular operating point. This is helpful to visualize a particular motor's performance.

## Assumptions

There are still some heat sources I'm not taking into consideration.

* Magnetic eddy currents are current loops that can occur within the permanent magnets on a rotor. I'll be modeling this heat in another post, but for now we'll assume its contribution is 0 across all parameters.
* PWM rippling - due to the rapid switching from the ESC, small fluctuations of current occur when fed into the motor. This results in ripple current copper loss, and ripple-flux core loss.
* Eddy current heating in the rotor itself
* Magnet thermal feedback loop - increase in magnet heat leads to decrease in field flux density, which leads to lower iron loss and torque constant. Therefore, more copper losses for the same output torque. Then the loop begins when copper losses increase again.
* Bearing loss - Real bearings have grease-viscosity temperature dependence.
* Drive electronics heat - the MOSFETs that compose the ESC are also sources of heat loss.

All these limitations will be tackled some day, but for now they will remain un-modeled.

## Tools

* FEMM - used to model geometry of motor and simulate core flux density, slot area, and mass
* pyFEMM - used to script FEMM
* Python - used to calculate heats and graphs

## Geometry

Here's a screenshot of the motor I modeled in FEMM with its flux density map. Max flux density is 1.845 T, which is very large. It's possible that the value came from a tooth tip or a corner mesh artifact. We used a 14P12S configuration with 8 turns per slot. 12AWG circular wire is used. 

![FEMM mesh and |B| flux density plot of the stator/rotor cross-section, peaking at 1.845 T](figures/femm-flux-density.png)

## Copper losses

Copper losses are one of the more significant heat sources in a motor. This is because for each additional amp of current, heat scales with the square of current: double the current, quadruple the loss. 

For a BLDC motor, this is a big issue because if you want to do anything useful you need high current to drive the motor hard. Motors are one of the more current-hungry electronic components out there. 

Copper losses can be modeled as $P_{cu} = 3 I^2 R$, for the copper loss in all 3 phases. To create the graph, we're going to simulate running the motor at a particular torque and speed, then determine the power loss.

![DC copper loss vs. torque and speed](figures/dc_copper_loss.png)

The graph is a bunch of horizontal stacks. The magnitude doesn't change as you vary speed, it stays constant. Torque on the other hand changes has a large impact, and copper losses dramatically scales from 0 to 210 W. This is because torque depends on current to increase. You want more torque, you need a lot of current, and current incurs heat losses. Speed of rotation depends on voltage, which would not incur copper losses at DC.

AC copper losses can be thought of as the decrease of effective conduction area in a wire as frequency/RPM increases. This area decrease is due to skin and proximity effects. 

Skin effect is a result of self-induced eddy currents within a wire's own body that go against the actual DC current. It directly opposes real current flow in the center so much so that the actual conducting area of the wire becomes shaped like an annulus.

![Skin effect: self-induced eddy currents push current density toward the conductor's surface, shrinking the effective conducting area to an annulus](figures/skin-effect.jpg)

Resistance of a wire is inversely proportional to its area; by decreasing area, you increase resistance, thus increasing losses since current would be held constant. $R = \frac{\rho L}{A}$

A current in a smaller area will produce more heat than that same current in a larger area. The rate at which this area decreases can be calculated with skin depth, where $\delta = \sqrt{\rho / (\pi f \mu_0)}$ — the depth over which current density decays by $1/e$ due to the conductor's own AC field pushing current toward its surface.

Proximity effect is essentially the same phenomenon but from a neighboring wire instead. Induced eddy currents from neighboring coils oppose real current flow. Depending on the orientation of this external wire, the area can skew and shrink towards the left or right edge of the wire. 

![Proximity effect: induced eddy currents from a neighboring conductor push current density toward one edge of the wire](figures/proximity-effect.jpg)

The impact of all this effective shrinkage is accounted for in computing the Dowell ratio $R_{ac}/R_{dc}$, a dimensionless number. This is what we measure changing in our graph this time, instead of pure watts.

![AC-to-DC resistance ratio (Dowell ratio) vs. frequency/RPM](figures/ac_ratio.png)

Vertical bands make sense here, Dowell ratio depends on frequency/RPM. If the Dowell ratio goes up that means the AC resistance goes up. If you wanted to actually get AC resistance, you multiply the Dowell ratio by the DC resistance. However, the ratio is so close to unity that the product of the ratio with any DC resistance is basically unchanged. In fact, AC resistance would only contribute a few percent of the total heat loss. It is nearly negligible.


## Core losses

Hysteresis losses and eddy current losses compose core losses in the stator.

Hysteresis loss is the heat generated when you repeatedly magnetize and demagnetize a metal. This can be seen in a hysteresis loop.

![Hysteresis loop for soft vs. hard ferromagnetic material](figures/hysteresis-loop.jpg)

As you increase magnetizing force (x-axis), the field density increases (y-axis). Once you reach the corner, you've reached saturation. If you follow the path of the graph counterclockwise, you can see that our magnetizing force has decreased to 0 but its flux density (B) is still high. To bring the magnet back to its original un-powered state, you have to reverse your magnetizing force in the opposite direction so that $B = 0T$ once again. The cost of doing this exerts heat because moving magnetic domains within a magnet to align and realign repeatedly causes a lot of magnetic friction. Faster switching of magnets equals more loss in the core. So hysteresis loss increases with speed as seen in $P_{hyst} = k_h \cdot f \cdot B_{pk}^{\alpha}$, where $B_{pk}$ is the 1.845 T peak flux density pulled from the FEMM sim above.

Eddy current losses in the stator are the reason stators are manufactured with thin silicon steel sheets with a varnish coating. Eddy currents are produced inside the stator, and they get stronger with frequency increase. By splitting stators into sheets, eddy currents are split into smaller components. Since power loss scales with current, decrease in current is greatly appreciated, as seen in $P_{eddy} = k_e \cdot f^2 \cdot B_{pk}^2$.

![Core loss (hysteresis + eddy current) vs. torque and speed](figures/core_loss.png)

## Mechanical loss

Windage is the aerodynamic drag acting on the spinning rotor. This loss is very minuscule and only increases at top speeds. The relationship to speed is quadratic as seen in $P_{windage} = k_{windage} \cdot \omega^2$.

Bearing friction is modeled as a constant 0.2W. Friction completely dominates mechanical loss.

![Mechanical loss (windage + bearing friction) vs. torque and speed](figures/mech_loss.png)

## Total power analysis and motor efficiency

Let's add up all the losses in power we have found.

![Total power loss vs. torque and speed](figures/total_loss.png)


Now, let's get the power output by multiplying torque and angular speed.

![Output power vs. torque and speed](figures/output_power.png)

Input power is then given by $P_{in} = P_{out} + P_{loss}$, so just sum the graphs.

![Input power vs. torque and speed](figures/input_power.png)

So we have the input power, we have the output power. Efficiency is simply the measure of the ratio between those two quantities. $\eta = P_{out}/P_{in} = \tau\omega/VI$. $\tau\omega$ are the output metrics and $VI$ are the input metrics. Together we get this.

![Motor efficiency map vs. torque and speed](figures/efficiency_map_clean.png)

Looking at the graph our most efficient contour sits at ~85% at around 0.2 N·m, if you move to the right, you can drive your motor at higher RPM's without increasing torque. If you travel up instead, you are able to increase your torque by ~0.2 N·m for no sacrifice in efficiency either. 

## Conclusion

Taking a look at the legends for all these graphs, we can see that copper losses are the majority of heat, followed by core and mechanical. The other sources of heat we assumed to be zero are also very insignificant compared to copper losses. Taken from the graphs, we have a table of values as torque varies.

| Torque [N·m] | $P_{cu}$ [W] | $P_{fe}$ [W] | $P_{mech}$ [W] | Winding temp [°C] | Efficiency |
|-------------:|-------------:|-------------:|---------------:|------------------:|-----------:|
| 0.1          | 1.02         | 6.80         | 0.20           | 27                | 79.1%      |
| 0.2          | 4.10         | 6.80         | 0.20           | 28                | 84.6%      |
| 0.3          | 9.28         | 6.80         | 0.20           | 30                | 84.9%      |
| 0.4          | 16.63        | 6.80         | 0.20           | 32                | 83.7%      |
| 0.6          | 38.32        | 6.80         | 0.20           | 38                | 80.1%      |
| 0.8          | 70.50        | 6.80         | 0.20           | 47                | 75.8%      |

Core and mechanical loss stay essentially fixed as torque increases (they track speed, not torque), so copper loss goes from a fifth of total heat at light load to over 90% of it near peak torque. That crossover is what makes copper the loss to optimize for once you push beyond light-duty operation.

Another table from our graphs, we vary speed here instead and hold torque constant at 0.3 N·m.


| Speed [RPM] | $f_{\text{elec}}$ [Hz] | $P_{cu}$ [W] | $P_{fe}$ [W] | Winding temp [°C] | Efficiency |
|------------:|-----------------------:|-------------:|-------------:|------------------:|-----------:|
| 200         | 23                     | 9.21         | 0.25         | 28                | 39.4%      |
| 1000        | 117                    | 9.22         | 1.57         | 28                | 74.1%      |
| 2000        | 233                    | 9.25         | 3.95         | 29                | 82.4%      |
| 2904        | 339                    | 9.28         | 6.80         | 30                | 84.9%      |
| 4000        | 467                    | 9.32         | 11.15        | 31                | 85.9%      |
| 5808        | 678                    | 9.42         | 20.45        | 34                | 85.9%      |

Copper loss barely moves with speed at fixed torque. Core loss is what climbs, from negligible to over 20W. 2x the copper loss by top speed. Efficiency rises fast as output power ramps up, peaks around 4000 RPM, then edges back down as core loss growth outpaces the gain in output power.

There's a myriad of techniques to minimize copper losses:

* Special stacking methods when winding stator
* Use square wires to improve slot factor and minimize air gaps
* Litz wiring 

Some more ways to decrease other heat and increase efficiency:

* Skewing magnets for smoother rotation
* Splitting magnets in half and skewing
* Backplate to concentrate magnetic fields inside motor
* Better magnets like N52
* Use [FOC control]({{< relref "blog-002" >}}) for smoother torque
* PWM tuning to reduce current rippling
* Silicon content in the electrical steel is a tradeoff, not a knob: more silicon raises resistivity and cuts eddy loss, but lowers saturation flux density. High-silicon grades favor high-speed operation; lower-silicon grades favor torque density.

## What this means for me

I'm working on a [robotic actuator]({{< relref "blog-003" >}}) right now and there were too many factors when considering buying a motor. Understanding how to do all these thermal simulations has been illuminating and I can now comfortably spec my needs. 

You can check out my scripts and figures at my [Github](https://github.com/JimmerLitter/Motor-Loss-Study)

## Sources

* [ODrive D5065 datasheet](https://docs.odriverobotics.com/v/latest/hardware/odrive-motors.html)
* [BLDC/PMSM efficiency and power basics — Things in Motion](https://things-in-motion.blogspot.com/2019/03/basic-bldc-pmsm-efficiency-and-power.html)
* [BLDC winding tool — H Laboratories](https://www.hlaboratories.com/tools/bldc-winding)
* [The Power of Dowell's Equations and Curves — Ridley Engineering](https://ridleyengineering.com/design-center-ridley-engineering/49-circuit-designs/289-112-the-power-of-dowells-equations-and-curves.html)
* [Understanding Losses in BLDC Motors — Machine Design (Portescap)](https://www.machinedesign.com/mechanical-motion-systems/article/21251043/portescap-understanding-losses-in-bldc-motors)