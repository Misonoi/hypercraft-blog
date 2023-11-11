手册部分内容的自用翻译

## 8.6 Traps

虚拟化扩展增加了Trap原因编码。表 8.6 列出了在实现虚拟化扩展时可能出现的 M 模式和 HS 模式陷阱原因代码。为 VS 级中断（中断 2、6、10）、监管级别客户机外部中断（中断 12）、虚拟指令异常（异常 22）以及客户机页错误（异常 20、21、23）添加了代码。

此外，来自 VS 模式的环境调用被分配为原因 10，而来自 HS 模式或 S 模式的环境调用则像往常一样使用原因 9。HS 模式和 VS 模式的 ECALL 使用不同的原因值，因此它们可以分别委派。

当 V=1 时，如果尝试执行的指令是 HS 合格的，但由于权限不足或由监管员或虚拟化扩展的 CSR（如 scounteren 或 hcounteren）明确禁用，导致无法执行，通常会引发虚拟指令异常（代码 22），而不是非法指令异常。如果 CSR mstatus 的 TSR 和 TVM 字段都为零，则指令是 HS 合格的，假定 CSR mstatus 的 TSR 和 TVM 字段都为零。

对于访问 32 位高半部分 CSR（如 cycleh 和 htimedeltah）的 CSR 指令，有特殊规则。当 V=1 且 XLEN>32 时，尝试访问高半部分监管级别 CSR、高半部分虚拟化扩展 CSR、高半部分 VS CSR 或高半部分非特权 CSR 始终引发非法指令异常。在 VS 模式中，如果 VU 模式的 XLEN 大于 32，则尝试访问高半部分用户级 CSR（与非特权 CSR 不同）始终引发非法指令异常。另一方面，当 V=1 且 XLEN=32 时，如果对高半部分 S 级、hypervisor、VS 或非特权 CSR 的无效尝试是 HS 合格的，则引发虚拟指令异常，而不是非法指令异常。同样，在 VS 模式中，如果 VU 模式的 XLEN 为 32，对高半部分用户级 CSR 的无效尝试是 HS 合格的，则引发虚拟指令异常，而不是非法指令异常。

具体而言，对于以下情况会引发虚拟指令异常：

1. 在 VS 模式中，尝试访问非高半部分计数器 CSR，当相应的 hcounteren 中的位为 0 且相同位在 mcounteren 中为 1 时。
2. 在 VS 模式中，如果 XLEN=32，尝试访问高半部分计数器 CSR，当相应的 hcounteren 中的位为 0 且相同位在 mcounteren 中为 1 时。
3. 在 VU 模式中，尝试访问非高半部分计数器 CSR，当相应的 hcounteren 或 scounteren 中的位为 0 且相同位在 mcounteren 中为 1 时。
4. 在 VU 模式中，如果 XLEN=32，尝试访问高半部分计数器 CSR，当相应的 hcounteren 或 scounteren 中的位为 0 且相同位在 mcounteren 中为 1 时。

5. 在 VS 模式或 VU 模式中， 尝试执行虚拟化指令（HLV、HLVX、HSV 或 HFENCE）。

6. 尝试访问已实现的非高半部分虚拟化扩展 CSR 或 VS CSR，当假定在 HS 模式下允许相同的访问（读 / 写），假设 mstatus.TVM=0。

7. 如果 XLEN=32，在 VS 模式或 VU 模式中，尝试访问已实现的高半部分虚拟化扩展 CSR 或高半部分 VS CSR，当假定在 HS 模式下允许对相同 CSR 低半部分伙伴的相同访问（读 / 写），假设 mstatus.TVM=0。

8. 在 VU 模式中，尝试执行 WFI，当 mstatus.TW=0 时，或尝试执行监管员指令（SRET 或 SFENCE）。

9. 在 VU 模式中，尝试访问已实现的非高半部分监管员 CSR，当假定在 HS 模式下允许相同的访问（读 / 写），假设 mstatus.TVM=0。

10. 如果 XLEN=32，在 VU 模式中，尝试访问已实现的高半部分监管员 CSR，当假定在 HS 模式下允许对相同 CSR 低半部分伙伴的相同访问（读 / 写），假设 mstatus.TVM=0。

11. 在 VS 模式中，尝试执行 WFI，当 hstatus.VTW=1 且 mstatus.TW=0 时，除非指令在实现特定的有界时间内完成。

12. 在 VS 模式中，尝试执行 SRET，当 hstatus.VTSR=1 时。

13. 在 VS 模式中，尝试执行 SFENCE.VMA 或 SINVAL.VMA 指令，或访问 satp，当 hstatus.VTVM=1 时。

RISC-V 特权体系结构的其他扩展可能会在 V=1 时增加引发虚拟指令异常的情形。在虚拟指令陷阱时，mtval 或 stval 的写入方式与非法指令陷阱相同。通常，虚拟化扩展需要模拟引发虚拟指令异常的指令，以支持嵌套虚拟化扩展或其他原因。机器级别通常期望将虚拟指令陷阱直接委派给 HS 级别，而非法指令陷阱可能会在首先在 M 模式中处理后（由软件）有条件地委派给 HS 级别。因此，虚拟指令陷阱通常预计会比非法指令陷阱更快地处理。

当不模拟陷阱指令时，虚拟机监控程序应将虚拟指令陷阱转换为客户虚拟机的非法指令异常。

由于 mstatus 中的 TSR 和 TVM 仅用于影响 S 模式（HS 模式），在确定 VS 模式中的异常时会忽略它们。

如果一条指令可能引发多个同步异常，Table 8.7 的降低优先级顺序表示取哪个异常，并将其报告在 mcause 或 scause 中。

#### Trap Entry

当一次Trap发生在HS-mode 或是 U-mode时会进入M-mode, ，除非这个trap通过`medeleg` or `mideleg`委托给了`HS-mode`处理。 当一次`Trap`发生在VS-mode或VU-mode, 会进入M-mode, 除非这个trap通过`medeleg`或者`mideleg`委托给了HS-mode, 如果再通过`hedeleg` or `hideleg`委托就会进入VS-mode处理

当在M-mode处理trap时， 设置V = 0, 然后如下表设置mstatus的`MPV and MPP`， 也会写入`mstatus`的`GVA MPIE MIE`, 和写入`mepc, mcause, mtval, mtval2, mtinst`

当在HS-mode处理时， 设置V = 0, 然后如表设置`hstatus.SPV and sstatus.SPP`, 如果`trap`前V = 1, `hstatus.SPVP = sstatus.SPP`, ，否则不变。 也会写入`htatus.GVA, sstatus.SPIE&SIE`, `spec, scause, stval, htval`

当在VS-mode处理时， 设置`vsstatus.SPP`, 不改变`hstatus sstatus`和V, 修改`vsstatus.SPIE&SIE`和`vsepc, vscause, vstval`

### Trap Return

MRET 指令用于从陷入 M 模式的Trap中返回。MRET 首先根据 mstatus 或 mstatush 中的 MPP 和 MPV 的值，在 Table 8.8 中编码，确定新的特权模式。然后，在 mstatus/mstatush 中设置 MPV=0，MPP=0，MIE=MPIE，并将 MPIE=1。最后，MRET 设置特权模式为先前确定的值，并将 pc=mepc。

SRET 指令用于从陷入 HS 模式或 VS 模式的陷阱中返回。其行为取决于当前的虚拟化模式。

当在 M 模式或 HS 模式执行时（即 V=0），SRET 首先根据 hstatus.SPV 和 sstatus.SPP 中的值，在 Table 8.9 中编码，确定新的特权模式。然后，在 hstatus 中设置 SPV=0，在 sstatus 中设置 SPP=0，SIE=SPIE，并将 SPIE=1。最后，SRET 设置特权模式为先前确定的值，并将 pc=sepc。

当在 VS 模式执行时（即 V=1），SRET 根据 Table 8.10 设置特权模式，在 vsstatus 中设置 SPP=0，SIE=SPIE，并将 SPIE=1，并最后设置 pc=vsepc。
