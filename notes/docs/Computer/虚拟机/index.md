# 引言

- 私有服务器：全部资源和权限
- 虚拟专用服务器（Virtual Private Server, VPS）：一台主机虚拟出多个服务器，资源共享，可自主安装各种软件
- 虚拟主机：资源共享，权限小，只允许安装特定软件

## VPS

国内

> 对外提供网站服务需要备案

- 阿里云 ECS(Elastic Compute Service)
- 腾讯云 CVM(Cloud Virtual Machine)

国外

- AWS(Amazon Web Services)
  - EC2(Elastic Compute Cloud)
  - ECS(Elastic Container Service)
  - EKS(Elastic Kubernetes Service)
- BandwagonHost
- Vultr
- Linode

## 虚拟机

Hypervisor（KVM、Xen等）是虚拟机最主要的部分。它通过硬件虚拟化功能，模拟出了运行一个操作系统需要的各种硬件，比如 CPU、内存、I/O 设备、磁盘、网卡等等。然后，它在这些虚拟的硬件上安装了一个新的操作系统，即 Guest OS。

虚拟机和K8s都很吃内存，所以建议宿主机内存8G+（可以支持开两个虚拟机组成最小集群）4核CPU，硬盘300G+

常见的虚拟机软件有

- Parallels 付费
- VMWare Workstation 付费（Player为免费版，Fusion适用于Mac）
- VirtualBox 免费
- UTM 免费（仅适用于Mac）

## QEMU

> <https://qemu.org>

QEMU是开源的纯软件实现的虚拟化模拟器，几乎可以模拟任何硬件设备。有两种主要工作模式

- 用户工作模式（仿真器）：可以在一种架构（如 x86 PC）下运行另一种架构（如 ARM）下的操作系统和程序，通过使用动态转换，它可以获得非常好的性能。
- 系统工作模式（虚拟机）：能模拟整个计算机系统，包括 CPU 及其他 IO 设备。它能运行和调试不同平台开发的操作系统，也能在宿主机上虚拟不同数量、不同平台的虚拟电脑。

由于是软件虚拟化，性能比较差，通常只用于模拟 I/O 设备，而 CPU 和内存等由 KVM 等硬件虚拟化技术实现

与其他的虚拟化软件（如 VMware）不同，QEUM 不提供管理虚拟机的 GUI（运行虚拟机时出现的窗口除外），也不提供创建具有已保存设置的持久虚拟机的方法。因此，我们可以选择创建自定义脚本以启动虚拟机，这样就可以保存相应的参数，避免每次启动时手动指定所有运行参数。

```shell
sudo apt install qemu
```
