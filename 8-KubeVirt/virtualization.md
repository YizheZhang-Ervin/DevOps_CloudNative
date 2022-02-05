# Virtualization

## 云计算
- 虚拟化平台技术
  - OpenStack(IaaS)
    - 虚拟机计算、网络、存储资源管理
  - Kubernetes(PaaS)
    - 容器自动化部署、编排调度、发布管理

- 虚拟化
  - 类型1：原生/裸金属hypervisor
    - VMWare ESXi
    - KVM
    - Hyper-V
  - 类型2：hosted hypervisor
    - QEMU
    - VirtualBox
    - VMware Player
    - VMWare WorkStation

- 平台虚拟化管理
  - Libvirt


- K8S虚拟机管理
  - KubeVirt

- 相关概念
  - VM虚拟机，为VMI提供管理功能(开机/关机/重启)
  - VMI类似K8S Pod，管理虚拟机的最小资源
  - VirtualMachineInstanceReplicaSet类似K8S 的ReplicaSet
  - VirtualMachineInstanceMigrations，提供虚拟机迁移能力
  - 虚拟机镜像CDI
  - 磁盘和卷
    - cloudInitNoCloud初始化虚拟机配置信息
    - containerDisk镜像(重启数据会丢)
    - dataVolume PVC(重启数据不会丢)
    - emptyDisk固定容量空间(重启数据会丢)
    - ephermeral临时卷(重启数据会丢)
    - hostDisk镜像文件(重启数据不会丢)
    - configMap
    - serviceAccount
    - secret

- 网络
  - Multus CNI给pod中配置多网卡
  - 配置
    - 前端/后端
    - pod/multus
    - bridge/sriov/masquerade/macvtap
- 虚拟机迁移
  - kubectl apply -f vm.yaml
  - virtctl start testvm
  - virtctl migrate testvm