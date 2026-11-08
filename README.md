# 🐍 Inside the dVRK: The Secret Snake Robot Origins of the da Vinci Research Kit

[![DOI](https://zenodo.org/badge/977265608.svg)](https://doi.org/10.5281/zenodo.23002813)

> This repository preserves the original [da Vinci Research Kit (dVRK)](https://www.intuitive-foundation.org/dvrk/) motion controller FPGA code. Active development has migrated to the Johns Hopkins University [`mechatronics-firmware`](https://github.com/jhu-cisst/mechatronics-firmware) repository beginning from commit [`a38917c`](https://github.com/jhu-cisst/mechatronics-firmware/tree/a38917c2ce4880597da052d8d369eaeaf47659b4).

Rather than being designed from scratch, the dVRK controller was ported from a completely separate robotic system: **a high-degree-of-freedom surgical snake robot.**

## 1. The Snake Robot Cabling Crunch

<p align="center"><img width="438" height="328" alt="Under a conventional control architecture, robot joints are wired directly to the workstation bus for processing, resulting in a complex cabling scenario." src="https://github.com/user-attachments/assets/4927e4fb-b4fe-4480-857b-854357ff21d5" /> <img height="328" alt="The original snake robot actuation unit also suffered from cable drag." src="https://github.com/user-attachments/assets/bc741eae-0082-45da-822e-0fd8371acf59" /></p>

### More Motors, More Problems

During the early days of the LCSR, the lab was making strides in dexterity with a [snake robot](https://ciis.lcsr.jhu.edu/doku.php?id=research.snake_robot) designed for remote telesurgery of the upper airways. However, standard control strategies involved running bulky wiring bundles from every control axis to a centralized electronics rack. The dozens of sensors and actuators needed to articulate such a high-dof robot created a physical bottleneck. The system became increasingly error prone and problematic.

To solve this, we designed a [motion control architecture](https://www.researchgate.net/publication/224353229_Centralized_processing_and_distributed_IO_for_robot_control) optimized for high-dof systems:
* **Centralized Processing, Distributed I/O:** Rather than routing bundles of wires from each joint to the workstation individually, sensor and actuator signals are digitized and serialized by a nearby FPGA.
* **The FireWire Bus:** The reformatted motor commands and sensor readings are then multiplexed over a single, high-speed IEEE 1394 (FireWire) serial bus, maintaining real-time performance.
* **The Original Codebase:** The snake robot embodiment of this architecture formed the [`SnakeFPGA-rev2`](https://github.com/deepsurgical/SnakeFPGA-rev2) codebase.

To top it all off (so to speak), we built a next-gen snake robot for the new controller and got [exciting results](https://www.researchgate.net/publication/254025466_Design_of_a_scalable_real-time_robot_controller_and_application_to_a_dexterous_manipulator).
<p align="center"><img height="583" alt="Next-gen surgical snake robot with a FireWire-based motion controller." src="https://github.com/user-attachments/assets/f4c8e592-6f26-42aa-ae15-6304806d144b" /></p>

## 2. Porting the Snake Robot Controller to the dVRK
When Intuitive Surgical later provided retired first-generation da Vinci systems for research, labs faced a similar engineering hurdle: The donated manipulators needed new motion control electronics. Because the communication protocols and packet definitions implemented in `SnakeFPGA-rev2` were neatly decoupled from robot kinematics, porting it to the da Vinci interface boards was relatively seamless. The boards themselves were designed and fabricated in a matter of weeks as a result of a firmware-first design philosophy.

<p align="center"><img height="218" alt="First-generation da Vinci Research Kit controller pictured on the left a year before launch, next to its predecessor the snake robot controller on the right" src="https://github.com/user-attachments/assets/ee1c27f2-e760-4bae-828f-442b6b8efd25" /> <img height="218" alt="The da Vinci Research Kit and snake robot controllers were developed in parallel so changes in one could be tested on the other immediately. This was at an advanced stage where robotic mechanisms and motors were being integrated and it was getting real." src="https://github.com/user-attachments/assets/01a7c7e6-b790-4ac5-a861-352e2342024d" />
</p>

Porting the Altera-based snake robot firmware to the Xilinx-based da Vinci Research Kit occurred in this very repository. Development on the two controllers continued in parallel to ensure consistency and generalizability. The generalized multi-axis data distribution design allowed the first functional dVRK systems to be brought up in no time.

[First known recorded power-on of the da Vinci Research Kit (dVRK) motion controller](https://github.com/user-attachments/assets/3037465c-1144-416a-8ffe-dfd9f047238d)

## 3. Digital Archaeology: Tracing the dVRK Firmware Origins

Version control allows us to retrace the evolution from `SnakeFPGA-rev2` to `daVinci1394FPGA` to its open source release as JHU's `mechatronics-firmware`.

### 🔍 Migration Artifacts: History Hiding in the Headers
Like a primitive cave painting, there's an old school revision log etched in the header comments of the official [FireWire.v source code](https://github.com/jhu-cisst/mechatronics-firmware/blob/a38917c2ce4880597da052d8d369eaeaf47659b4/FPGA1394_QLA/Verilog/FireWire.v#L17-L23), chronicling how it was initially written for the snake robot in 2008 (`SnakeFPGA-rev2`), ported to the Xilinx-based da Vinci robot in 2010 (`daVinci1394FPGA`), and received sporadic updates until eventually moving to the dVRK repository (JHU `mechatronics-firmware`):

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
 * ...
 */
```

### 🔍 Digital DNA Testing

Testing via the `git diff` command shows that `daVinci1394FPGA` at its final commit d8995fc led straight into to the dVRK firmware in JHU `mechatronics-firmware` at initial its commit a38917c, a nearly identical match:

```bash
$ git init
$ git remote add --fetch daVinci1394FPGA https://github.com/deepsurgical/dvrk-davinci1394fpga-original.git
$ git remote add --fetch mechatronics-firmware https://github.com/jhu-cisst/mechatronics-firmware.git
$ git diff d8995fc a38917c:FPGA1394_QLA --summary
 rename {simulation => Simulation}/CtrlDac.do (100%)
 rename {simulation => Simulation}/EncCtrl.do (100%)
 rename {simulation => Simulation}/EncQuad.do (100%)
 rename {simulation => Simulation}/FireWireBlock.do (100%)
 rename {simulation => Simulation}/FireWireQuad.do (100%)
 rename {simulation => Simulation}/Ltc1864x4.do (100%)
 rename {simulation => Simulation}/Ltc2601x4.do (100%)
```

For details on how `SnakeFPGA-rev2` was ported to `daVinci1394FPGA`, check out this [patch](https://gist.github.com/deepsurgical/8045ac1c79d1b66993b8fab58c08054f) and the [corresponding writeup](https://www.linkedin.com/posts/paul-thienphrapa_happyhalloween-surgicalsnakerobot-dvrk-activity-7522644367107616769-EDQi).

### 🔍 Examining The Commit Archives
Mechanically adding a robot axis might require a complete redesign, but adding it to a _scalable controller_ can be as simple as... adding. This [prehistoric commit](https://github.com/jhu-cisst/mechatronics-firmware/commit/b6e0830884eb06d80bc08d6f908d80e43b5e1405) shows another step in 'transforming' the snake robot into the dVRK:

> Paul Thienphrapa committed on Oct 16, 2012<br>
> FPGA1394_QLA: Increase maximum number of axes from 7 to 8

The fact that the snake robot had 7 control axes while the dVRK controller had 8 was hardcoded into the firmware in `SnakeFPGA-rev2` and `daVinci1394FPGA` respectively.

### 🔍 Excavating the 3 Pillars of the dVRK

The _hardware + software + electronics_ recipe had been circulating for some time, culminating in the [official dVRK publication](https://doi.org/10.1109/ICRA.2014.6907809) that documented how this trinity of pillars was toppled one-by-one, leading to the final monument:
1. **Pillar #1: Hardware**
   * _Quote:_ Clinical-grade surgical robots were becoming "increasingly available to researchers via the reuse of retired clinical systems."
   * _Meaning:_ Hardware was no longer a bottleneck because in a rare twist, da Vinci mechanisms were becoming abundant.
2. **Pillar #2: Software**
   * _Quote:_ "Open-source robot software, such as the Robot Operating System (ROS) [[1]](https://www.researchgate.net/publication/233881999_ROS_an_open-source_Robot_Operating_System), has seen widespread adoption."
   * _Meaning:_ Software was no longer a bottleneck because by 2014, multiple open source options had proliferated the landscape.
3. **Pillar #3: Electronics**
   * _Quote:_ There were "relatively few open hardware/software platforms in widespread use within the robotics research community."
   * _Meaning:_ With the native da Vinci controller withheld, electronics became the main bottleneck for the dVRK.
   * _Quote:_ Researchers needed "complete access to all levels of control via open-source electronics and software."
   * _Meaning:_ A dedicated initiative was required to unblock the impasse via efforts such as `daVinci1394FPGA`.
   * _Quote:_ The scope included "an FPGA to enable a centralized computation and distributed I/O architecture."
   * _Meaning:_ What started off as `SnakeFPGA-rev2` unexpectedly became the defining enabler for the dVRK.

The [dVRK article](https://doi.org/10.1109/ICRA.2014.6907809) highlights the motion controller, and the `SnakeFPGA-rev2` -> `daVinci1394FPGA` -> `mechatronics-firmware` progression, as the foundation of the da Vinci Research Kit and downstream innovations in surgical robotics.

### 🔍 Unearthing the _Intuitive Research Kit_
Before the open source rollout and rebrand to the _da Vinci Research Kit (dVRK)_, the platform was provisionally referred to as the _Intuitive Research Kit_. The name can still be found in the [dVRK GitHub repository](https://github.com/jhu-dvrk/sawIntuitiveResearchKit) and in some [transient documents](http://jhir.library.jhu.edu/handle/1774.2/37924) ([pdf](https://rose.mepaul.com/w/images/b/b1/Pault_thesis-136-final.pdf#page=218)):
> _Besides enabling use of the Snake Robot (Section 3.4.2), the outcomes of this effort formed the basis for JHU Open Source Mechatronics [155, 156], which publicly hosts a set of electronics design files, FPGA code, and basic software for a FireWire-based motion controller. This in turn is a component of the **Intuitive Research Kit** [157,158]._

## 4. Priceless Discoveries
This repository highlights the power of **modular mechatronics** for **ageless architecture**. A control architecture abstracted enough to handle multi-axis distributed I/O over a high-speed serial link can easily outlast its physical form, jumping from a bespoke miniature snake robot to a medical robotics research platform found in labs all around the world.

What started off as a modest set of .v files has expanded to support a wide array of hardware variants and technical features. These are just a few of the many [interesting facts](https://www.mepaul.com/wiki/10-background-facts-da-vinci-research-kit-dvrk) about the dVRK!

## 5. Publication Timeline
* **2008**: Design of the motion control architecture: [_Centralized processing and distributed I/O for robot control_](https://doi.org/10.1109/TEPRA.2008.4686678)
* **2008**: Initial prototype for the snake robot: [_A distributed I/O low-level controller for highly-dexterous snake robots_](https://doi.org/10.1109/BIOCAS.2008.4696861)
* **2009**: Architecture generalization and API: [_A scalable system for real-time control of dexterous surgical robots_](https://doi.org/10.1109/TEPRA.2009.5339651)
* **2010**: Interim Master's thesis reporting: [_A centralized processing, distributed I/O motor controller based on IEEE 1394 for the snake robot_](https://rose.mepaul.com/w/images/3/39/Qual2010.pdf)
* **2011**: Fully integrated implementation: [_Design of a scalable real-time robot controller and application to a dexterous manipulator_](https://doi.org/10.1109/ROBIO.2011.6181640)
* **2014**: Running an autonomous snake robot: [_Guidance of a high dexterity robot under 3D ultrasound for minimally invasive retrieval of foreign bodies from a beating heart_](https://doi.org/10.1109/ICRA.2014.6907572)
* **2014**: Evolution into to the dVRK: [_An open-source research kit for the da Vinci® Surgical System_](https://doi.org/10.1109/ICRA.2014.6907809)
* **2025**: The next generation of dVRK: [_dVRK-Si: The next generation da Vinci Research Kit_](https://doi.org/10.1109/ISMR67322.2025.11025986)
