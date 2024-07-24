![image](https://github.com/user-attachments/assets/49f7c441-a078-4e94-ad0e-b2b02b045055)


# Open3DFlow


Amid the escalating need for high-performance, low-power, and densely integrated electronic systems, 3D-ICs emerge as a promising "more than Moore'' integration solution. Nevertheless, the scarcity of specialized EDA tools and standardized design flows tailored for 3D chiplets hinders silicon innovation. To address this gap, we propose a 3D RISC-V processor, mimicking AMD's 3D V-cache architecture. To realize this architecture, we develop 'Open3DFlow', an open-source 3D IC design platform that leverages existing openEDA tools while incorporating tailored abstractions and customizations optimized for 3D chiplet designs. Besides OpenROAD, we also integrate other open tools to enable Through Silicon Via (TSV) modeling, thermal analysis, and signal integrity (SI) assessments. Our CPU consists of two tiers: a cache die and a logic die, stacked face-to-face (F2F) using different processes. The interconnects converge at the central bonding layer, where the bonding pads are located. Our ambition is to establish a fully open-source realization of cutting-edge technologies, not only to facilitate the resolution of future challenges within this platform but also to pave the way for the exploration of novel packaging paradigms.

## 1.   Introduction and Spec
In the realm of microelectronics, the pursuit of superior performance, energy efficiency, optimized space utilization, and economical solutions has spurred the development of novel integration techniques. Given the need for semiconductor memory chips with higher density to support recently released CPU components, the technique for advanced packages has emerged as a promising solution.

The technological trend can be roughly divided into four stages:    
- 2D package, with solder balls or bumps as the representative connection method. (Flip chip, wire bonding)
- Wafer Level Packaging, utilizing Redistribution Layer (RDL) as the representative connection approach. (INFO-PoP, eWLB)
- 2.5D package, typically based on interposer or Embedded Multi-die Interconnect Bridge. (EMIB, CoWoS-x)
- 3D stacking, employing hybrid bonding and Through Silicon Vias (TSVs) for signal and power transmission. (3D SoIC)

As depicted in the following diagram, the trend consistently leans towards achieving greater integration and shorter interconnection distances:

![image](https://github.com/user-attachments/assets/46826f07-f490-40d2-a25a-42a753a4a8d2)

Currently, the 3D IC flow faces several limitations. Primarily, it relies heavily on 2D tools and lacks comprehensive thermal analysis as well as power and signal integrity (SI) assessments. And because there are knowledge gaps between chip design and packaging. Modeling for 3D items such as TSVs and bonding pads hardly exists in IC back-end processes. Additionally, due to the immature and closed nature of existing 3D design EDA tools, future designs face challenges in integrating with large models and cloud platforms. Commercial licensing costs and limitations on processor scalability in cloud environments further complicate the matter.

To address these issues, in the work, we present our own 3D IC design methodology. We develop a 5-stage-pipeline RISC-V CPU with its cache die stacking on the logic die, which mimics AMD's 3D V-cache architecture. We incorporate openPDKs and openEDA tools while doing the hardening. For that end, we propose an open source 3D chip EDA design platform with TSV and thermal modeling named 'Open3DFlow'. It is also the first fully open-source process for designing 3D chips, which is poised to significantly contribute to the vibrant growth of the open-source community. Our goal is to tape out this design through the OpenMPW program, and the process will be easily extended to include logic+DRAM or logic+logic configurations. 

The significance of this work can be summarized:
1.  We build a 3D RISC-V chip in fully open-source mode, leveraging open EDA tools, incorporating hybrid bonding and TSV technologies.
2.  we propose 'Open3DFlow', an open source EDA platform that offers comprehensive TSV/Chip's SI and thermal analysis, ensuring reliable and efficient 3D IC designs
3.  The open-source nature facilitates seamless integration with AI, large models, and cloud computing, boosting design efficiency for complex 3D ICs.

4.      Architecure Specification：
We employ a five-stage pipelined RISC-V CPU equipped with an L1 cache to demonstrate our 3D IC design flow. In the future, we will transition towards more intricate designs to co-optimize our toolchains. The core was created for an OSU undergraduate course in July 2019, we have modified it to fit for our 3D design flow. Here we summarize the key features:
- 32-bit CPU RISC-V supporting I extension 
- 5-stage pipeline in-order execution
- 1KB L1 cache concatenate by 4 sram macros
- PDK for logic die：SkyWater 130nm (Sky130A)
- PDK for cache die: GlobalFoundries 180nm (GF180)
- Connection methods: bonding pads for hybrid bonding & TSV

![image](https://github.com/user-attachments/assets/72cbbdb7-bb9e-40c8-88f5-b368f6930650)


## 2.   Architecture Design of 3D RISC-V Processor with V-Cache
### 2.1 Chiplet Consideration
There are some typical structures for 3D processors. As depicted in the following figure:

![image](https://github.com/user-attachments/assets/dbf4aeec-29a6-4029-b6bb-dfdfa0205aa5)

(a) depicts combinations of multiple packages; (b) represents 2.5D IC, applying passive or active interposers; (c) illustrates for DRAMs on logic dies; (d) stacks one logic circuit on another; and in (d), SRAMs are positioned beneath the logic but for (f), the logic is on the upper side. 

However, modern 3D processor architectures often combine multiple stacking approaches. The prevailing trend in 3D architecture evolution is towards increased density integration, reduced micro-bumps and TSV pitches, as well as shorter chiplet distances. For instance, AMD's 3D-Vcache does not follow the traditional approach of placing caches alongside the processor; instead, it stacks additional cache layers on top of the CPU. This architecture enables AMD to compress more cache without fabricating larger CPUs, resulting in improved speed and power efficiency in gaming applications. To achieve broader bandwidth and faster transmission speeds, the hybrid bonding technology even eliminates bumps. In our design, we select the "CPU + Caches" structure for experimentation. We aim to develop a fully open-source 3D-Vcache structure utilizing openEDA tools and openPDKs. Our aspiration is also to establish a platform that can be flexibly extended to other stacking modes in the future.

### 2.2 V-Cache Structure
The L1 cache is composed by concatenating 4 GF180 256*8-SRAM  macros, utilizing a 7-bit address for both reading and writing operations. As shown in this figure:


![image](https://github.com/user-attachments/assets/9113eff9-1c2c-4f87-a6a8-7c3559287203)

All signals related to this cache die, excluding the clock signal, establish direct communication with the logic die via bonding pads and TSVs. The floorplan for the macros is manually crafted. The two ties possess their own independent power supply networks, but converge on the logic die. Detailed insights into the back-end design are outlined in the subsequent sections.

### 2.3  RISC-V CPU with 3D Stacking SRAM
The overall stacked architecture is illustrated as:

![image](https://github.com/user-attachments/assets/37b821aa-cee4-4ab1-9a09-eafb05a90f58)

In our study, two dies are stacked in a F2F configuration, which means that their corresponding connection signal points converge onto the same bonding pads on the bonding layer. Additionally, the TSVs are via-last fabricated, where they reach the topmost layer of the top die and penetrate through all the metal layers of the sub die. Finally, the entire 3D chip is mounted on the substrate using flip-chip packaging with solder bumps.



</br>

#### Team Members

|Name|Affiliation|
|:--:|:----------:|
| Yifei Zhu (zhuyf20@mails.tsinghua.edu.cn)| PhD Student, RIOS Lab, Tsinghua-Berkeley Shenzhen Institute, Tsinghua University|
| Weiwei Chen (weiwei.c@rioslab.org)| Scientist, RIOS Lab, Tsinghua-Berkeley Shenzhen Institute, Tsinghua University|
| Lei Ren (ren@rioslab.org)| Scientist, RIOS Lab, Tsinghua-Berkeley Shenzhen Institute, Tsinghua University|
| Zhangxi Tan (xtan@rioslab.org)| Doctor, RIOS Lab, Tsinghua-Berkeley Shenzhen Institute, Tsinghua University|
<!-- | Xieyuan Wu (wuxy23@mails.tsinghua.edu.cn)| RIOS Lab, Tsinghua-Berkeley Shenzhen Institute, Tsinghua University|
| Yucheng Wang (wangyc23@mails.tsinghua.edu.cn)| RIOS Lab, Tsinghua-Berkeley Shenzhen Institute, Tsinghua University| -->



## About RIOS Lab

![image](https://github.com/riosmpw/OpenRPDK28/assets/109063674/6aae13c6-50a5-40c3-9a4e-ed4c79d41c20)


**Ecosystem Wants to be Free**

By David A. Patterson · Director of RIOS Lab

**RISC-V International Open Source Laboratory** (RIOS Lab) is a Shenzhen-based research facility focused on computer system architecture, supported by the Tsinghua-Berkeley Shenzhen Research Institute. As an Open Source and Nobel Prize Laboratory, RIOS Lab promotes open-source innovation and collaboration. Our philosophy is that the computer architecture ecosystem should be free for all to access and build upon.

In November 2019, RIOS Lab was officially unveiled. Under the leadership of 2017 A.M. Turing Award winner Prof. David A. Patterson and operational support from TBSI,  RIOS Lab will conduct cutting-edge research in RISC-V hardware and software technology. Patterson first proposed the Reduced Instruction Set Computer (RISC), an open and free instruction set architecture enabling a new era of processor innovation through open standard collaboration. Released in 2010, the latest Fifth Generation RISC has gained worldwide attention.

The name for the lab RIOS is also inspired by the Spanish word for “rivers.” It symbolizes the flow of information from many sources, coming together to create a whole that is greater than the sum of its parts.











