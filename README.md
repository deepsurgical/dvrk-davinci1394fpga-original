# 🐍 Inside the dVRK: The Secret Snake Robot Origins of the da Vinci Research Kit

[![DOI](https://zenodo.org/badge/977265608.svg)](https://doi.org/10.5281/zenodo.23002813)

> This repository preserves the original [da Vinci Research Kit (dVRK)](https://www.intuitive-foundation.org/dvrk/) motion controller FPGA code. Active development has migrated to the Johns Hopkins University [`mechatronics-firmware`](https://github.com/jhu-cisst/mechatronics-firmware) repository beginning from commit [`a38917c`](https://github.com/jhu-cisst/mechatronics-firmware/tree/a38917c2ce4880597da052d8d369eaeaf47659b4).

Rather than being designed from scratch, the dVRK controller was ported from a completely separate robotic system: **a high-degree-of-freedom surgical snake robot.**

## 1. The Snake Robot Cabling Crunch

<p align="center"><img width="438" height="328" alt="Under a conventional control architecture, robot joints are wired directly to the workstation bus for processing, resulting in a complex cabling scenario." src="https://www.mepaul.com/dvrk-origins/user-attachments/assets/4927e4fb-b4fe-4480-857b-854357ff21d5.png" /> <img height="328" alt="The original snake robot actuation unit also suffered from cable drag." src="https://www.mepaul.com/dvrk-origins/user-attachments/assets/bc741eae-0082-45da-822e-0fd8371acf59.png" /></p>

### More Motors, More Problems

During the early days of the LCSR, the lab was making strides in dexterity with a [snake robot](https://ciis.lcsr.jhu.edu/doku.php?id=research.snake_robot) designed for remote telesurgery of the upper airways. However, standard control strategies involved running bulky wiring bundles from every control axis to a centralized electronics rack. The dozens of sensors and actuators needed to articulate such a high-dof robot created a physical bottleneck. The system became increasingly error prone and problematic.

To solve this, we designed a [motion control architecture](https://www.researchgate.net/publication/224353229_Centralized_processing_and_distributed_IO_for_robot_control) optimized for high-dof systems:
* **Centralized Processing, Distributed I/O:** Rather than routing bundles of wires from each joint to the workstation individually, sensor and actuator signals are digitized and serialized by a nearby FPGA.
* **The FireWire Bus:** The reformatted motor commands and sensor readings are then multiplexed over a single, high-speed IEEE 1394 (FireWire) serial bus, maintaining real-time performance.
* **The Original Codebase:** The snake robot embodiment of this architecture formed the [`SnakeFPGA-rev2`](https://github.com/deepsurgical/SnakeFPGA-rev2) codebase.

To top it all off (so to speak), we built a brand new snake robot to pair with the brand new controller, leading to [exciting results](https://www.researchgate.net/publication/254025466_Design_of_a_scalable_real-time_robot_controller_and_application_to_a_dexterous_manipulator).
<p align="center"><img height="583" alt="Next-gen surgical snake robot with a FireWire-based motion controller." src="https://www.mepaul.com/dvrk-origins/user-attachments/assets/f4c8e592-6f26-42aa-ae15-6304806d144b.png" /></p>

## 2. Porting the Snake Robot Controller to the dVRK
When Intuitive Surgical provided retired first-generation da Vinci systems for research, labs faced a similar engineering hurdle: The donated manipulators needed new motion control electronics. Because the communication protocols and packet definitions implemented in `SnakeFPGA-rev2` were neatly decoupled from robot kinematics, porting it to the da Vinci interface boards was relatively seamless, if a bit arduous—it _is_ firmware after all. The boards themselves were designed and fabricated in a matter of weeks as a result of a firmware-first design philosophy.

<p align="center"><img height="218" alt="First-generation da Vinci Research Kit controller pictured on the left in late 2011, next to its predecessor the snake robot controller on the right from 2008" src="https://www.mepaul.com/dvrk-origins/user-attachments/assets/ee1c27f2-e760-4bae-828f-442b6b8efd25.jpg" /> <img height="218" alt="The da Vinci Research Kit and snake robot controllers were developed in parallel so changes in one could be tested on the other immediately. This was at an advanced stage where robotic mechanisms and motors were being integrated and it was getting real." src="https://www.mepaul.com/dvrk-origins/user-attachments/assets/01a7c7e6-b790-4ac5-a861-352e2342024d.jpg" />
</p>

### The Big Port
The migration from `SnakeFPGA-rev2` to `daVinci1394FPGA` and the inventive steps taken to generalize a common architecture across both (aka The Big Port) took place between late 2010 and 2011; further details are in this [patch](https://gist.github.com/deepsurgical/8045ac1c79d1b66993b8fab58c08054f) and the [corresponding writeup](https://www.linkedin.com/posts/paul-thienphrapa_happyhalloween-surgicalsnakerobot-dvrk-activity-7522644367107616769-EDQi). The port went from the Altera-based snake robot controller to the Xilinx-based da Vinci Research Kit controller. Rather than a one-shot port, the two controllers benefited from continued parallel development to ensure cross-platform consistency, generalizability, and refinement.

The generalized multi-axis data distribution design allowed the first functional systems to be brought up with little fanfare. The snake robot version became the [first robot-integrated demo](https://www.researchgate.net/publication/254025466_Design_of_a_scalable_real-time_robot_controller_and_application_to_a_dexterous_manipulator) of the controller at ROBIO in December 2011. Bugfixes and refinements carried into 2012 before the firmware went live as JHU `mechatronics-firmware` in September 2012, supporting the official dVRK announcement.

[First known recorded power-on of the da Vinci Research Kit (dVRK) motion controller, June 2013](https://github.com/user-attachments/assets/3037465c-1144-416a-8ffe-dfd9f047238d)

## 3. Digital Archaeology: Tracing the dVRK Firmware Origins

Version control allows us to retrace the evolution from `SnakeFPGA-rev2` to `daVinci1394FPGA` to its open source release as JHU's `mechatronics-firmware`.

### 🔍 Migration Artifacts: History Hiding in the Headers

Like a primitive cave painting, there's an old-school revision log etched in the header comments of the official [FireWire.v source code](https://github.com/jhu-cisst/mechatronics-firmware/blob/a38917c2ce4880597da052d8d369eaeaf47659b4/FPGA1394_QLA/Verilog/FireWire.v#L17-L23), chronicling how it was initially written for the snake robot in 2008 (`SnakeFPGA-rev2`), ported to the Xilinx-based da Vinci robot between 2010 and 2011 (`daVinci1394FPGA`), receiving sporadic updates until eventually moving to the dVRK repository in 2012 (JHU `mechatronics-firmware`):
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

[BoardRegs.v](https://github.com/jhu-cisst/mechatronics-firmware/blob/a38917c2ce4880597da052d8d369eaeaf47659b4/FPGA1394_QLA/Verilog/BoardRegs.v#L8-L11) also started off under `SnakeFPGA-rev2` in 2008. It was later adapted for the `FPGA1394_QLA` boards under `daVinci1394FPGA` in 2011, then touched up before migrating to JHU `mechatronics-firmware` in 2012.
```verilog
/*
 * Revision history
 *     07/17/08    Paul Thienphrapa    Initial revision - SnakeFPGA-rev2
 *     12/21/11    Paul Thienphrapa    Adapted for FPGA1394_QLA
 *     02/22/12    Paul Thienphrapa    Minor fixes for power enable and reset
 */
```

The dVRK controller featured some notable hardware changes versus the snake, including the digital-to-analog converters (LTC2601), analog-to-digital converters (LTC1864), and temperature sensor (MAX6576). While the physical layout prompted some high-level restructuring of the firmware, its modular design made the process efficient. The Verilog modules written for them trace a tantalizing tale of the timeline. Header comments show that the controller boards were manufactured in July 2010, and the accompanying manufacturer test scripts (denoted by `MfgTest`) became bona fide modules in late 2011.

**Fun fact:** The first line of each module's revision history was logged with a blank entry in deference to the vendor, who wished to remain unnamed. As the first such module developed, [Ltc2601x4.v](https://github.com/jhu-cisst/mechatronics-firmware/blob/a38917c2ce4880597da052d8d369eaeaf47659b4/FPGA1394_QLA/Verilog/Ltc2601x4.v#L21-L23) also received the "Revisionist history" badge of honor, a title it held for a decade before being, ironically, revised.
```verilog
/*
 * Revisionist history
 *     07/15/10                        Initial revision - MfgTest
 *     10/26/11    Paul Thienphrapa    Initial revision
 */
```

Likewise for [Ltc1864x4.v](https://github.com/jhu-cisst/mechatronics-firmware/blob/a38917c2ce4880597da052d8d369eaeaf47659b4/FPGA1394_QLA/Verilog/Ltc1864x4.v#L32-L34) - the "x4" suffix indicates that the module interfaced with a quadruplet of DACs or ADCs, hence the "Quad" in Quad Linear Amplifier (QLA).
```verilog
/*
 * Revision history
 *     07/15/10                        Initial revision - MfgTest
 *     11/14/11    Paul Thienphrapa    Initial revision
 */
```

Next up, [Max6576.v](https://github.com/jhu-cisst/mechatronics-firmware/blob/a38917c2ce4880597da052d8d369eaeaf47659b4/FPGA1394_QLA/Verilog/Max6576.v#L31-L33), the temperature sensor:
```verilog
/*
 * Revision history
 *     07/15/10                        Initial revision - MfgTest
 *     11/15/11    Paul Thienphrapa    Initial revision
 */
```

Last but not least, [FPGA1394-QLA.v](https://github.com/jhu-cisst/mechatronics-firmware/blob/a38917c2ce4880597da052d8d369eaeaf47659b4/FPGA1394_QLA/Verilog/FPGA1394-QLA.v#L7-L10). As the top-level module, this specimen exhibits another peculiar glyph: an _email address_, with a quaint fortification against scrapers. Leaving a contact there was a developer's way of giving future maintainers a place to ask questions and vent.
```verilog
/*
 * Revision history
 *     07/15/10                        Initial revision - MfgTest
 *     10/27/11    Paul Thienphrapa    Initial revision (pault at cs.jhu.edu)
 *     02/29/12    Zihan Chen
 */
```

Early 2012 also heralds the arrival of **Zihan**! Spending years in this corner of engineering could get pretty lonely, and not just in the lab. Globally, the intersection of custom motion controllers, custom robots, and FPGA link-level FireWire was vanishingly small. Having him aboard was a game changer!

### 🔍 Digital DNA Testing

Checking with `git diff` confirms that after the `SnakeFPGA-rev2` to `daVinci1394FPGA` migration (aka The Big Port), `daVinci1394FPGA` at its final commit (`d8995fc`) went straight into the JHU `mechatronics-firmware` repo's initial commit (`a38917c`), showing a nearly identical match:

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

### 🔍 Examining The Commit Archives
Mechanically adding a robot axis might require a complete redesign, but adding it to a _scalable controller_ can be as simple as... adding. This [prehistoric commit](https://github.com/jhu-cisst/mechatronics-firmware/commit/b6e0830884eb06d80bc08d6f908d80e43b5e1405) shows another step in 'transforming' the snake robot into the dVRK:

> Paul Thienphrapa committed on Oct 16, 2012<br>
> FPGA1394_QLA: Increase maximum number of axes from 7 to 8

The fact that the snake robot had 7 control axes while the dVRK controller had 8 (2 QLA boards daisy-chained) was parameterized in the firmware of `SnakeFPGA-rev2` and `daVinci1394FPGA` respectively.

### 🔍 Unearthing the _Intuitive Research Kit_
Before the open source rollout and rebrand to the _da Vinci Research Kit (dVRK)_, the platform was provisionally referred to as the _Intuitive Research Kit_. The name can still be found in the [dVRK GitHub repository](https://github.com/jhu-dvrk/sawIntuitiveResearchKit) and in some [transient documents](http://jhir.library.jhu.edu/handle/1774.2/37924) ([pdf](https://rose.mepaul.com/w/images/b/b1/Pault_thesis-136-final.pdf#page=218)):
> _Besides enabling use of the Snake Robot (Section 3.4.2), the outcomes of this effort formed the basis for JHU Open Source Mechatronics [155, 156], which publicly hosts a set of electronics design files, FPGA code, and basic software for a FireWire-based motion controller. This in turn is a component of the **Intuitive Research Kit** [157,158]._

## 4. Analyzing the Literature

The open source da Vinci Research Kit consists of a set of **donated robot hardware**, **robot software**, and a **motion controller**; the [official dVRK publication](https://doi.org/10.1109/ICRA.2014.6907809) characterizes this collection as follows:

* **On hardware**
  * Clinical-grade surgical robots were becoming "increasingly available to researchers via the reuse of retired clinical systems."
  * da Vinci surgical systems became available for research [as early as 2004](https://dvrk.readthedocs.io/main/pages/introduction/groups.html), though broader access was initially limited.
* **On software**
  * "Open-source robot software, such as the Robot Operating System (ROS) [[1]](https://www.researchgate.net/publication/233881999_ROS_an_open-source_Robot_Operating_System), has seen widespread adoption."
  * Additional alternatives included [OROCOS](https://doi.org/10.1109/ROBOT.2001.933002) (2001) and the standalone [cisst](https://doi.org/10.1109/ROBOT.2006.1642285)/[SAW](https://doi.org/10.54294/faegg8) libraries (2006–2008).
* **On controllers**
  * "There are relatively few open hardware/software platforms in widespread use within the robotics research community."
  * [NSF MRI Grant 0722943](https://www.nsf.gov/awardsearch/show-award/?AWD_ID=0722943) (2007–2012) funded the development of open source motion controllers, recognizing that the lack of open, modular alternatives was a fundamental bottleneck in robotics research.
* **Emergent realizations**
  * Robot hardware and software were available while the native controller was not, making the controller the dVRK's logical bottleneck; consequently, the paper supports "complete access to all levels of control via open-source electronics and software" with "an FPGA to enable a _centralized computation and distributed I/O architecture_."
  * Indeed, the paper cites our earlier work, [_Centralized Processing and Distributed I/O for Robot Control_](https://doi.org/10.1109/TEPRA.2008.4686678). That success helped me "commission" The Big Port of 2010–2011—my small role in [the making of the dVRK](https://www.mepaul.com/wiki/Creating-da-vinci-research-kit-prequel).

The [original dVRK article](https://doi.org/10.1109/ICRA.2014.6907809) focuses on the technical details of the combined package. By analyzing its components individually, we can better understand the rapid rise of the da Vinci Research Kit in 2012: (1) timely availability of retired da Vinci hardware, (2) preexisting availability of open source robot software, and finally (3) opportunistic development of a novel motion controller via The Big Port.

## 5. Priceless Discoveries
This historical repository celebrates the power of **modular mechatronics** for **ageless architecture**. A control architecture abstracted enough to handle multi-axis distributed I/O over a high-speed serial link can easily outlast its physical form, jumping from a bespoke miniature snake robot to a medical robotics research platform found in labs all around the world.

What started off as a modest set of .v files has expanded to support a wide array of hardware variants and technical features. These are just a few of the many [interesting facts](https://www.mepaul.com/wiki/10-background-facts-da-vinci-research-kit-dvrk) about the dVRK!

## 6. Publication Timeline
* **2008**: Design of the motion control architecture: [_Centralized processing and distributed I/O for robot control_](https://doi.org/10.1109/TEPRA.2008.4686678)
* **2008**: Initial prototype for the snake robot: [_A distributed I/O low-level controller for highly-dexterous snake robots_](https://doi.org/10.1109/BIOCAS.2008.4696861)
* **2009**: Architecture generalization and API: [_A scalable system for real-time control of dexterous surgical robots_](https://doi.org/10.1109/TEPRA.2009.5339651)
* **2010**: PhD qualifying project report: [_A centralized processing, distributed I/O motor controller based on IEEE 1394 for the snake robot_](https://rose.mepaul.com/w/images/3/39/Qual2010.pdf)
* **2010**: `FPGA1394_QLA` boards fabricated during author's summer internship 
* **2010**: The Big Port begins: [_Code groundbreaking log, mechatronics-firmware/FPGA1394_QLA/Verilog/FireWire.v_](https://github.com/jhu-cisst/mechatronics-firmware/blob/a38917c2ce4880597da052d8d369eaeaf47659b4/FPGA1394_QLA/Verilog/FireWire.v#L19-L20)
* **2011**: The Big Port ends: [_Register file definition, mechatronics-firmware/FPGA1394_QLA/Verilog/BoardRegs.v_](https://github.com/jhu-cisst/mechatronics-firmware/blob/a38917c2ce4880597da052d8d369eaeaf47659b4/FPGA1394_QLA/Verilog/BoardRegs.v#L10)
* **2011**: First robot-integrated snake demo: [_Design of a scalable real-time robot controller and application to a dexterous manipulator_](https://doi.org/10.1109/ROBIO.2011.6181640)
* **2012**: Snake/dVRK controller bugfixes and refinements continue: [_Humans vs. synthesizer, mechatronics-firmware/FPGA1394_QLA/Verilog/EncQuad.v_](https://github.com/jhu-cisst/mechatronics-firmware/blob/a38917c2ce4880597da052d8d369eaeaf47659b4/FPGA1394_QLA/Verilog/EncQuad.v#L12-L13)
* **2012**: dVRK announcement and launch: [_Welcome to the da Vinci Research Kit wiki community_](https://research.intusurg.com/index.php/Main_Page#Background)
* **2013**: Initial autonomous snake robot: [_3D ultrasound-guided retrieval of foreign bodies from a beating heart using a dexterous surgical robot_](https://www.researchgate.net/publication/328529988_3D_ultrasound-guided_retrieval_of_foreign_bodies_from_a_beating_heart_using_a_dexterous_surgical_robot)
* **2013**: First known recorded dVRK controller power-on ([June 2013](https://rose.mepaul.com/w/images/6/6b/Dvrk-first.mp4)); first dVRK demo video ([September 2013](https://www.youtube.com/watch?v=0n_9CgNSx6Y))
* **2014**: Improved autonomous control: [_Guidance of a high dexterity robot under 3D ultrasound for minimally invasive retrieval of foreign bodies from a beating heart_](https://doi.org/10.1109/ICRA.2014.6907572)
* **2014**: Formal debut of the dVRK: [_An open-source research kit for the da Vinci® Surgical System_](https://doi.org/10.1109/ICRA.2014.6907809)
* **2025**: Next generation dVRK: [_dVRK-Si: The next generation da Vinci Research Kit_](https://doi.org/10.1109/ISMR67322.2025.11025986)

*Fun coincidence: Both 2014 papers—the snake robot and the dVRK platform it helped spawn—were presented on the [exact same day at ICRA 2014 in Hong Kong](https://web.archive.org/web/20170313165036/http://ras.papercept.net/conferences/conferences/icra14/program/icra14_contentlistweb_5.html) (in sessions co-chaired by the same person)!*
