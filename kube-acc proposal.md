Kube-acc Proposal
===============

* [Motivation](#motivation)
* [Goals](#goals)
* [Non-goals](#non-goals)
* [Current Design](#current-design)
* [Proposal](#proposal)
* [Challenges](#challenges)
* [Reference](#reference)

# Motivation
With technologies such as AI, 5G, HPC,  Edge Computing, Quantum Computing, and Neuromorphic Computing have becoming ever more prevalent, the need for Kubernetes to better support dedicated devices or accelerators such as FPGA, GPU, SoCs, ASICs (domain specific architectures like RISC-V based chips) is growing stronger and stronger.

Kubernetes community has been working on this area and various solutions are out with recent releases to provide solutions for accelerator support. We have device plugin interface now could work with NVIDIA GPUs and some other types of accelerators for basic functionalities. 

However for large enterprise which needs to build their own cloud infrastructure and also deploy various types of accelerator for a variety of services, the current DPI support might not be fully satisfactory for the task. The user will need a general management framework for all kinds of accelerators which could work with scheduler to support features like topology awareness, unified resource management, policy enforcement, and so forth.

To put it simply, it would be great to have 

- One CRD framework for more than one types of accelerator altogether, instead of one CRD framework per type of accelerator
- All Accelerator life cycle management related APIs exposed via CRD, instead of hacking out using method like sidecars.
- Possibly new CRI support for accelerator runnable images
- General resource representation/data modeling across a variety of accelerators, instead of relying on annotations/labels which is not standardized and messy.
- Reuse existing accelerator IaaS drivers if possible


# Use Cases:
- As a cluster admin, I want to be able to have a unified management of FPGA, GPU and smart nics which are coexisting in my cloud. 
- As a cluster admin, I need Kubernetes cluster to perform topology-aware scheduling for my accelerators to better support the services
- As a cluster admin, I want to be able to configure different policies for different accelerators.
- As a cluster admin, I want to be able to schedule the workload amid a variety of accelerators onto the node that has the most desired accelerator.

# Goals:
- Create a CRD based framework (kube-acc) for out-of-band general management for accelerators in addition to DPI mechanism.
- Coordinate with any scheduler implementation that supports topology-aware and other accelerator friendly scheduling mechanism
- Capable of being integrated with various IaaS implementations (bare metal, OpenStack based, propetiery clouds, etc)
- Perform life cycle management for various types of accelerators with the help of DPI plugins or IaaS layer functionalities. Resource management operations include but not limited to the following:
  - Attach
  - Detach
  - Discovery
  - Programming (FPGA, NP, ...)
  - Power
  - ...
 
- Standardized resource representation/data modeling for accelerators including metadata, topology and other information
- Policy enforcement for different accelerators

# Non-goals:
- Drop in replacement or reinvent of DPI mechanism
- Have yet another scheduler dedicated for accelerators

# Current Design:
![dpi](https://user-images.githubusercontent.com/1736354/63739440-c1f6da80-c8bf-11e9-9e8a-b41202388b48.jpg)
![intel](https://user-images.githubusercontent.com/1736354/63739423-b6a3af00-c8bf-11e9-9347-14fa76b7b5e5.jpg)

# Proposal:
Below is a very high level architecture figure:

![arch_recompress](https://user-images.githubusercontent.com/1736354/63739219-12ba0380-c8bf-11e9-9b65-366efb031692.jpg)

Notes: 
- There will be two sets of components for kube-acc to work: CRD based controllers running on control node, and kube-acc plugin running on compute node
- Kube-acc plugin should be able to run on bare metal or existing IaaS Acc Controllers (e.g OpenStack Cyborg) to take advantage of existing acc drivers
- The interface for kube-acc plugin to interact with kube-acc controller should be, in theory, a superset of DPI
- When co-existing and deployed DPI plugin and kube-acc plugin,
    - For locally attached accelerator scenario, i.e the life cycle of accelerator is more coupled with compute workload, users could use both plugins at the same time, and should rely on DPI plugin for main LCM functionality and only use kube-acc plugin for resource discovery/reporting; 
    - For remotely attached accelerator scenario, i.e the life cycle of accelerator is more decoupled from compute workload, it is recommended to use either DPI or kube-acc plugin, but not to use both at the same time.

# Challenges:
1.	How to better coordinate and co-evolve with DPI ? For example how to collaborate with some of the ongoing DPI enhancement work [1][2]

    Answer: none of the DPI ongoing work targets management for more than one type of accelerator altogether, which is where kube-acc proposition differentiates.

2.	If user utilize kube-acc for accelerator attachment, how could we bind the resource to the pod (using PIP ?)

# Reference:
[0] Extending Kubernetes* with Intel® accelerator devices - Alexander Kanevskiy

[1] Node Topology Manager https://github.com/kubernetes/enhancements/issues/693

[2] DPI topo info report https://github.com/kubernetes/kubernetes/pull/74423
