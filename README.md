# Cluster Platform Architecture

The lean cluster computing system.

## Overview

The Cluster Platform, or CTPF for short, is a collection of services that allow for the easy distribution, control and development of clusters. The nature of said clusters is flexible in nature; they can span anything from a collection of physical servers, virtual machines, mobile devices to IoT nodes and support complex topologies and dynamic changes.

Fundamentally, the CTPF provides three essential resources in clusters: compute, networking and storage. Whenever possible, already existing systems (such as Kubernetes or Linux) are used and extendend with simple and clearly scoped microservices, so using the CTPF should be fairly straight forward even for beginners with some knowledge of cloud computing. Improving the DevOps user experience is one of the primary goals of the project, and because of that a lot of work is put into making the creation and control of clusters simple and zero-configuration where possible.

Additionally, a lean and modern development stack is used. The development target is Kubernetes, Linux and the web, the development language is Go (for both the backends and frontends), APIs are done using gRPC and the entire system is being developed openly under the AGPL-3.0 license on GitHub. PRs are welcome, of course!

## Components

### Core

The CTPF core is composed of multiple independent microservices, which manage the basic infrastructure of the cluster - that is to say, enough to run Linux and Kubernetes on it.

#### liwasc

> List, wake and scan nodes in a network.

Liwasc is a network scanner and Wake-on-LAN server. It is responsible to get a overview of the nodes in a cluster and to wake them on command; this is particularly useful for "lights-off" datacenters or just generic remote managment. In short, it supports the following functionality through a gRPC API and a web app:

- Scan the network using ARP and subscribe to changes
- Lookup node manufacturer information using the mac2vendor database
- Do a TCP and UDP port scan on the nodes in the network and subscribe to changes
- Lookup port/service information using the IANA data
- Wake up nodes using Wake-on-LAN and subscribe to the wake state

For more information, check out [pojntfx/liwasc](https://github.com/pojntfx/liwasc).

#### necon

Neocon is modern DHCP server. It is low on ressource usage, can be configured using a gRPC API and a web app and is easy to deploy. Unlike many other existing DHCP servers, it also supports dynamic configuration through the API, which makes it possible to configure the network without much overhead. Features include:

- Full DHCP functionality
- Subscribing to leases
- Simple configuration through the gRPC API or the web app

For more information, check out [pojntfx/necon](https://github.com/pojntfx/necon).

#### bofied

Bofied is modern TFTP and HTTP server designed to distribute OS images by i.e. PXE booting. Like liwasc and necon, bofied is configured by a gRPC API or a web app, which makes switching the images to deploy very simple and also allows for quite complex setups without to much configuration or complex combinations of different services. Among other things, the following is possible:

- Serving iPXE or other network boot images using TFTP
- Selecting different boot files for different architectures
- Serving scripts, apkvols for Alpine Linux, repo mirrors, preseed files for Debian or SSH keys using HTTP and securely accessing them
- Full CRUD on files using the gRPC API or the web app

For more information, check out [pojntfx/bofied](https://github.com/pojntfx/bofied).

#### nodin

Nodin is a node inventory service; it is in charge of providing unified models and an API for managing nodes independend of deployment method or location. When combined, liwasc, necon and bofied provide full private cloud functionality, even without human oversight in a "lights-off" scenario; using nodin, it is possible to manage both nodes in this system as well as nodes across public cloud providers such as GCP, DigitalOcean or AWS using the same API. This makes it possible to dynamically add and remove both physical and virtual nodes in the cluster when needed. Using nodin, one can:

- List nodes across private and public cloud (using CTPF core services or a public cloud provider APIs)
- Create nodes (using CTPF core services or public cloud provider APIs)
- Power on and off nodes (using wascan or public cloud provider APIs)
- Delete nodes (using public cloud provider APIs)

For more information, check out [pojntfx/nodin](https://github.com/pojntfx/nodin).

#### gloeth

> Fast, lean and secure distributed layer 2 overlay networks

Gloeth is a new type of layer 2 VPN that support highly complex network topologies. It does so by providing strong end-to-end encryption, having no central point of failure and using a graph-based routing and frame forwarding system. Gloeth can be thought of as a globally scalable virtual ethernet switch; it makes it possible to easily traverse NATs and connect nodes across public cloud providers and private cloud datacenters without complex configuration and on layer 2. In combination with liwasc and necon, nestable and secure overlay networks can be created without ever leaving the CTPF core services. Some features include:

- Single static binary per node
- Zero-configuration graph based mesh frame routing & forwarding
- Strong TLS-protected node-to-node links and in-cluster end-to-end encryption between every node (no node except for the target node can listen on traffic in the cluster)
- No central point of failure

For more information, check out [pojntfx/gloeth](https://github.com/pojntfx/gloeth).

### Deployers

The CTPF deployers are tools that allow for easy deployment of the CTPF components from a developer workstation. Some deployers include:

- gloethdeployer (creates gloeth overlay networks without having to set up each node)
- k3sdeployer (creates Kubernetes clusters in gloeth overlay networks that can span multiple public and private clouds)

### Builders

The CTPF builders are services that build artifacts which would commonly have to be compiled manually. This is particularly useful due to the complex setups that are required to compile low-level software such as bootloaders; using the builders, it is possible to create many artifacts with the following builders in a Kubernetest cluster through a simple gRPC API or web app:

- ipxebuilder (creates custom iPXE network bootloaders with custom embedded scripts, this makes it possible to chainload a script from bofied for a fully automatic/headless installation over the network)
- grubbuilder (creates custom GRUB bootloaders for fully automatic installation over ISOs for amd64 UEFI systems)
- syslinuxbuilder (creates custom SYSLINUX bootloaders for fully automatic installation over ISOs for amd64 BIOS systems)
- ubootbuilder (creates custom U-Boot bootloaders for fully automatic installations over IMGs for arm64 systems and IoT devices in general, such as Raspberry Pis)
- isobuilder (creates bootable ISOs for amd64 systems for non-network based automatic installations; this is particularly useful for joining remote solo nodes into a CTPF cluster)
- imgbuilder (creates IMGs for arm64 systems that allow for non-network based automatic installations; this allows for IoT devices without reliable or fast internet connections to be used in a CTPF cluster)
- rpibuilder (patches IMGs with Raspbery Pi usercfgs and binary blobs)
- alpinebuilder (builds custom Alpine Linux images, apkvols, repo mirrors and more)

## Progress

The Cluster Platform is in a constant, but still an early stage of it's development with scopes and APIs still changing rapidly. Due to it's architecture of many independent services, it is already possible to use some services, while others are not yet production ready or are still in the planning phase. Please check the individual repositories for more informations on how to contribute or the individual status reports.

## License

Cluster Platform (c) 2020 Felicitas Pojtinger

SPDX-License-Identifier: AGPL-3.0
