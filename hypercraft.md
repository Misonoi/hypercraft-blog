## hypercraft 设计理念与架构

H拓展新增了几个特权级

- `HS`: S mode with hypervisor capabilities and new CSRs.
- `Virtualized S-mode`: 替代传统`S-mode`
- `VU-mode`: Virtualized U-mode 提到原有的`U-mode`

新增了一些控制状态寄存器

- `HS-mode CSRs`
    - `hstatyus`: Hypervisor Status 特权级切换
    - `hideleg`: Hypervisor Interrupt Delegate 中断代理
    - `hedeleg`: Hypervisor Trap/Exception Delegate 异常代理
    - `htimedelta`: Hypervisor Guest Time Delta 不太重要
    - `hgatp`: Hypervisor Guest Address Translation 
- `VS-mode CSRs` 和虚拟化关系不大
    - `vsstatus`
    - `vsie`
    - `vsip`
    - `vstvec`
    - `vsepc`
    - `vscause`
    - `vstval`
    - `vsatp`
    - `vsscrath`

### RISC-V G Stage

- `vsatp` 用于第一阶段页表翻译
- `hgatp` 用于第二阶段页表翻译
- GVA -> GPA -> HPA

### RISC-V Interrupt Virtualization Technology

- `PLIC`
    - 不支持MSI
    - 不支持中断投递
    - 需要在`hypervisor`中模拟`PLIC`进行中断注入
- `AIA(Advanced Interrupt Architecture)`
    - `IMSIC` 支持`MSI` & `IPI Virtualization`
    - `VS-mode`下运行的客户端操作系统对设备中断的直接控制， 减少虚拟机监视器的干预
    - `APLIC` 更高效地处理中断

### RISC-V I/O Virtualization Technology

- `Emulate`
    - 纯软件模拟
    - 平台稳定
    - 性能低
- `Passthrough`
    - `VM`独占`Guest`
    - 性能高
    - 需要大量设备

### hypercraft Overview

![image-20231110200109861](./images/1)

![image-20231110200623374](./images/2)