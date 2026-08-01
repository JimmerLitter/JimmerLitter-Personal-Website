**Ramblings on heat and efficiency**

Today, I’ll be showing all the sources of heat within a BLDC motor. We’ll be using the Odrive D5065 motor as our model. 

To start, there are 3 types of heat present in a motor at any time: Iron losses, Copper losses, and mechanical losses.  
Loss\_taxonomy.png

Starting with copper losses, you can have heat loss due to the motor demanding current from the electronic speed controller(ESC) which would represent dc current loss as described by P \= I^2 \* R. Additionally, as a result of the motor requiring 3-phase power to work, we also have skin effect and proximity effect. The effects of which increase greatly at higher frequencies. 

Core losses primarily occur within the stator. Heat is generated here by way of hysteresis loss, which is the heat generated from all the alternating magnetic waves present in a motor, and eddy current loss, which are small current loops created within the stator that creates heat. 

Lastly, mechanical heat is the easiest to imagine. The touchpoints between the stator, bearings, and shaft are all contact points that rub against one another with nonzero friction. As a result, no matter how precise the machining, there will be friction losses, especially at high speeds. Finally, windage is waste heat caused by air drag and turbulence as air moves within the motor. 

Our final goal here is to simulate and graph all these individual sources of heat and see how they change as we adjust torque and RPM. By plotting the combined influence of power losses, and calculating power output as a function of torque and speed with P \= T \* w where w \= RPM \* 2pi / 60, we can determine input power as a function of torque and speed, Pin  \= Pout \+ Ploss.

Ultimately, the goal is to take input power and output power graphs to determine the efficiency of the motor when driven at a particular power and speed. This is helpful to visualize a particular motor’s performance.

**Assumptions**  
There are still some heats I'm not taking into consideration.

*  Magnetic eddy currents are current loops that can occur within the permanent magnets on a stator. I’ll be modeling this heat in another post but for now we’ll asume its contribution is 0 across all parameters.   
* PWM rippling \- due to the rapid switching from the esc, small fluctuations of current occur when fed into the motor. This reuslts in ripple current copper loss, and ripple-flux core loss.   
* Eddy current heating in the rotor itself   
* Magnet thermal feedback loop \- increase in magnet heats leads to decrease in field density, which leads to iron loss and torque constant loss. Therefore, copper losses for the same output torque rises.   
* Bearing loss \- We mentioned bearing loss but it was modeled as a constant. Real bearings have grease-viscosity temperature dependence.   
* Drive electronics heat \- the mosfets that compose the esc are also sources of heat loss.   
* Temperature rise \- Temperature is assumed to stay ambient, of course this isn’t true in reality. Modeling properly requires feedback loops

All these limitations will be tacked some day, but for now they will remain unacknowledged. 

**Tools**

* FEMM \- used to model geometry of motor and simulate core flux density, slot area, and mass  
* Pyfemm \- used to script FEMM  
* Python \- used to calculate heats and graphs

**Copper losses**

Copper losses are one of the more significant heat sources in a motor. This is because for each additional amp of current, the heat squares in magnitude. For a BLDC motor, this is a big issue because if you want to do anything useful you need high current to drive the motor hard. Motors are one of the more current hungry electronic components out there. Copper losses can be modeled as Pcu \= 3 \* I^2 \* R, for the copper loss in all 3 phases. To create the graph, we’re going to simulate running the motor at a particular torque and speed, then determine the power loss. 

Dc\_copper\_loss.png

As you can see, the graph is a bunch of horizontal stacks. The magnitude doesn’t change as you vary speed, it stays constant. Torque on the other hand changes dramatically and scales from 0 to 210 W in losses. This is because torque depends on current to increase. You want more torque, you need a lot of current, and current increases heat losses by a square factor. Speed of rotation depends on voltage, which would not increase copper losses at dc. 

Now let’s consider AC copper effects. 

AC copper losses can be thought of as the decrease of effective conduction area in a wire as frequency/rpm increases. This area decrease is due to skin and proximity effects. Skin effect is a result of self induced eddy currents within a wires own body that goes against the actual dc current. It directly opposes real current flow in the center so much so that the actual conducting area of the wire becomes shaped like an annulus. 

Insert a png of skin effect

As you can see, the area is much smaller than before. Resistance of a wire is proportional to its area, by decreasing area, you decrease resistance, thus increasing losses since current would be held constant. A current in a smaller area will produce more heat than that same current in a larger area. The rate at which this area decreases can be calculated with skin depth, where δ \= sqrt(ρ / (π f μ₀)) — the depth over which current density decays by 1/e due to the conductor's own AC field pushing current toward its surface.

Proximity effect is essentially the same phenomenon but from another wire instead. Induced eddy currents from neighboring coils oppose real current flow. Depending on the orientation of this external wire, the area can skew and shrink towards the left or right edge of the wire. The point is that the area shrinks anyways. 

The impact of all this effective shrinkage is accounted for in computing the dowell ratio R\_ac/R\_dc, a dimensionless number. This is what we measure changing in our graph this time, instead of pure watts. 

ac\_ratio.png

Vertical bands make sense here, Dowell ratio depends on frequency/RPM. If the Dowell ratio goes up that means the ac resistance goes up. If you wanted to actually get AC  resistance, you multiply the Dowell ratio by the DC resistance.   
**Core losses**

Hysteresis loss and eddy current loss compose core losses in the stator. 

Hysteresis loss is the heat generated when you repeatedly magnetize and demagnetize a metal. This can be seen in a hysteresis loop. 

Insert a hysteresis loop png.

As you increase magnetizing force(x-axis), the field density increases(y-axis). Once you reach the corner, you have saturated the magnet and created a permanent magnet. If you follow the path of the graph counterclockwise, you can see that our magnetizing force has decreased to 0 but its flux density(B) is still high. To bring the magnet back to its original unpowered state, you have to reverse your magnetizing force in the opposite direction so that B \= 0T once again. The cost of doing this exerts heat because moving magnetic domains within a magnet to align and realign repeatedly causes a lot of magnetic friction. Faster switching of magnets equals more loss in the core. So hysteresis loss increases with speed as seen in $P\_{hyst} \= k\_h \\cdot f \\cdot B\_{pk}^{\\alpha}$.

Eddy current losses in the stator are the reason stators are manufactured with thin silicon steel sheets with a varnish coating. Eddy currents are produced inside the stator that get stronger with frequency. By building stators in sheets, eddy currents are split into smaller components. Since power loss scales by a scale factor with current, decrease in current is greatly appreciated, as seen in $P\_{eddy} \= k\_e \\cdot f^2 \\cdot B\_{pk}^2$.

Core\_loss.png

**Mechanical loss**

Windage is the aerodynamic drag acting on the spinning rotor. This loss is very miniscule and only increases at top speeds. The relationship to speed is quadratic as seen in  $k\_{windage} \\cdot \\omega^2$. 

Bearing friction is modeled as a constant 0.2W. Friction dominates mechanical loss. Within 0.2W is preload, static, and speed dependent friction.

Mech\_loss.png

**Total power analysis and motor efficiency**  
Let’s add up all the losses in power we have found. 

Total\_loss.png

Graph makes a lot of sense, power loss is attributed to torque in many cases. However, some losses don’t depend on speed much.

Now, let's get the power output by multiplying torque and angular speed. 

Output\_power.png

Input power is then given by Pin\=Pout\+Ploss . so just sum the graphs.

Input\_power.png

So we have the input power, we have the output power. Efficiency is simply the measure of the ratio between those two quantities. η=Pout /Pin\=τω/VI. Τω are the output metrics and VI are the input metrics. Together we get this. 

Efficiency\_map\_clean.png

**Conclusion**  
Taking a look at the legends for all these graphs, we can see that copper losses are the majority of heat, followed by iron and mechanical. The other sources of heat we assumed to be zero are also very insignificant compared to copper losses. There's a myriad of techniques to minimize copper losses: 

* Special stacking methods when winding stator  
* Use square wires to improve winding factor and minimize air gaps  
* Litz wiring

Some more ways to decrease other heat and increase efficiency:

* Skewing magnets for smoother rotation  
* Splitting magnets in half and skewing  
* Backplate to concentrate magnetic fields inside motor  
* Better magnets like N52  
* Use FOC control for smoother torque  
* Pwm tuning to reduce current rippling  
* Decreasing silicon concentration in steel for higher saturation limits in stator  
* Increasing silicon concentration for lower core loss