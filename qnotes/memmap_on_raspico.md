# memory map on RasPico

Here will go over the memory map view on Raspberry Pico.



# Top Level View

There is a reference 2.2. Address Map** in  `rp2040-datasheet.pdf`  to give a big picture. 

> **2.2. Address Map**
> The address map for the device is split in to sections as shown in Table 15. Details are shown in the following sections. Unmapped address ranges raise a bus error when accessed.
> **2.2.1. Summary**
> Table 15. Address Map Summary
>
> |        Function Block         | Starting Address |
> | :---------------------------: | :--------------: |
> |              ROM              |    0x00000000    |
> |              XIP              |    0x10000000    |
> |             SRAM              |    0x20000000    |
> |        APB Peripherals        |    0x40000000    |
> |     AHB-Lite Peripherals      |    0x50000000    |
> |       IOPORT Registers        |    0xd0000000    |
> | Cortex-M0+ internal registers |    0xe0000000    |
>
> 

**2.1. Bus Fabric** in `rp2040-datasheet.pdf ` provides a picture for top level components interconnection.

# (Real) Memory 

refer to **2.6. Memory** in `rp2040-datasheet.pdf`

> **2.6. Memory**
> RP2040 has embedded ROM and SRAM, and access to external Flash via a QSPI interface. Details of internal memory are given below.
>
> **2.6.1. ROM**
> A 16kB read-only memory (ROM) is at address 0x00000000 . The ROM contents are fixed at the time the silicon is manufactured. It contains:
> 	• Initial startup routine
>
> ​	• Flash boot sequence
> ​	• Flash programming routines
> ​	• USB mass storage device with UF2 support
> ​	• Utility libraries such as fast floating point
> The boot sequence of the chip is defined in Section 2.8.1, and the ROM contents is described in more detail in Section 2.8. The full source code for the RP2040 bootrom is available at: 
> https://github.com/raspberrypi/pico-bootrom
> The ROM offers single-cycle read-only bus access, and is on a dedicated AHB-Lite arbiter, so it can be accessed simultaneously with other memory devices. Attempting to write to the ROM has no effect (no bus fault is generated).
>
> **2.6.2. SRAM**
> There is a total of 264kB of on-chip SRAM. Physically this is partitioned into six banks, as this vastly improves memory bandwidth for multiple masters, but software may treat it as a single 264kB memory region. There are no restrictions on what is stored in each bank: processor code, data buffers, or a mixture. There are four 16k x 32-bit banks (64kB each) and two 1k x 32-bit banks (4kB each).
>
> ** IMPORTANT**
> Banking is a physical partitioning of SRAM which improves performance by allowing multiple simultaneous accesses. Logically there is a single 264kB contiguous memory.
> Each SRAM bank is accessed via a dedicated AHB-Lite arbiter. This means different bus masters can access different SRAM banks in parallel, so up to four 32-bit SRAM accesses can take place every system clock cycle (one per master). 
>
> SRAM is mapped to system addresses starting at 0x20000000 . The first 256kB address region is word-striped across the four larger banks, which provides a significant memory parallelism benefits for most use cases. 
>
> The next two 4kB regions (starting at 0x20040000 and 0x20041000 ) are mapped directly to the smaller, 4kB memory banks. Software may choose to use these for per-core purposes, e.g. stack and frequently-executed code, guaranteeing that the processors never stall on these accesses. However, like all SRAM on RP2040, these banks have single-cycle access from all masters providing no other masters are accessing the bank in the same cycle, so it is reasonable to treat memory as a single 264kB device.
> The four 64kB banks are also available at a non-striped mirror. The four 64kB regions starting at 0x21000000 , 0x21010000 , 0x21020000 , 0x21030000 are each mapped directly to one of the four 64kB SRAM banks. Software can explicitly allocate data and code across the physical memory banks, for improved memory performance in exceptionally demanding cases. This is often unnecessary, as memory striping usually provides sufficient parallelism with less software complexity. The non-striped mirror starts at an offset of +16MB above the base of SRAM, as this is the maximum offset that allows ARMv6M subroutine calls between the smaller banks and the non-striped larger banks.
>
> **2.6.2.1. Other On-chip Memory**
> Besides the 264kB main memory, there are two other dedicated RAM blocks that may be used in some circumstances:
> 	• If flash XIP caching is disabled, the cache becomes available as a 16kB memory starting at 0x15000000
> 	• If the USB is not used, the USB data DPRAM can be used as a 4kB memory starting at 0x50100000
> This gives a total of 284kB of on-chip SRAM. There are no restrictions on how these memories are used, e.g. it is possible to execute code from the USB data RAM if you choose.
>
> **2.6.3. Flash**
> External Flash is accessed via the QSPI interface using the execute-in-place (XIP) hardware. This allows an external flash memory to be addressed and accessed by the system as though it were internal memory. Bus reads to a 16MB memory window starting at 0x10000000 are translated into a serial flash transfer, and the result is returned to the master that initiated the read. This process is transparent to the master, so a processor can execute code from the external flash without first copying the code to internal memory, hence "execute in place". An internal cache remembers the
> contents of recently-accessed flash locations, which accelerates the average bandwidth and latency of the interface.
> Once correctly configured by RP2040’s bootrom and the flash second stage, the XIP hardware is largely transparent, and software can treat flash as a large read-only memory. However, it does provide a number of additional features to serve more demanding software use cases.



# APB Peripherals

start from  0x40000000

|                          |            |
| :----------------------: | :--------: |
|       SYSINFO_BASE       | 0x40000000 |
|       SYSCFG_BASE        | 0x40004000 |
|       CLOCKS_BASE        | 0x40008000 |
|       RESETS_BASE        | 0x4000c000 |
|         PSM_BASE         | 0x40010000 |
|      IO_BANK0_BASE       | 0x40014000 |
|       IO_QSPI_BASE       | 0x40018000 |
|     PADS_BANK0_BASE      | 0x4001c000 |
|      PADS_QSPI_BASE      | 0x40020000 |
|        XOSC_BASE         | 0x40024000 |
|       PLL_SYS_BASE       | 0x40028000 |
|       PLL_USB_BASE       | 0x4002c000 |
|       BUSCTRL_BASE       | 0x40030000 |
|        UART0_BASE        | 0x40034000 |
|        UART1_BASE        | 0x40038000 |
|        SPI0_BASE         | 0x4003c000 |
|        SPI1_BASE         | 0x40040000 |
|        I2C0_BASE         | 0x40044000 |
|        I2C1_BASE         | 0x40048000 |
|         ADC_BASE         | 0x4004c000 |
|         PWM_BASE         | 0x40050000 |
|        TIMER_BASE        | 0x40054000 |
|      WATCHDOG_BASE       | 0x40058000 |
|         RTC_BASE         | 0x4005c000 |
|        ROSC_BASE         | 0x40060000 |
| VREG_AND_CHIP_RESET_BASE | 0x40064000 |
|        TBMAN_BASE        | 0x4006c000 |



# AHB-Lite peripherals

start from 0x50000000

|                    |            |
| :----------------: | :--------: |
|      DMA_BASE      | 0x50000000 |
|    USBCTRL_BASE    | 0x50100000 |
| USBCTRL_DPRAM_BASE | 0x50100000 |
| USBCTRL_REGS_BASE  | 0x50110000 |
|     PIO0_BASE      | 0x50200000 |
|     PIO1_BASE      | 0x50300000 |
|    XIP_AUX_BASE    | 0x50400000 |



# IOPORT Peripherals

start from 0xd0000000

|          |            |
| :------: | :--------: |
| SIO_BASE | 0xd0000000 |



# Cortex-M0+ Internal Peripherals

start from 0xe0000000 , seperated per-core.

|          |            |
| :------: | :--------: |
| PPB_BASE | 0xe0000000 |



> **2.4.1.2. Configuration**
> Each processor is configured with the following features:
> 	• Architectural clock gating (for power saving)
> 	• Little Endian bus access
> 	• Four Breakpoints
> 	• Debug support (via 2-wire debug pins SWD / SWCLK )
> 	• 32-bit instruction fetch (to match 32-bit data bus)
> 	• IOPORT (for low latency access to local peripherals (see SIO)
> 	• 26 interrupts
> 	• 8 MPU regions
> 	• All registers reset on powerup
> 	• Fast multiplier (MULS 32x32 single cycle)
> 	• SysTick timer
> 	• Vector Table Offset Register (VTOR)
> 	• 34 WIC (Wake-up Interrupt Controller) lines (32 IRQ and NMI, RXEV)
> 	• DAP feature: Halt event support
> 	• DAP feature: SerialWire debug interface (protocol 2 with multidrop support)
> 	• DAP feature: Micro Trace Buffer (MTB) is not implemented



> **2.4.8. List of Registers**
>
> The ARM Cortex-M0+ registers start at a base address of 0xe0000000 (defined as PPB_BASE in SDK).
>
> | Offset |    Name    |                       Info                       |
> | :----: | :--------: | :----------------------------------------------: |
> | 0xe010 |  SYST_CSR  |       SysTick Control and Status Register        |
> | 0xe014 |  SYST_RVR  |          SysTick Reload Value Register           |
> | 0xe018 |  SYST_CVR  |          SysTick Current Value Register          |
> | 0xe01c | SYST_CALIB |        SysTick Calibration Value Register        |
> | 0xe100 | NVIC_ISER  |          Interrupt Set-Enable Register           |
> | 0xe180 | NVIC_ICER  |         Interrupt Clear-Enable Register          |
> | 0xe200 | NVIC_ISPR  |          Interrupt Set-Pending Register          |
> | 0xe280 | NVIC_ICPR  |         Interrupt Clear-Pending Register         |
> | 0xe400 | NVIC_IPR0  |          Interrupt Priority Register 0           |
> | 0xe404 | NVIC_IPR1  |          Interrupt Priority Register 1           |
> |        |            |                                                  |
> | 0xe418 | NVIC_IPR6  |          Interrupt Priority Register 6           |
> | 0xe41c | NVIC_IPR7  |          Interrupt Priority Register 7           |
> | 0xed00 |   CPUID    |               CPUID Base Register                |
> | 0xed04 |    ICSR    |       Interrupt Control and State Register       |
> | 0xed08 |    VTOR    |           Vector Table Offset Register           |
> | 0xed0c |   AIRCR    | Application Interrupt and Reset Control Register |
> | 0xed10 |    SCR     |             System Control Register              |
> | 0xed14 |    CCR     |        Configuration and Control Register        |
> | 0xed1c |   SHPR2    |        System Handler Priority Register 2        |
> | 0xed20 |   SHPR3    |        System Handler Priority Register 3        |
> | 0xed24 |   SHCSR    |    System Handler Control and State Register     |
> | 0xed90 |  MPU_TYPE  |                MPU Type Register                 |
> | 0xed94 |  MPU_CTRL  |               MPU Control Register               |
> | 0xed98 |  MPU_RNR   |            MPU Region Number Register            |
> | 0xed9c |  MPU_RBAR  |         MPU Region Base Address Register         |
> | 0xeda0 |  MPU_RASR  |      MPU Region Attribute and Size Register      |
> |        |            |                                                  |

## M0PLUS: VTOR (Vector Table Offset) Register 

> **M0PLUS: VTOR Register**
> Offset: 0xed08
> Description
> The VTOR holds the vector table offset address.
>
> | Bits |   Name    |                         Description                          | Type |  Reset   |
> | :--: | :-------: | :----------------------------------------------------------: | :--: | :------: |
> | 31:8 |  TBLOFF   | Bits [31:8] of the indicate the vector table offset address. |  RW  | 0x000000 |
> | 7:0  | Reserved. |                              -                               |  -   |    -     |
>
> 

As this is an implementation per-core, it is possible to have core_0 and core_1 set a different Vector Table in RAM, in separated/dedicated RAM bank, could possible to improve the performance of IRQ response, as well as execution speed. 

To be aware that the low 8bit in VTOR is reserved zeroed out, this makes it can't simply define a 48 elements uint32_t array for the Vector_Table.  The address has to be at 8-bits aligned, so it came to be  , 0x2000 0000 , 0x2000 0400 , 0x2000 0800 etc. 

For RP2040 the real Vector_Table needs only 48 words, which is total 0x30 * 4 = 0xC0 bytes space. 0x400 is 1K, which is a much overkilled to 0x0c0. The last 2 of 4K-RAM-Bank may configured as Vector_Table and ISR_Stack_Space for each core respectively, to gain better performance.  Starting at 0x20040000 and 0x20041000 ,  regions of 4K-byte space. 
