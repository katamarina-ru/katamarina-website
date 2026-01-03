---
title: "Руководство Пользователя платформы"
linkTitle: "Руководство Пользователя платформы"
description: "Руководство Пользователя платформы"
weight: 43
---

# Interactive Dashboard

> A tool to inspect the running Talos machine state on the physical video console.

Interactive dashboard is enabled for all Talos platforms except for SBC images.
The dashboard can be disabled with kernel parameter `talos.dashboard.disabled=1`.

The dashboard runs only on the physical video console (not serial console) on the 2nd virtual TTY.
The first virtual TTY shows kernel logs same as in Talos `<1.4.0>`.
The virtual TTYs can be switched with `<Alt+F1>` and `<Alt+F2>` keys.

Keys `<F1>` - `<Fn>` can be used to switch between different screens of the dashboard.

The dashboard is using either UEFI framebuffer or VGA/VESA framebuffer (for legacy BIOS boot).

## Dashboard Resolution Control

On legacy BIOS systems, the screen resolution can be adjusted with the [`vga=` kernel parameter](https://docs.kernel.org/fb/vesafb.html).

In modern kernels and platforms, this parameter is often ignored. For reliable results, it is recommended to boot with **UEFI**.

When running in **UEFI mode**, you can set the screen resolution through your hypervisor or UEFI firmware settings.

## Summary Screen (`F1`)

<img src="https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=d35339535e51eb7b15456a5af52b8404" alt="Interactive Dashboard Summary Screen" width="920" data-og-width="1556" data-og-height="1234" data-path="talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?w=280&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=e14103d301b3ef32aa72c7e86c35336e 280w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?w=560&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=bb8736e7d213e21f1626c415ab79ac78 560w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?w=840&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=7f46f23ed09dd6514cd2142e96ed4442 840w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?w=1100&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=0a95e859089462874314eed0f0fbf3c3 1100w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?w=1650&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=c441116ab6ca1b037ef5756d725e5f53 1650w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?w=2500&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=974f517194643fca7d4a6a2443fadc26 2500w" />

The header shows brief information about the node:

* hostname
* Talos version
* uptime
* CPU and memory hardware information
* CPU and memory load, number of processes

Table view presents summary information about the machine:

* UUID (from SMBIOS data)
* Cluster name (when the machine config is available)
* Machine stage: `Installing`, `Upgrading`, `Booting`, `Maintenance`, `Running`, `Rebooting`, `Shutting down`, etc.
* Machine stage readiness: checks Talos service status, static pod status, etc. (for `Running` stage)
* Machine type: controlplane/worker
* Number of members discovered in the cluster
* Kubernetes version
* Status of Kubernetes components: `kubelet` and Kubernetes controlplane components (only on `controlplane` machines)
* Network information: Hostname, Addresses, Gateway, Connectivity, DNS and NTP servers

Bottom part of the screen shows kernel logs, same as on the virtual TTY 1.

## Monitor Screen (`F2`)

<img src="https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=151073a1b4f39370e311c3967a01966d" alt="Interactive Dashboard Summary Screen" width="920" data-og-width="1556" data-og-height="1234" data-path="talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?w=280&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=03aebccdc1abf22cf841796dfe29652a 280w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?w=560&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=5e07a80a15f329b389cbc7e2a1ebf1e4 560w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?w=840&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=20e466fed281c7aa0c48ec45b1b6789d 840w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?w=1100&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=8a98b536f7a84ca01b17dbd9d5ceff94 1100w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?w=1650&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=124a8b59ee91508ee094b422831ff8e0 1650w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?w=2500&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=719bb7c8178068437e57b222c842ab23 2500w" />

Monitor screen provides live view of the machine resource usage: CPU, memory, disk, network and processes.

## Network Config Screen (`F3`)

> Note: network config screen is only available for `metal` platform.

<img src="https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=2aed8497fba28c5c79bb8bcce95187b8" alt="Interactive Dashboard Summary Screen" width="920" data-og-width="1556" data-og-height="1234" data-path="talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?w=280&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=0c79880a78f985fca04ef3c2471e015e 280w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?w=560&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=fb97a0c9c2f4b08c1719534504155e55 560w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?w=840&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=c03944da93c166b50f9085d26f7ba858 840w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?w=1100&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=8d346f280ac480635fea29e156e4f41c 1100w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?w=1650&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=0cf7393ee7d9963fe48ac2d9504254b7 1650w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?w=2500&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=cb99976afffc0984cf300f89ae389cf8 2500w" />

Network config screen provides editing capabilities for the `metal` [platform network configuration](../platform-specific-installations/bare-metal-platforms/network-config).

The screen is split into three sections:

* the leftmost section provides a way to enter network configuration: hostname, DNS and NTP servers, configure the network interface either via DHCP or static IP address, etc.
* the middle section shows the current network configuration.
* the rightmost section shows the network configuration which will be applied after pressing "Save" button.

Once the platform network configuration is saved, it is immediately applied to the machine.


---

> To find navigation and other pages in this documentation, fetch the llms.txt file at: https://docs.siderolabs.com/llms.txt
