# 🐍 Inside the dVRK: The Secret Snake Robot Origins of the da Vinci Research Kit

> This repository preserves the original [da Vinci Research Kit (dVRK)](https://www.intuitive-foundation.org/dvrk/) motion controller FPGA code. Active development has migrated to the Johns Hopkins University [`mechatronics-firmware`](https://github.com/jhu-cisst/mechatronics-firmware) repository beginning from commit [`a38917c`](https://github.com/jhu-cisst/mechatronics-firmware/tree/a38917c2ce4880597da052d8d369eaeaf47659b4).

Rather than being designed from scratch, the dVRK controller was ported from a completely separate robotic system: **a high-degree-of-freedom surgical snake robot.**

### 1. The Snake Robot Cabling Crunch

<p align="center"><img width="438" height="328" alt="Under a conventional control architecture, robot joints are wired directly to the workstation bus for processing, resulting in a complex cabling scenario." src="https://github.com/user-attachments/assets/4927e4fb-b4fe-4480-857b-854357ff21d5" /> <img height="328" alt="The original snake robot actuation unit also suffered from cable drag." src="https://github.com/user-attachments/assets/bc741eae-0082-45da-822e-0fd8371acf59" /></p>

#### More Motors, More Problems

During my early days in the LCSR, the lab was making strides in dexterity with a snake robot designed for remote telesurgery of the upper airways. However, standard control strategies involved running bulky wiring bundles from every control axis to a centralized electronics rack. The dozens of sensors and actuators needed to articulate such a high-dof robot created a physical bottleneck. The system became increasingly error prone and problematic.

To solve this, we designed a [motion control architecture](https://www.researchgate.net/publication/224353229_Centralized_processing_and_distributed_IO_for_robot_control) optimized for high-dof systems:
* **Centralized Processing, Distributed I/O:** Rather than routing bundles of wires from each joint to the workstation individually, sensor and actuator signals are digitized and serialized by a nearby FPGA.
* **The FireWire Bus:** The reformatted motor commands and sensor readings are then multiplexed over a single, high-speed IEEE 1394 (FireWire) serial bus, maintaining real-time performance.
* **The Codebase:** The snake robot embodiment of this architecture formed the [`SnakeFPGA-rev2`](https://github.com/deepsurgical/SnakeFPGA-rev2) codebase.

To top it all off (so to speak), we built a next-gen snake robot for the new controller and got [exciting results](https://www.researchgate.net/publication/254025466_Design_of_a_scalable_real-time_robot_controller_and_application_to_a_dexterous_manipulator).
<p align="center"><img height="583" alt="Next-gen surgical snake robot with a FireWire-based motion controller." src="https://github.com/user-attachments/assets/f4c8e592-6f26-42aa-ae15-6304806d144b" /></p>

### 2. Porting the Snake Robot Legacy to the dVRK
When Intuitive Surgical later provided retired first-generation da Vinci systems for research, labs faced a similar engineering hurdle: The donated manipulators needed new motion control electronics. Because the communication protocols and packet definitions implemented in `SnakeFPGA-rev2` were neatly decoupled from robot kinematics, porting it to the da Vinci interface boards was relatively seamless. The boards themselves were designed and fabricated in a matter of weeks as a result of a firmware-first design philosophy.

<p align="center"><img height="218" alt="First-generation da Vinci Research Kit controller pictured on the left a year before launch, next to its predecessor the snake robot controller on the right" src="https://github.com/user-attachments/assets/ee1c27f2-e760-4bae-828f-442b6b8efd25" /> <img height="218" alt="The da Vinci Research Kit and snake robot controllers were developed in parallel so changes in one could be tested on the other immediately. This was at an advanced stage where robotic mechanisms and motors were being integrated and it was getting real." src="https://github.com/user-attachments/assets/01a7c7e6-b790-4ac5-a861-352e2342024d" />
</p>

Porting the Altera-based snake robot firmware to the Xilinx-based da Vinci Research Kit took place in this very repository. Development on the two controllers continued in parallel to ensure consistency and generalizability. The generalized multi-axis data distribution design allowed the first functional dVRK systems to be brought up in no time.

[First known recorded power-on of the da Vinci Research Kit (dVRK) motion controller](https://github.com/user-attachments/assets/3037465c-1144-416a-8ffe-dfd9f047238d)

### 3. Digital Archaeology: Tracing the Modern dVRK Firmware

We can retrace the steps leading from the predecessor `SnakeFPGA-rev2` to this `daVinci1394FPGA` to its eventual open source release as JHU `mechatronics-firmware` on GitHub.

#### 🔍 History Hiding in the Headers: The FireWire.v Migration Logs
Like a primitive cave painting, there's an old school revision log etched in the header comments of the official [FireWire.v source code](https://github.com/jhu-cisst/mechatronics-firmware/blob/main/FPGA1394_QLA/Verilog/FireWire.v), showing how it was initially written for the snake robot in 2008, ported to the da Vinci about 2.5 years later in 2010, and received sporadic updates until moving to GitHub in Sep 2012:

```verilog
/*
 * ...
 * Revision history
 *     04/24/08    Paul Thienphrapa    Initial revision
 *     10/13/10    Paul Thienphrapa    Copied from SnakeFPGA-rev2 and tweaked
 *                                       for Xilinx
 *     10/31/11    Paul Thienphrapa    React to rx packets only when addressed
 *     11/11/11    Paul Thienphrapa    Happy 111111!!11!
 *                                     Fixed mixed blocking/non-blocking issues
 *     10/16/13    Zihan Chen          Modified to support hub capability
 * ...
 */
```

#### 🔍 Scaling the System: The Firmware Parameter Shift
Mechanically adding a robot axis might require a complete redesign, but adding it to the controller can be as simple as... adding. This [prehistoric commit](https://github.com/jhu-cisst/mechatronics-firmware/commit/b6e0830884eb06d80bc08d6f908d80e43b5e1405) shows another step in 'transforming' the snake robot into the dVRK:

> Paul Thienphrapa, Oct 16, 2012, FPGA1394_QLA: Increase maximum number of axes from 7 to 8

The fact that the snake robot had 7 control axes while the dVRK controller had 8 was hardcoded into the firmware in `SnakeFPGA-rev2` and `daVinci1394FPGA` respectively.

#### 🔍 Unlocking the Machine

The _hardware + software + electronics_ recipe had been circulating for some time, culminating in the [official dVRK publication](https://doi.org/10.1109/ICRA.2014.6907809) that documented how this trinity of pillars was toppled one-by-one, leading to the final monument:
1. **Software became commoditized.** The robot software landscape of 2014 featured multiple options. According to the paper, "open-source robot software, such as the Robot Operating System (ROS) [1], has seen widespread adoption."
2. **Hardware became abundant.** The 2014 paper observed that, in a rare twist, clinical-grade surgical robots were becoming "increasingly available to researchers via the reuse of retired clinical systems."
3. **Electronics became the critical enabler.** The paper notes that there were "relatively few open hardware/software platforms in widespread use within the robotics research community." Furthermore, the native da Vinci motion controller—the presumptive bridge between the above hardware and software—was withheld to protect sensitive IP. Thus a dedicated initiative was needed to unblock the impasse and provide "complete access to all levels of control via open-source electronics and software." This was achieved through carefully engineered firmware and "an FPGA to enable a centralized computation and distributed I/O architecture" as detailed in this repository.

With the building blocks combined and the framework in place, the paper highlighted several emergent advantages:
* **Firmware modularizes software.** By handling FireWire transactions entirely on the FPGA, the architecture's API virtualized the hardware, turning software into interchangeable blocks. Users who "prefer to use a different real-time framework, such as Orocos [15]" could bypass the default software entirely.
* **Frameworks accelerate development.** The firmware-level hardware abstraction in turn allowed high-level control algorithms to be implemented "in a familiar development environment (Linux PC)," thereby reducing the learning curve. The framework further enabled "reduced cabling... and high flexibility in control algorithms while maintaining precise real-time hardware control. This is particularly useful for developing haptic interactions and virtual fixtures," as well as advanced capabilities such as [autonomous surgery](https://doi.org/10.1126/scirobotics.adt5254) and safety features.

The [dVRK article reveals](https://doi.org/10.1109/ICRA.2014.6907809) the motion controller's latent emergence as a key enabler of the da Vinci Research Kit. The electronics, firmware, and architecture that went from `SnakeFPGA-rev2` to `daVinci1394FPGA` to JHU `mechatronics-firmware` helped pave the way for downstream innovations in surgical robotics.

#### 🔍 Uncovering the Intuitive Research Kit
Before the open-source rollout and rebrand to the _da Vinci Research Kit (dVRK)_, the platform was provisionally referred to as the _Intuitive Research Kit_. The name can still be found in the [dVRK GitHub repository](https://github.com/jhu-dvrk/sawIntuitiveResearchKit) and in some [transient documents](http://jhir.library.jhu.edu/handle/1774.2/37924) ([pdf](https://rose.mepaul.com/w/images/b/b1/Pault_thesis-136-final.pdf#page=218)):
> _Besides enabling use of the Snake Robot (Section 3.4.2), the outcomes of this effort formed the basis for JHU Open Source Mechatronics [155, 156], which publicly hosts a set of electronics design files, FPGA code, and basic software for a FireWire-based motion controller. This in turn is a component of the **Intuitive Research Kit** [157,158]._

#### 🔍 Adaptation and Resilience
What started off as a modest set of .v files has expanded to support a wide array of hardware variants and technical features. These are just a few of the many [interesting facts](https://www.mepaul.com/wiki/10-background-facts-da-vinci-research-kit-dvrk) about the dVRK!

### 4. Modular Mechatronics and Ageless Architecture
This repository highlights the power of modular digital system design. A control architecture abstracted enough to handle multi-axis distributed I/O over a high-speed serial link can easily outlast its physical form, jumping from a bespoke miniature snake robot to a medical robotics research platform found in labs all around the world.

### 5. References
1. Design of the motion control architecture: [_Centralized processing and distributed I/O for robot control_](https://doi.org/10.1109/TEPRA.2008.4686678)
2. Initial prototype for the snake robot: [_A distributed I/O low-level controller for highly-dexterous snake robots_](https://doi.org/10.1109/BIOCAS.2008.4696861)
3. Architecture generalization and API: [_A scalable system for real-time control of dexterous surgical robots_](https://doi.org/10.1109/TEPRA.2009.5339651)
4. Fully integrated implementation: [_Design of a scalable real-time robot controller and application to a dexterous manipulator_](https://doi.org/10.1109/ROBIO.2011.6181640)
5. Running an autonomous snake robot: [_Guidance of a high dexterity robot under 3D ultrasound for minimally invasive retrieval of foreign bodies from a beating heart_](https://doi.org/10.1109/ICRA.2014.6907572)
6. Evolution into to the dVRK: [_An open-source research kit for the da Vinci® Surgical System_](https://doi.org/10.1109/ICRA.2014.6907809)
7. The next generation of dVRK: [_dVRK-Si: The next generation da Vinci Research Kit_](https://doi.org/10.1109/ISMR67322.2025.11025986)
