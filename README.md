## Programmers Guide

### Primary Inputs to REV Runtime
__EngineSimulationUpdateParams__
* Throttle: Float [0,1]: Throttle of the engine. 
* Gear: Int [1,5]: What gear is the car in. There's no neutral or reverse
* RPM: Float [0,1]: __Normalized__ engine RPM where 0 is Idle and 1 is Redline
* Velocity: Float [0, 150] : KPH or the car. Only utilized for off the line Clutch Modeling.
* Pitch Float [0.5, 1.9]: Default 1.0f : Repitch the engine
* EnableShifting : Bool Default: True: enable the up/down shifting effects of the car. When driving normally, keep these enable. Scenarios where you'd disable are A) Resetting the car, eg Teleporting to an apprupt stop. B) Crashing into a wall.

### Output Parameteters
Depending on the REV environment you're using, you may have access to 
__Visual RPM__ Float [0,1]:  This value shows a smooth visually accurate representation of what the tachometer should actually look like its doing, without any jitters or pops.

### Gear and RPM Relationship
REV's implementation has settled on the simplest expectations of any game engine it integrates with. Of particular importance is the GEAR and RPM values during up shifts and down shifts.

REV expects that the physics of the game engine will provide it with the new RPM value instantly when the GEAR changes, eg: form 2nd to 3rd, the RPM would likely go from 0.98 to 0.63 instantly with no lagging.

REV Itself will perform a "Shift Pattern" that is far more realistic.

![](https://github.com/CrankcaseAudio/REVWiki/blob/master/Docoumentation/Up%20Shift%20Patterns.jpg)
![](https://github.com/CrankcaseAudio/REVWiki/blob/master/Docoumentation/Down%20Shift%20Pattern.jpg)

Below is an example stream of values your game engine might send to REV
```
Thottle,RPM,Velocity,Gear
1,0.9075058,22.68764,2
1,0.9141787,22.85447,2
1,0.9208513,23.02128,2
1,0.9275238,23.1881,2
1,0.934196,23.3549,2
1,0.940868,23.5217,2
1,0.9475397,23.68849,2
1,0.9542113,23.85528,2
1,0.9608825,24.02206,2
1,0.9675536,24.18884,2
1,0.6819571,24.35561,3 <<<<< Notice the abrupt drop to the new 3rd Gear RPM
1,0.6848525,24.45902,3
1,0.6878135,24.56477,3
1,0.6907086,24.66817,3
1,0.6936036,24.77156,3
```
	

### Programmer tools
Available upon request support@crankcaseaudio.com

#### __AI Physics Simulator__
A lot of times, the game has an AI car that is driving onscreen, but it drives crazy. You average out the values and feed them to REV with this component.
- Initialize with gear ratios, 
- Update with velocity, 
- Generate a Gear & RPM value to feed to REV.
- You should be able to infer Throttle from the change in the velocity being +/-. Smooth out or use a sticky on/off'latch' if even thats too noisy a signal.
- Example: Average the input throttle over 10 frames. If > 0.5f, send output throttle of 1.0f into REV. Lock that throttle at 1.0f for a minimum of 1 second. Then revaluate.
- This way the AI cars won't sound like they're tapping the throttle like maniacs.

#### __Physics Simulator__
This is what powers the REV.Tool driving simulator as well as the Wwise driving simulator.
- Initialize with Gear ratios/Car weight/torque, 
- Update with a Throttle/Brake value 
- generates RPM/Throttle/Gear/Velocity to feed to rev.

Example Values:
	Gear Ratios [3.50f, 2.0f, 1.40f, 1.0f, 0.70f]
	Weight = 900.0f;
    EngineTorque = 2500.0f;
	BreakingHorsePower = 6000.0f;

Additionally, there's engine_model_streamdata.csv which is a stream representing an ideal acceleration run of a car. 
This is useful for reference purposes.
