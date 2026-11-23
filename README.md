# AngelOS
[ THE PROJECT IS WITHIN THE COMPRESSED ZIP FILE BECAUSE I AM TOO LAZY TO UPLOAD THE FILES. IT IS NOT A ZIP BOMB OR MALICIOUS SOFTWARE. ]
## HaloRoot — A 64-bit Operating System

AngelOS is an independently developed operating system written primarily in C and x86-64 assembly. It is designed to explore the internal architecture of operating systems through direct implementation of kernel functionality, memory management, hardware communication, storage systems, filesystems, and command-line interfaces.

AngelOS is built from the ground up using the Limine boot protocol. It targets the x86-64 architecture and is currently developed and tested primarily in QEMU.

The operating system contains its own kernel, physical and virtual memory management, dynamic heap allocator, interrupt subsystem, hardware drivers, ATA storage interface, custom filesystem, interactive shell, and built-in text editor.

The underlying kernel is known as **HaloRoot**.

The primary objective of the project is to progressively develop a functional operating system without depending on an existing general-purpose operating system kernel.

This document describes the architecture, implementation, interfaces, internal operation, current capabilities, and future development of AngelOS.

---

# Table of Contents

1. Introduction
2. Project Philosophy
3. General Technical Information
4. Kernel Architecture
5. Bootloader Integration
6. Kernel Entry and Initialization
7. Physical Memory Manager
8. Virtual Memory Manager
9. Kernel Heap
10. Interrupt Descriptor Table
11. Programmable Interrupt Controller
12. Programmable Interval Timer
13. Keyboard Driver
14. Serial Driver
15. Real-Time Clock
16. ATA Storage Driver
17. ACPI Power Management
18. HaloRoot Filesystem
19. Filesystem Disk Format
20. Filesystem Path Resolution
21. Filesystem Operations
22. Filesystem Persistence
23. AngelOS Shell Architecture
24. Command Parsing and Dispatch
25. Shell Input System
26. Command History
27. Command Aliases
28. Tab Completion
29. Built-in Text Editor
30. Disk Utilities
31. Date, Time, and Uptime
32. System Reboot and Shutdown
33. Kernel Diagnostics
34. Building AngelOS
35. Running AngelOS in QEMU
36. Persistent Storage Configuration
37. Complete Command Reference
38. Example Workflows
39. Current Implementation Limitations
40. Future Development
41. Contributing

---

# 1. Introduction

AngelOS is a custom operating system project focused on implementing the fundamental mechanisms that allow a computer to operate independently.

A traditional application runs on top of an existing operating system. It relies on the operating system to provide memory allocation, hardware access, storage, process management, and other services.

AngelOS takes a different approach.

It implements these facilities as part of its own kernel.

The kernel communicates with the processor, manages memory, receives hardware interrupts, interacts with storage devices, and provides an environment through which the system can be controlled.

The project has developed through several major stages.

### Initial kernel development

The first major objective was establishing a bootable x86-64 kernel.

This involved:

- Creating an assembly entry point.
- Integrating the Limine boot protocol.
- Establishing the higher-half kernel environment.
- Creating a linker script.
- Transferring execution into C code.
- Initializing serial output.
- Validating bootloader information.

### Memory management

After the initial kernel environment was established, memory management was introduced.

This involved developing:

- Physical page allocation.
- Physical page tracking.
- Reserved memory handling.
- Virtual memory integration.
- Higher Half Direct Map access.
- Dynamic kernel allocation.

### Hardware interaction

The next stage introduced direct communication with hardware.

This included:

- IDT installation.
- CPU exception handling.
- PIC configuration.
- PIT timer interrupts.
- PS/2 keyboard input.
- RTC access.
- ATA disk communication.
- ACPI system control.

### Storage and filesystem development

AngelOS subsequently gained a custom filesystem.

This allowed the kernel to organize data into files and directories and store filesystem information on an ATA-backed disk.

The filesystem supports path resolution, directory navigation, file creation, reading, writing, and removal.

### Interactive environment

The kernel also includes an interactive command-line shell.

The shell provides access to system information, filesystem operations, disk utilities, and system controls.

A basic text editor is also integrated into the shell.

---

# 2. Project Philosophy

AngelOS follows a modular, incremental approach to operating system development.

Each subsystem is responsible for a particular area of functionality.

For example, the filesystem does not need to understand how keyboard scancodes are translated. The keyboard driver does not need to understand the filesystem's disk layout. The shell interacts with both through defined functions.

This separation makes it possible to develop individual components without unnecessarily modifying unrelated parts of the kernel.

## 2.1 Implementation from the ground up

The project emphasizes direct implementation of operating system functionality.

Examples include:

- Writing to hardware I/O ports.
- Installing interrupt handlers.
- Allocating physical memory.
- Managing kernel heap allocations.
- Reading ATA sectors.
- Serializing filesystem metadata.
- Parsing shell commands.
- Processing keyboard input.

The intention is to understand how these mechanisms work internally rather than treating them as black boxes.

## 2.2 Incremental development

New functionality is introduced in stages.

A subsystem should first work independently, then be integrated into the kernel initialization sequence.

This is particularly important for components such as memory management and interrupts, because failures in these areas can prevent the entire operating system from starting.

## 2.3 Persistent data

Persistent storage is a central part of AngelOS.

A filesystem that loses everything after a reboot is useful for early development, but it is not sufficient for a practical operating system.

The ATA driver and HaloRoot filesystem provide the foundation for retaining data across restarts.

## 2.4 Future independence

The long-term objective is to develop an environment capable of running independently compiled applications, managing multiple processes, providing a graphical interface, and supporting a wider range of hardware.

---

# 3. General Technical Information

| Property | Implementation |
|---|---|
| Operating system | AngelOS |
| Kernel | HaloRoot |
| Architecture | x86-64 |
| Main language | C |
| Assembly | NASM |
| Bootloader | Limine |
| Kernel architecture | Monolithic |
| Kernel memory layout | Higher-half |
| Virtual address base | `0xffffffff80000000` |
| Physical page size | 4096 bytes |
| Memory management | PMM, VMM, kernel heap |
| Interrupt management | IDT and PIC |
| Timer | PIT |
| Keyboard | PS/2-compatible |
| Hardware clock | RTC |
| Storage | ATA PIO |
| Filesystem | Custom HaloRoot filesystem |
| Filesystem identifier | `HALORFS1` |
| Maximum filesystem nodes | 128 |
| Maximum file content | 511 bytes |
| User interface | Interactive serial shell |
| Development environment | Linux |
| Main emulator | QEMU |

## 3.1 Kernel address layout

AngelOS uses a higher-half virtual address layout.

The kernel's virtual base is:

```text
0xffffffff80000000
```

The higher-half arrangement places kernel memory in the upper region of the canonical x86-64 virtual address space.

The kernel receives physical and virtual address information from Limine.

These addresses are important for calculating the relationship between the kernel's linked virtual addresses and its physical location in memory.

## 3.2 Execution environment

AngelOS currently operates as a monolithic kernel.

The kernel, shell, drivers, filesystem, and memory management code execute within the same privileged environment.

There is not yet a complete separation between kernel space and user space.

---

# 4. Kernel Architecture

AngelOS uses a modular monolithic kernel architecture.

A monolithic kernel provides core operating system services within a single privileged execution environment.

This means that the filesystem, drivers, interrupt handling, memory management, and shell are integrated into the kernel rather than running as separate user-space services.

## 4.1 Major subsystems

The kernel is divided conceptually into the following components.

| Subsystem | Responsibility |
|---|---|
| Kernel initialization | Starts and coordinates the system |
| PMM | Manages physical pages |
| VMM | Handles virtual memory integration |
| Heap | Provides dynamic memory allocation |
| IDT | Defines exception and interrupt handlers |
| PIC | Manages legacy hardware IRQs |
| PIT | Provides timer interrupts |
| Keyboard | Processes keyboard input |
| Serial | Provides console communication |
| RTC | Provides hardware date and time |
| ATA | Provides disk sector operations |
| ACPI | Provides power-management operations |
| Filesystem | Manages files and directories |
| Shell | Provides the interactive command interface |

## 4.2 Subsystem interaction

The subsystems interact through function interfaces.

For example, a shell command that reads a file follows this general sequence:

```text
Keyboard
   |
   v
Keyboard Driver
   |
   v
Shell Input Buffer
   |
   v
Command Parser
   |
   v
Command Dispatcher
   |
   v
Filesystem Read Function
   |
   v
Filesystem Node
   |
   v
Serial Output
```

A file write follows a different path:

```text
Shell Command
     |
     v
Argument Parser
     |
     v
Filesystem Write
     |
     v
Node Contents Updated
     |
     v
Filesystem Save
     |
     v
ATA Driver
     |
     v
Disk Sectors
```

This separation allows the shell to perform filesystem operations without directly manipulating disk sectors.

## 4.3 Kernel responsibilities

The kernel is responsible for:

- Establishing the execution environment.
- Validating bootloader information.
- Initializing memory management.
- Installing interrupt handlers.
- Initializing hardware drivers.
- Detecting storage devices.
- Initializing the filesystem.
- Providing system-level operations.
- Launching the interactive shell.

## 4.4 Error handling

The kernel reports initialization errors through serial output.

Critical initialization failures generally cause the kernel to halt.

This prevents later subsystems from operating with invalid dependencies.

For example, continuing after physical memory initialization fails could result in memory corruption or unpredictable behavior.

---

# 5. Bootloader Integration

AngelOS uses Limine as its bootloader.

Limine is responsible for loading the kernel and providing the information needed to establish the initial kernel environment.

The bootloader handles the initial boot process, allowing AngelOS to concentrate on operating system development.

## 5.1 Limine requests

The kernel uses Limine requests to retrieve boot information.

Important responses include:

- Memory map response.
- Kernel address response.
- Higher Half Direct Map response.

These responses are used during kernel initialization.

## 5.2 Memory map

The memory map describes the physical memory regions available to the operating system.

Different memory regions have different purposes and availability.

The PMM uses this information to determine which pages can be managed and allocated.

The kernel must not treat every physical memory region as available RAM.

Some memory is reserved for firmware, the kernel, bootloader information, and other system components.

## 5.3 Kernel address information

AngelOS retrieves:

```c
kernel_address_request.response->physical_base;
kernel_address_request.response->virtual_base;
```

These values describe the physical and virtual bases associated with the loaded kernel image.

The kernel uses them to translate addresses within its linked image.

## 5.4 Higher Half Direct Map

The HHDM response provides the offset used to access mapped physical memory through the higher-half direct map.

AngelOS stores this information during VMM initialization.

## 5.5 Boot validation

Before continuing, AngelOS checks that the required Limine responses are available.

If a required response is missing, the kernel prints an error and halts.

This avoids attempting memory initialization with incomplete boot information.

---

# 6. Kernel Entry and Initialization

The kernel entry point is responsible for transferring execution into the main kernel initialization routine.

AngelOS uses NASM assembly for its entry code and C for most kernel functionality.

## 6.1 Entry point

The assembly entry code establishes the initial conditions needed by the kernel and transfers control to the C initialization function.

The kernel's linker script defines its memory layout and entry point.

The kernel is linked at its higher-half virtual address.

## 6.2 Main initialization routine

The main initialization routine coordinates the startup sequence.

The initialization process can be divided into several stages.

### Stage 1: Basic kernel output

Serial output is initialized.

The kernel displays startup information and validates the Limine boot protocol.

### Stage 2: Memory discovery

The kernel checks the memory map and obtains physical and virtual kernel addresses.

It calculates the physical address range corresponding to the kernel image.

The calculation uses the relationship:

```c
physical_address = physical_base +
                   (virtual_address - virtual_base);
```

The kernel checks for invalid ranges and arithmetic overflow before proceeding.

### Stage 3: Physical memory

The PMM is initialized using the memory map and calculated physical kernel boundaries.

The kernel then performs basic allocation and release tests.

### Stage 4: Virtual memory

The VMM is initialized using the HHDM response.

The kernel reports the HHDM offset and active PML4 address.

### Stage 5: Heap

The heap allocator is initialized.

Allocation, writing, resizing, and freeing tests are performed.

### Stage 6: Filesystem and storage

The filesystem is initialized.

The ATA driver probes the available storage devices.

### Stage 7: Hardware clock

The RTC is initialized.

If it is unavailable, the kernel reports a warning.

### Stage 8: Interrupt infrastructure

The PIC is initialized.

The PIT and keyboard drivers are configured.

### Stage 9: Shell

The kernel enables interrupts and starts the interactive shell.

The shell becomes the primary interface through which the user interacts with AngelOS.

---

# 7. Physical Memory Manager

The Physical Memory Manager, or PMM, is responsible for managing physical memory.

Physical memory consists of the actual RAM pages available to the processor.

The PMM provides the allocation mechanisms needed by the rest of the kernel.

## 7.1 Page-based memory management

AngelOS uses 4096-byte physical pages.

```c
#define PMM_PAGE_SIZE 4096
```

Memory is managed in units of these pages.

For example, allocating one page requests 4096 bytes of physical memory.

Allocating multiple pages requires tracking multiple physical page addresses.

## 7.2 Initialization

The PMM receives information from Limine.

Its initialization process must distinguish between usable physical memory and reserved memory.

The kernel image occupies a physical memory range that must remain reserved.

The PMM uses the calculated physical kernel start and end addresses to avoid treating kernel memory as freely allocatable memory.

Conceptually, initialization requires:

1. Reading memory map entries.
2. Identifying usable regions.
3. Excluding reserved regions.
4. Accounting for the kernel image.
5. Preparing page allocation metadata.
6. Making eligible pages available for allocation.

The exact allocator metadata representation is an implementation detail of the current PMM.

## 7.3 Page allocation

The PMM exposes physical page allocation.

A typical use is:

```c
uint64_t page = pmm_alloc_page();
```

The returned value represents the allocated physical page address.

A zero result indicates allocation failure in the current interface.

The caller must check the result before using the allocation.

## 7.4 Page release

Previously allocated pages can be released:

```c
pmm_free_page(page);
```

Releasing a page allows it to become available for future allocations.

The PMM must distinguish allocated pages from reserved pages.

Releasing a reserved kernel page must fail.

## 7.5 Managed and reserved pages

AngelOS distinguishes between pages that are managed by the PMM and pages that are reserved.

The PMM provides checks such as:

```c
pmm_is_reserved_page(address);
pmm_is_managed_page(address);
```

These checks are useful for validation and debugging.

## 7.6 Memory statistics

The PMM exposes:

```c
pmm_get_total_pages();
pmm_get_used_pages();
pmm_get_free_pages();
pmm_get_allocation_count();
pmm_get_release_count();
```

These functions provide information about physical memory usage.

The kernel displays these values during startup.

## 7.7 Initialization tests

AngelOS performs a basic physical allocation test.

The kernel:

1. Requests a physical page.
2. Checks whether allocation succeeded.
3. Releases the page.
4. Verifies that a kernel page remains reserved.
5. Attempts to release a reserved page.
6. Checks that the PMM rejects the invalid release.

These tests help detect errors in the PMM before the rest of the kernel starts.

## 7.8 Importance to other subsystems

The PMM provides the low-level memory allocation foundation for:

- Page-table management.
- Kernel heap expansion.
- Future process address spaces.
- Kernel memory structures.
- Device-related memory allocations.

## 7.9 Current limitations

The PMM currently provides basic physical page management.

More advanced memory management features, such as NUMA awareness, memory compaction, and sophisticated allocation policies, are not implemented.

---

# 8. Virtual Memory Manager

The Virtual Memory Manager, or VMM, handles virtual memory integration.

Unlike physical memory, virtual memory provides the addresses that software uses when accessing memory.

On x86-64, address translation is performed through hierarchical page tables.

## 8.1 x86-64 paging

The x86-64 architecture uses a multi-level page-table structure.

A conventional four-level paging configuration includes:

- PML4.
- Page Directory Pointer Table.
- Page Directory.
- Page Table.

The processor uses these structures to translate virtual addresses into physical addresses.

## 8.2 PML4

The PML4 is the top-level structure in the four-level page-table hierarchy.

AngelOS reports the active PML4 physical address through its VMM interface.

This information is useful for understanding the current paging environment.

## 8.3 Higher Half Direct Map

AngelOS uses the Higher Half Direct Map provided by Limine.

The HHDM makes physical memory accessible through a virtual address offset.

Conceptually:

```text
Virtual address = Physical address + HHDM offset
```

The VMM retrieves and stores this offset.

It exposes the offset through:

```c
vmm_get_hhdm_offset();
```

## 8.4 Initialization

The VMM is initialized after the PMM.

It receives the Limine HHDM response and establishes the information needed by the kernel to work with the existing virtual memory environment.

## 8.5 Kernel address translation

The kernel uses the physical and virtual base information from Limine to calculate physical addresses corresponding to its linked virtual address range.

This is particularly important when identifying which physical pages belong to the kernel image.

## 8.6 Relationship with the PMM

The VMM and PMM have different responsibilities.

The PMM manages physical pages.

The VMM manages virtual address mappings and the structures used by the processor for address translation.

A future implementation of independent address spaces will require both subsystems to work together.

## 8.7 Future development

Potential improvements include:

- Creating new address spaces.
- Mapping and unmapping pages.
- User-space memory mappings.
- Page-level permissions.
- Process isolation.
- Guard pages.
- Demand paging.
- Copy-on-write support.

These are future capabilities, not claims about the current implementation.

---

# 9. Kernel Heap

The kernel heap provides dynamic memory allocation for the kernel.

Physical page allocation is useful for obtaining large units of memory, but kernel subsystems often require smaller allocations.

The heap provides a higher-level interface.

## 9.1 Initialization

The heap is initialized after virtual memory management.

AngelOS checks whether heap initialization succeeds before proceeding.

If initialization fails, the kernel halts.

## 9.2 Allocation interface

The kernel provides four main allocation functions.

### `kmalloc()`

```c
void *pointer = kmalloc(128);
```

Requests a block of memory of the specified size.

### `kcalloc()`

```c
uint32_t *values = kcalloc(16, sizeof(uint32_t));
```

Allocates memory for multiple elements and initializes the allocated memory to zero.

### `krealloc()`

```c
void *resized = krealloc(pointer, 256);
```

Requests a different allocation size while preserving existing data where supported by the allocator.

### `kfree()`

```c
kfree(pointer);
```

Releases a heap allocation.

## 9.3 Heap testing

AngelOS performs several tests during startup.

### Multiple allocations

The kernel allocates separate memory blocks of different sizes.

### Memory writing

The kernel writes test values into the allocated memory.

### Memory verification

The kernel reads the values back to verify that the memory behaves as expected.

### Zero allocation

The kernel uses `kcalloc()` and checks that the allocated elements are initialized to zero.

### Resizing

The kernel uses `krealloc()` to resize an existing allocation.

It verifies that the original data remains intact.

### Freeing

The kernel releases the allocated blocks using `kfree()`.

## 9.4 Heap statistics

The heap exposes:

```c
heap_get_page_count();
heap_get_free_block_count();
```

These functions provide information about heap usage.

## 9.5 Relationship with the filesystem

The heap provides the dynamic memory infrastructure that can be used by future filesystem implementations.

For example, a dynamically allocated file buffer could eventually allow files to grow beyond the current fixed-size content limit.

---

# 10. Interrupt Descriptor Table

The Interrupt Descriptor Table is one of the most important processor structures used by AngelOS.

It defines the handlers that the processor invokes when exceptions or interrupts occur.

## 10.1 Purpose

The CPU must have a defined response to exceptional conditions.

Examples include:

- Division by zero.
- Invalid instructions.
- Protection violations.
- Page faults.
- Double faults.

Hardware devices can also request CPU attention through interrupts.

The IDT provides the mechanism for associating interrupt vectors with handler functions.

## 10.2 Exception handlers

AngelOS installs handlers for 32 CPU exception vectors.

These correspond to the processor's architectural exception range.

Exception handlers allow the kernel to detect and report CPU faults.

## 10.3 Hardware interrupt handlers

Hardware interrupt handlers are used for devices such as the keyboard and timer.

The kernel registers and installs the appropriate interrupt handling functionality.

## 10.4 Interrupt frames

Interrupt handlers receive an interrupt frame.

The frame provides information associated with the interrupted execution context.

The exact contents depend on the assembly entry code and the processor's interrupt behavior.

## 10.5 Keyboard interrupt flow

The keyboard interrupt handler retrieves the incoming scancode:

```c
static void keyboard_interrupt_handler(
    struct interrupt_frame *frame)
{
    (void)frame;
    keyboard_process_scancode(
        inb(KEYBOARD_DATA_PORT)
    );
}
```

The keyboard driver processes the scancode.

The shell later retrieves translated key values from the keyboard input interface.

## 10.6 Interrupt safety

Interrupt handlers should remain short and predictable.

Complex operations are better performed outside interrupt context.

For example, the keyboard interrupt handler does not execute shell commands or write files. It only forwards the received scancode to the keyboard subsystem.

## 10.7 Current scope

AngelOS currently provides exception and hardware interrupt infrastructure.

A complete advanced interrupt framework, including sophisticated interrupt prioritization and interrupt-safe synchronization primitives, remains future work.

---

# 11. Programmable Interrupt Controller

AngelOS uses the legacy Programmable Interrupt Controller to manage hardware IRQs.

The PIC receives interrupt requests from hardware devices and signals the processor.

## 11.1 Remapping

Historically, the PIC's default interrupt vectors overlap with CPU exception vectors.

To avoid this conflict, AngelOS remaps the PIC.

Current configuration:

```text
Master PIC base: 32
Slave PIC base:  40
```

This assigns the master PIC's IRQ range to vectors 32–39 and the slave PIC's IRQ range to vectors 40–47.

## 11.2 IRQ lines

The legacy PIC supports 16 IRQ lines.

These are divided between the master and slave controllers.

The slave PIC is connected through the master PIC's cascade mechanism.

## 11.3 Initialization

AngelOS initializes the PIC before enabling hardware interrupts.

The kernel then selectively unmasks the required IRQ lines.

Currently, the timer and keyboard interrupts are enabled.

```c
pic_unmask_irq(0);
pic_unmask_irq(1);
```

IRQ0 is used by the PIT.

IRQ1 is used by the keyboard.

## 11.4 Interrupt acknowledgement

Hardware interrupt handling must acknowledge completed interrupts to the PIC through end-of-interrupt signaling.

The interrupt framework is responsible for handling this as appropriate to the interrupt path.

## 11.5 Future development

Possible improvements include:

- More flexible IRQ registration.
- Improved interrupt masking.
- Additional hardware IRQ support.
- APIC support.
- I/O APIC support.
- MSI and MSI-X support.

---

# 12. Programmable Interval Timer

The Programmable Interval Timer provides periodic hardware interrupts.

AngelOS uses the PIT as its initial system timer.

## 12.1 Configuration

The kernel configures the PIT at:

```text
100 Hz
```

This means the timer generates approximately 100 ticks per second.

The duration of one tick is therefore:

```text
1 / 100 = 0.01 seconds
```

Or 10 milliseconds.

## 12.2 Initialization

The timer is initialized through:

```c
pit_init(100);
```

The kernel checks the result and halts if initialization fails.

## 12.3 IRQ0

The PIT uses IRQ0 on the legacy PIC.

AngelOS unmasks IRQ0 after initializing the timer.

## 12.4 Timer ticks

Each timer interrupt represents another timer event.

A software tick counter can be used to track elapsed time.

At 100 Hz:

```text
100 ticks = approximately 1 second
```

## 12.5 Uptime

The timer provides the basis for uptime tracking.

The shell exposes an uptime command that reports how long the system has been running.

## 12.6 Future applications

A periodic timer is an important foundation for:

- Task scheduling.
- Sleep operations.
- Timeouts.
- Delayed execution.
- Time slicing.
- Performance measurements.

A complete preemptive scheduler is not yet implemented.

---

# 13. Keyboard Driver

AngelOS uses a PS/2-compatible keyboard driver to receive user input.

The driver communicates with the keyboard controller and translates scancodes into usable key values.

## 13.1 Keyboard controller

The keyboard driver communicates with the PS/2 controller through its I/O ports.

The primary data port is:

```text
0x60
```

The status and command port is:

```text
0x64
```

The keyboard controller reports incoming key information through the data port.

## 13.2 Scancodes

When a key is pressed or released, the keyboard produces a scancode.

The driver translates the scancode into an appropriate key value.

AngelOS uses a Set 1-style keymap for character translation.

## 13.3 Character translation

The driver contains normal and shifted keymaps.

The appropriate map is selected based on keyboard state.

For example, pressing a letter without Shift produces its normal character, while pressing it with Shift produces the corresponding uppercase character.

The driver also tracks modifier states such as:

- Shift.
- Caps Lock.
- Ctrl.
- Alt.

## 13.4 Special keys

The driver supports special keys and extended scancode sequences.

Examples include:

- Arrow keys.
- Home.
- End.
- Delete.
- Backspace.
- Enter.
- Tab.

These keys are represented by special key values rather than ordinary printable characters.

## 13.5 Interrupt handling

The keyboard receives input through IRQ1.

The registered interrupt handler reads the data port and passes the scancode into the keyboard processing function.

## 13.6 Input buffering

The keyboard driver provides a buffer that allows the shell to retrieve processed input.

This separates hardware interrupt handling from shell processing.

The interrupt handler can process the incoming scancode while the shell retrieves the resulting key later.

## 13.7 Shell integration

The shell obtains keyboard input using:

```c
int key = keyboard_getkey();
```

If no key is available, the shell waits for an interrupt.

The input loop uses interrupt control and the processor's halt instruction while waiting.

## 13.8 Current limitations

The current keyboard implementation is focused on PS/2-compatible input.

USB keyboards, international layouts, advanced input methods, and graphical input are not currently supported as complete subsystems.

---

# 14. Serial Driver

Serial output provides the primary text communication mechanism used by AngelOS.

It is especially useful during kernel development because it can operate before a graphical display system exists.

## 14.1 COM1

AngelOS uses the conventional COM1 serial interface.

Its base I/O port is:

```text
0x3F8
```

## 14.2 Initialization

The serial driver initializes the communication interface before the kernel begins printing boot messages.

This allows early initialization failures to be reported.

## 14.3 Output

The kernel uses serial output to display:

- Boot information.
- Memory statistics.
- Hardware detection.
- Filesystem messages.
- Errors.
- Shell prompts.
- Command output.

## 14.4 Character output

The shell uses the serial output interface to display characters and text.

It also uses terminal escape sequences for operations such as cursor movement and line editing.

For example:

```c
serial_write("\033[1C");
```

This sends a terminal escape sequence requesting cursor movement to the right.

## 14.5 Development advantages

Serial output is useful because it allows the kernel to report errors even when the framebuffer or graphical environment is unavailable.

In QEMU, serial output can be redirected to the host terminal.

---

# 15. Real-Time Clock

The Real-Time Clock provides hardware date and time information.

AngelOS initializes the RTC during kernel startup.

## 15.1 Purpose

The RTC allows the operating system to obtain the current date and time independently of its uptime timer.

The PIT tracks elapsed time during operation.

The RTC provides the hardware clock information.

These two mechanisms have different responsibilities.

## 15.2 Initialization

AngelOS calls:

```c
rtc_init();
```

The kernel checks whether initialization succeeds.

If the RTC is unavailable, a warning is displayed.

## 15.3 Shell integration

The RTC is used by the shell's date and time commands.

```text
date
time
```

These commands provide access to the system's hardware clock information.

## 15.4 Future development

Potential improvements include:

- More robust clock synchronization.
- Timezone support.
- Calendar utilities.
- System clock management.
- Higher-resolution timekeeping.

---

# 16. ATA Storage Driver

The ATA driver provides sector-level storage access.

AngelOS currently uses ATA Programmed Input/Output.

PIO allows the processor to communicate with storage devices through I/O port operations.

## 16.1 ATA interface

The ATA controller exposes registers used for device selection, status checking, commands, and data transfer.

The driver communicates with these registers to perform disk operations.

## 16.2 Device detection

During startup, AngelOS probes the available legacy IDE channels.

The initialization function returns the number of detected supported devices:

```c
uint32_t ata_devices = ata_init();
```

The kernel reports the result.

## 16.3 Sector addressing

ATA storage operations use Logical Block Addressing.

Each sector is identified by an LBA number.

For example:

```text
LBA 0
LBA 1
LBA 2
LBA 3
```

This provides a linear way to address disk sectors.

## 16.4 Reading sectors

The driver provides:

```c
ata_read_sector();
```

This reads a sector from a specified disk location into a memory buffer.

The filesystem uses sector reads when loading stored filesystem information.

## 16.5 Writing sectors

The driver provides:

```c
ata_write_sector();
```

This writes a memory buffer to a specified disk sector.

The filesystem uses this functionality when persisting changes.

## 16.6 Device capacity

The driver also exposes:

```c
ata_get_sector_count();
```

This provides the detected device's sector capacity.

The filesystem can use this information to check whether its required disk region is available.

## 16.7 Availability

The driver exposes:

```c
ata_is_available();
```

This allows other components to determine whether a supported ATA device is available.

## 16.8 Filesystem integration

The filesystem is built on top of the ATA interface.

The filesystem manages logical objects such as files and directories.

The ATA driver manages the actual sector transfers.

This separation means filesystem code does not need to implement ATA register communication itself.

## 16.9 Current limitations

The current implementation uses PIO rather than DMA.

More advanced storage interfaces, including AHCI, NVMe, and modern PCI storage controllers, are future development areas.

---

# 17. ACPI Power Management

AngelOS includes ACPI-based system control functionality.

ACPI is used to provide standardized power-management mechanisms.

## 17.1 Shutdown

The shell's shutdown command invokes the ACPI shutdown functionality.

Conceptually:

```c
acpi_shutdown();
```

The kernel disables normal interrupt processing before attempting the shutdown operation.

If ACPI shutdown fails, the kernel prints an error and halts safely.

## 17.2 Reboot

AngelOS also includes a reboot command.

The reboot functionality uses the kernel's system control implementation.

## 17.3 Error handling

Power-management operations are hardware- and firmware-dependent.

The kernel must handle unsuccessful shutdown requests instead of assuming that the machine has powered off.

## 17.4 Future development

Further ACPI development could include:

- Improved firmware table parsing.
- Power-state management.
- ACPI device discovery.
- Battery information.
- More complete platform support.

---

# 18. HaloRoot Filesystem

HaloRoot is the custom filesystem used by AngelOS.

It provides file and directory organization without depending on an existing general-purpose filesystem implementation.

The current implementation uses a fixed-size node table and ATA-backed persistence.

## 18.1 Filesystem model

The filesystem represents files and directories as nodes.

Each node contains metadata describing the object.

The node structure includes information corresponding to:

- Name.
- Parent node.
- Used state.
- Directory state.
- File contents.

The parent relationship is used to construct directory hierarchies.

## 18.2 Node capacity

The current implementation defines:

```c
#define FS_MAX_NODES 128
```

This means the filesystem has a fixed capacity of 128 nodes.

Each file and directory occupies one node.

The root directory is part of the filesystem's node representation.

## 18.3 Node allocation

When creating a file or directory, the filesystem must locate an unused node.

The node is then populated with the requested name, parent information, and object type.

The node becomes part of the active filesystem state.

## 18.4 Directory relationships

Directories are represented through parent relationships.

For example:

```text
/
|
+-- projects
    |
    +-- notes.txt
    |
    +-- development
        |
        +-- kernel.txt
```

The files and directories are associated with their respective parent nodes.

This allows the filesystem to distinguish between objects in different directories even when their names are identical.

## 18.5 File content

The current content limit is:

```c
#define FS_CONTENT_MAX 512
```

The contents are stored as a null-terminated string.

Therefore, the maximum usable content is 511 bytes.

The current implementation does not provide an arbitrary-length file allocation system.

## 18.6 Filesystem API

The filesystem exposes functions for use by the kernel and shell.

The primary interfaces include:

```c
int fs_init(void);

int fs_create_dir(const char *name);
int fs_create_file(const char *name);
int fs_remove(const char *name);

int fs_change_dir(const char *path);

void fs_list(fs_output_fn output);
void fs_pwd(fs_output_fn output);

int fs_write(const char *name, const char *contents);
const char *fs_read(const char *name);

int fs_getcwd(char *buffer, uint32_t size);
```

### `fs_init()`

Initializes the filesystem.

It prepares the filesystem's internal structures and handles the existing filesystem storage format.

### `fs_create_dir()`

Creates a directory.

The supplied path is resolved relative to the current directory or from the root when an absolute path is used.

### `fs_create_file()`

Creates a regular file.

### `fs_remove()`

Removes a filesystem object according to the current removal rules.

### `fs_change_dir()`

Changes the current working directory.

### `fs_list()`

Enumerates the contents of the current directory.

The caller supplies an output callback.

### `fs_pwd()`

Reports the current directory through an output callback.

### `fs_write()`

Writes contents into an existing file.

### `fs_read()`

Returns a pointer to the contents of a file.

A missing file is represented by a null result.

### `fs_getcwd()`

Copies the current working directory path into a supplied buffer.

## 18.7 Output callbacks

The filesystem uses an output callback type:

```c
typedef void (*fs_output_fn)(const char *text);
```

This allows filesystem operations to report text without being directly tied to a particular output device.

The shell supplies an output function that writes through the serial console.

This design separates filesystem logic from terminal output.

---

# 19. Filesystem Disk Format

The HaloRoot filesystem uses a custom on-disk representation.

The current disk format is identified by a magic value and version number.

## 19.1 Format constants

```c
#define FS_MAX_NODES 128

#define FS_DISK_MAGIC "HALORFS1"
#define FS_DISK_VERSION 1

#define FS_DISK_SECTORS (1 + FS_MAX_NODES * 2)

#define FS_HEADER_LBA 0
```

## 19.2 Header

The filesystem header is located at:

```text
LBA 0
```

The header contains information used to identify and validate the filesystem.

The magic identifier is:

```text
HALORFS1
```

The version is:

```text
1
```

The filesystem checks this information when interpreting the stored disk format.

## 19.3 Node storage

Each node occupies two sectors.

The first node starts at:

```text
LBA 1
```

The address of node `i` is calculated using:

```c
1 + i * 2
```

For example:

| Node index | Starting LBA |
|---|---:|
| 0 | 1 |
| 1 | 3 |
| 2 | 5 |
| 3 | 7 |
| 4 | 9 |

The second sector of each node follows its first sector.

## 19.4 Total reserved sectors

The filesystem reserves:

```text
1 + (128 * 2)
```

Which equals:

```text
257 sectors
```

The total byte requirement depends on the sector size supported by the disk interface.

For a conventional 512-byte sector:

```text
257 * 512 = 131584 bytes
```

This is approximately 128.5 KiB.

## 19.5 Serialization

Filesystem structures exist as C data structures while the kernel is running.

Before being stored on disk, their relevant fields must be represented in the filesystem's disk format.

This is known as serialization.

The reverse operation reconstructs the in-memory filesystem state from the stored representation.

The current filesystem uses fixed-size structures, making its disk layout relatively straightforward.

## 19.6 Versioning

The version field provides a way to distinguish between filesystem formats.

If the filesystem format changes in the future, a new version can be introduced.

This is important because changing node layouts without changing the version could cause old disk contents to be interpreted incorrectly.

## 19.7 Compatibility

Changes to the following areas may require a filesystem format migration:

- Node structure.
- Name storage.
- Content storage.
- Parent representation.
- Metadata fields.
- Sector allocation.
- Header layout.

Existing disk images should not be casually reformatted when changing these structures.

---

# 20. Filesystem Path Resolution

Path resolution is responsible for translating a textual path into a filesystem node.

This allows users and shell commands to refer to objects using directory paths rather than internal node indexes.

## 20.1 Absolute paths

An absolute path begins at the root directory.

Example:

```text
/projects/notes.txt
```

The filesystem begins resolution at the root and follows each path component.

## 20.2 Relative paths

A relative path begins from the current working directory.

Example:

```text
notes.txt
```

If the current directory is `/projects`, the path refers to:

```text
/projects/notes.txt
```

## 20.3 Current directory

The filesystem maintains a current directory node.

Relative paths use this node as their starting point.

## 20.4 Parent directory

The special component:

```text
..
```

refers to the parent directory.

For example:

```text
/projects/development
```

After resolving:

```text
..
```

the resulting directory is:

```text
/projects
```

## 20.5 Current directory component

The special component:

```text
.
```

refers to the current directory.

This means paths such as:

```text
./notes.txt
```

can be resolved correctly.

## 20.6 Nested paths

The filesystem supports nested directory resolution.

For example:

```text
/projects/development/kernel.txt
```

requires resolving:

1. Root directory.
2. `projects`.
3. `development`.
4. `kernel.txt`.

Each component must be resolved within the directory represented by the previous component.

## 20.7 Repeated separators

The path resolver also handles repeated slash characters.

For example, paths containing repeated separators can be normalized during resolution.

## 20.8 Path creation

File and directory creation can accept nested paths.

The filesystem resolves the parent path and then creates the final node within that directory.

The parent directory must already exist.

## 20.9 Working directory output

The filesystem can construct the current path using:

```c
fs_getcwd(buffer, size);
```

The implementation walks the directory hierarchy and builds the path in the supplied buffer.

The buffer size must be sufficient for the resulting path.

---

# 21. Filesystem Operations

This section describes how the filesystem performs its main operations.

## 21.1 Directory creation

When creating a directory:

1. The requested path is examined.
2. The parent directory is resolved.
3. An available node is located.
4. The directory name is stored.
5. Its parent relationship is established.
6. Its directory flag is set.
7. The node is marked as used.
8. Filesystem changes are persisted.

The operation fails if the required parent cannot be resolved or no node is available.

## 21.2 File creation

File creation follows a similar process.

The filesystem resolves the parent directory, allocates a node, stores the name, and marks the node as a regular file.

The new file can subsequently be written to and read from.

## 21.3 Reading

When a file is read:

1. Its path is resolved.
2. The resulting node is checked.
3. The node must represent a file.
4. Its content is returned to the caller.

The current API returns a pointer to the stored contents.

The caller must not treat a missing file as a valid string.

## 21.4 Writing

Writing updates the content associated with a file node.

The current content buffer has a fixed maximum size.

The filesystem must ensure that contents fit within the supported buffer.

The resulting filesystem state is persisted through the disk backend.

## 21.5 Removal

The removal operation resolves the requested path and checks the target node.

The filesystem then updates its node state according to its removal rules.

A future filesystem version could introduce more advanced recursive deletion and recovery mechanisms.

## 21.6 Directory listing

The listing operation examines the nodes associated with the current directory.

It reports their names and object types through the supplied output callback.

The shell is responsible for displaying the resulting information.

## 21.7 Current working directory

The filesystem maintains a current directory.

Commands that use relative paths depend on this state.

The `pwd` command uses the filesystem's path reconstruction functionality to display it.

---

# 22. Filesystem Persistence

Persistence is the process of retaining filesystem data after the kernel stops running.

AngelOS uses ATA sector operations to store filesystem information on disk.

## 22.1 In-memory state

While running, the filesystem maintains its node information in memory.

Operations such as file creation and writing modify this state.

## 22.2 Disk state

The filesystem also maintains a serialized representation on the ATA-backed storage device.

The header and node records occupy predefined sectors.

## 22.3 Saving changes

When filesystem data changes, the filesystem saves the updated representation through the ATA driver.

The ATA driver performs the actual sector writes.

## 22.4 Loading data

During filesystem initialization, the implementation uses the stored filesystem information to reconstruct the filesystem state.

The magic identifier and version help identify a valid filesystem.

## 22.5 Persistence testing

A practical persistence test is:

1. Boot AngelOS.
2. Create a directory.
3. Create a file.
4. Write some content.
5. Save the file.
6. Reboot the system.
7. Reattach the same disk image.
8. List the directory.
9. Read the file.

The file and directory should remain available if the filesystem data was successfully persisted.

## 22.6 Storage safety

The filesystem uses a defined region of the disk.

Changes to its layout must be handled carefully.

Reformatting or replacing the disk image can destroy existing files.

---

# 23. AngelOS Shell Architecture

The AngelOS shell provides an interactive interface to kernel functionality.

It currently runs inside the kernel rather than as an independent user-space program.

## 23.1 Shell responsibilities

The shell is responsible for:

- Receiving keyboard input.
- Maintaining the current input line.
- Handling cursor movement.
- Processing command history.
- Parsing arguments.
- Expanding aliases.
- Dispatching commands.
- Displaying output.
- Returning to the prompt.

## 23.2 Input buffer

The shell stores the current command in an input buffer.

The buffer has a fixed maximum size defined by the shell configuration.

The shell tracks:

- Current input length.
- Cursor position.
- Command history position.
- Current input contents.

## 23.3 Command execution

When Enter is pressed, the shell:

1. Prints a newline.
2. Terminates the input string.
3. Saves the command to history.
4. Resets history navigation state.
5. Executes the command.
6. Clears the current input buffer.
7. Prints the shell prompt again.

## 23.4 Argument representation

Commands are represented as an argument count and an argument array.

For example:

```text
write notes.txt Hello World
```

is conceptually parsed into:

```text
argc = 4

argv[0] = "write"
argv[1] = "notes.txt"
argv[2] = "Hello"
argv[3] = "World"
```

The dispatcher uses these arguments to determine which operation should be executed.

## 23.5 Command dispatch

The shell compares the first argument against known command names.

Each recognized command invokes its corresponding handler or subsystem function.

Unknown commands produce an error message.

## 23.6 Shell prompt

The current prompt is:

```text
angel>
```

The prompt is printed after command execution and during initial shell startup.

---

# 24. Command Parsing and Dispatch

The shell separates command parsing from command execution.

This makes the command system easier to expand.

## 24.1 Argument parsing

The parser receives a command string and produces an argument array.

Its interface is conceptually:

```c
int shell_parse_arguments(
    char *command,
    char *argv[],
    int max_arguments
);
```

The returned value is the number of parsed arguments.

## 24.2 Empty commands

If no arguments are found, command execution returns without dispatching anything.

This prevents empty input from being interpreted as a command.

## 24.3 Alias processing

Before normal command dispatch, the shell checks whether the command name matches an alias.

If an alias exists:

1. Its stored command is copied into an expansion buffer.
2. The expansion is parsed.
3. Original additional arguments are appended.
4. The expanded command is dispatched.

## 24.4 Dispatching

The dispatcher receives:

```c
shell_dispatch(argc, argv);
```

It checks the command name and invokes the appropriate implementation.

This keeps command recognition in one central location.

## 24.5 Error messages

If no command matches, the shell displays:

```text
Unknown command: command_name
Type 'help' to see available commands.
```

## 24.6 Current parser limitations

The current shell uses a simple argument parser.

Advanced shell features such as pipelines, shell quoting rules, variable expansion, command substitution, and redirection are not yet implemented as a complete system.

---

# 25. Shell Input System

The shell input loop continuously retrieves processed keyboard input.

It is responsible for turning individual key events into a complete command line.

## 25.1 Waiting for input

The shell retrieves keys using:

```c
int key = keyboard_getkey();
```

If no key is available, the shell disables interrupts briefly and waits using the processor's halt instruction.

The general sequence is:

```text
Disable interrupts
       |
       v
Check keyboard buffer
       |
       +---- Key available ----> Process key
       |
       +---- No key -----------> Enable interrupts and halt
```

This avoids continuously polling the keyboard without sleeping.

## 25.2 Enter

When Enter is detected:

- The command line is terminated.
- The command is stored in history.
- The command is executed.
- The input state is reset.
- The prompt is displayed again.

## 25.3 Backspace

Backspace removes the character before the cursor.

The shell updates the input buffer and terminal display.

## 25.4 Tab

Tab invokes the shell's completion functionality.

## 25.5 Arrow keys

The Up and Down arrows navigate command history.

The Left and Right arrows move the cursor.

## 25.6 Home and End

Home moves the cursor to the beginning of the input line.

End moves it to the end.

## 25.7 Delete

Delete removes the character at the current cursor position.

## 25.8 Printable characters

Printable ASCII characters are inserted into the input buffer.

Characters outside the supported printable range are ignored by the normal shell input path.

## 25.9 Terminal rendering

The shell uses serial output to update the displayed input line.

It sends terminal control sequences for cursor movement and uses backspace operations to remove visible characters.

---

# 26. Command History

Command history allows previously entered commands to be accessed without typing them again.

## 26.1 History storage

The shell maintains a history buffer with a configured maximum number of entries.

Each executed command can be stored in this buffer.

## 26.2 Previous command

Pressing Up retrieves the previous history entry.

The shell updates the input buffer and redraws the command line.

## 26.3 Next command

Pressing Down moves forward through history.

The shell restores the appropriate command or the current draft input.

## 26.4 Draft input

The shell maintains a draft buffer so that partially typed input can be restored when navigating through history.

## 26.5 History position

The shell tracks the current history position.

After normal input changes or command execution, history navigation is reset to the current end of the history list.

## 26.6 Future development

Possible improvements include:

- Persistent command history.
- Searchable history.
- History expansion.
- Configurable history capacity.
- Duplicate-command filtering.

---

# 27. Command Aliases

Aliases provide shortcuts for frequently used commands.

## 27.1 Alias storage

AngelOS maintains an alias table containing alias names and their corresponding command strings.

## 27.2 Creation

The command:

```text
alias ll=list
```

creates an alias named `ll`.

The alias expands into:

```text
list
```

## 27.3 Listing

Running:

```text
alias
```

without additional arguments displays the current aliases.

## 27.4 Removal

The command:

```text
unalias ll
```

removes the specified alias.

## 27.5 Argument forwarding

Alias expansion supports additional arguments.

For example:

```text
alias show=src
show notes.txt
```

expands to the equivalent of:

```text
src notes.txt
```

## 27.6 Current limitations

Aliases are currently maintained by the shell.

They are not necessarily persistent across reboot.

A future configuration system could allow aliases to be loaded automatically during startup.

---

# 28. Tab Completion

Tab completion is integrated into the shell input loop.

When Tab is pressed, the shell calls its completion function.

The completion function receives the current input buffer, its length, and the cursor position.

This allows it to inspect and update the current command line.

The completion implementation can be expanded to support:

- Command names.
- Filesystem paths.
- File names.
- Directory names.
- Alias names.
- Context-aware argument completion.

The current completion behavior is limited to the implemented shell completion logic.

---

# 29. Built-in Text Editor

AngelOS includes a simple built-in text editor.

The editor is designed to provide basic file editing directly from the shell.

It uses the existing filesystem API, allowing edited documents to be saved using the same storage system as ordinary files.

## 29.1 Opening a document

The editor is launched using:

```text
edit filename
```

For example:

```text
angel> edit notes.txt
```

The editor checks whether the requested file exists.

If the file exists, its contents are loaded.

If it does not exist, the editor attempts to create it.

## 29.2 Editor buffer

The editor maintains a fixed-size document buffer.

The buffer size is based on:

```c
FS_CONTENT_MAX
```

Currently:

```c
#define FS_CONTENT_MAX 512
```

The editor reserves one byte for the null terminator.

The maximum usable content is therefore 511 bytes.

## 29.3 Existing document loading

The editor uses:

```c
fs_read(filename);
```

to retrieve existing contents.

It copies the returned contents into its editing buffer.

The buffer is terminated with a null character.

## 29.4 New documents

If the file does not exist, the editor calls:

```c
fs_create_file(filename);
```

It then prepares an empty document buffer.

## 29.5 Character input

Printable characters are appended to the document buffer.

The editor updates its buffer length after each successful insertion.

It also sends the character to serial output.

## 29.6 Newlines

When Enter is pressed, a newline character is inserted into the buffer.

The terminal moves to a new line.

This allows the editor to store multiline text.

## 29.7 Backspace

Backspace removes the last character in the buffer.

The editor updates the buffer length and null terminator.

The current implementation is append-oriented rather than supporting arbitrary cursor positioning.

## 29.8 Saving

Ctrl+S is handled through the control character value:

```c
19
```

The editor terminates the buffer and calls:

```c
fs_write(filename, buffer);
```

A successful operation displays a save confirmation.

The filesystem handles the underlying persistence operation.

## 29.9 Exiting

Ctrl+Q is handled through the control character value:

```c
17
```

The editor exits and returns control to the shell.

The current implementation warns when modifications have not been saved.

Unsaved changes are discarded when leaving the editor.

## 29.10 Data flow

The editor uses the following general structure:

```text
Shell
  |
  v
edit filename
  |
  v
Editor Initialization
  |
  v
Filesystem Read
  |
  v
Document Buffer
  |
  v
Keyboard Input
  |
  +---- Printable character --> Append
  |
  +---- Enter ---------------> Newline
  |
  +---- Backspace -----------> Delete last character
  |
  +---- Ctrl+S --------------> Filesystem Write
  |
  +---- Ctrl+Q --------------> Return to shell
```

## 29.11 Current limitations

The editor is an initial implementation.

It currently lacks:

- Arbitrary cursor positioning.
- Inserting characters in the middle of a document.
- Text selection.
- Copy and paste.
- Search and replace.
- Syntax highlighting.
- Multiple open documents.
- Dynamic document buffers.
- Advanced terminal screen management.

These features can be added as the shell input system and filesystem become more advanced.

---

# 30. Disk Utilities

AngelOS provides shell commands for inspecting detected storage devices.

## 30.1 Disk listing

The `disk` command invokes the kernel's disk listing functionality.

It reports detected disk devices.

The ATA driver provides the underlying device information.

## 30.2 Sector reading

The `diskread` command allows a sector to be read using its device index and LBA.

Syntax:

```text
diskread <device-index> <LBA>
```

Example:

```text
diskread 0 0
```

The command parses the device index and LBA as unsigned integers.

It validates the supplied arguments before attempting the read.

## 30.3 Raw sector access

A raw sector read bypasses the filesystem's higher-level abstraction.

This is useful for examining filesystem metadata and debugging storage operations.

## 30.4 Safety

Reading sectors is generally non-destructive.

Writing raw sectors is more dangerous because incorrect writes can overwrite filesystem metadata or stored contents.

Disk inspection should therefore be performed carefully.

---

# 31. Date, Time, and Uptime

AngelOS provides access to hardware time and system elapsed time.

These are separate concepts.

## 31.1 Hardware date and time

The RTC provides the machine's hardware clock.

The shell exposes this information through:

```text
date
time
```

## 31.2 System uptime

Uptime represents the time elapsed since the operating system started.

The PIT provides periodic timer interrupts.

At 100 Hz, approximately 100 ticks represent one second.

The kernel can use these ticks to calculate elapsed time.

## 31.3 Difference between RTC and PIT

The RTC represents calendar time.

The PIT provides periodic events during system execution.

The two mechanisms are not interchangeable.

The RTC is used for date and time information, while the PIT provides the basis for uptime tracking and future scheduling.

---

# 32. System Reboot and Shutdown

AngelOS provides system-level power commands.

## 32.1 Reboot

The shell's reboot command invokes the kernel reboot functionality.

The reboot implementation uses the supported hardware control mechanism.

## 32.2 Shutdown

The shutdown command invokes ACPI shutdown functionality.

The kernel disables normal interrupt processing before attempting the shutdown operation.

If the operation fails, an error is reported and the kernel halts.

## 32.3 Error handling

Shutdown and reboot are hardware-dependent operations.

The kernel must not assume that a requested operation succeeded without confirming that the relevant mechanism completed.

---

# 33. Kernel Diagnostics

AngelOS uses serial output and internal tests to provide information about the kernel's state.

Diagnostics are particularly important in early operating system development because many failures occur before a graphical environment is available.

## 33.1 Initialization messages

Each major subsystem reports its initialization status.

Examples include:

```text
[PMM] Physical memory manager initialized.
[VMM] Virtual memory initialized.
[HEAP] Kernel heap initialized.
[RTC] Hardware clock initialized.
[PIT] Timer configured at 100 Hz.
```

## 33.2 Error reporting

Critical failures are reported with error messages.

For example:

```text
[ERROR] PMM initialization failed.
```

The kernel then halts if continuing would be unsafe.

## 33.3 Memory diagnostics

The kernel prints PMM and heap statistics.

This helps track allocation behavior during startup.

## 33.4 Hardware diagnostics

The kernel reports detected ATA devices and the initialization status of the keyboard and RTC.

## 33.5 Future debugging facilities

Potential improvements include:

- Kernel panic reporting.
- Stack traces.
- Symbol resolution.
- Debug logging levels.
- Memory corruption detection.
- Assertions.
- Exception register dumps.
- Structured kernel logs.

---

# 34. Building AngelOS

AngelOS is developed using a Linux-based toolchain.

The build process compiles the kernel sources, assembles the entry code, links the kernel, and creates a bootable ISO.

## 34.1 Requirements

The development environment requires:

- GCC or a suitable x86-64 cross-compiler.
- GNU binutils.
- NASM.
- xorriso.
- Limine.
- QEMU for emulation and testing.
- A Linux shell environment.

## 34.2 Installing common dependencies

On Debian or Ubuntu:

```bash
sudo apt update

sudo apt install \
    build-essential \
    nasm \
    xorriso \
    qemu-system-x86 \
    git
```

Limine must also be installed or available through the project's bootloader files.

## 34.3 Freestanding compilation

An operating system kernel cannot assume the existence of a normal hosted C runtime.

Kernel source files are compiled in a freestanding environment.

This means ordinary operating system services and hosted library facilities are not automatically available.

The kernel must provide or implement the functionality it needs.

## 34.4 Assembly

NASM assembles the x86-64 entry code into an object file.

The resulting object is linked with the C kernel objects.

## 34.5 Linking

The GNU linker uses the kernel linker script.

The linker script defines:

- The kernel entry point.
- Virtual address placement.
- Section alignment.
- Text section.
- Read-only data section.
- Writable data section.
- Loadable program headers.

The higher-half virtual address is defined in the linker configuration.

## 34.6 ISO preparation

The build script creates the required ISO directory structure.

It places the kernel and Limine configuration in their expected locations.

It also prepares the necessary BIOS and UEFI bootloader files.

## 34.7 Running the build

Make the build script executable:

```bash
chmod +x build.sh
```

Run:

```bash
./build.sh
```

The script performs the configured compilation, linking, and ISO creation steps.

The resulting ISO filename depends on the current build script.

## 34.8 Build troubleshooting

Common build problems include:

**Undefined references**

A source file may be missing from the link command, or a function may be declared without an implementation.

**Missing headers**

An include path may be incorrect, or a required header may be missing.

**Linker errors**

The linker script may contain an invalid section or address configuration.

**Missing Limine files**

The bootloader installation may be incomplete.

**Compiler warnings treated as errors**

Unused functions, incompatible pointer types, and other warnings may prevent compilation when strict compiler flags are enabled.

---

# 35. Running AngelOS in QEMU

QEMU is the primary environment for testing AngelOS.

It provides an emulated x86-64 machine.

## 35.1 Basic launch

A typical launch command is:

```bash
qemu-system-x86_64 \
    -cdrom build/angelos.iso \
    -m 512M \
    -serial stdio
```

The ISO path should match the actual output of the build script.

## 35.2 Memory configuration

The following option provides 512 MB of emulated memory:

```bash
-m 512M
```

Different memory sizes can be tested to evaluate PMM behavior.

## 35.3 Serial communication

The option:

```bash
-serial stdio
```

redirects the emulated serial port to the host terminal.

This is particularly useful for interacting with AngelOS during development.

## 35.4 Keyboard input

QEMU provides an emulated keyboard device.

The PS/2 keyboard driver receives input through the emulated controller.

## 35.5 Development workflow

A typical development cycle is:

1. Modify the kernel source.
2. Run the build script.
3. Check the build output.
4. Start QEMU.
5. Observe initialization messages.
6. Test the relevant subsystem.
7. Check for errors.
8. Modify the implementation if necessary.
9. Repeat.

---

# 36. Persistent Storage Configuration

AngelOS can use a QEMU disk image as persistent ATA storage.

## 36.1 Creating a new image

A new raw disk image can be created using:

```bash
qemu-img create -f raw angelos-disk.img 16M
```

This creates a 16 MB image.

This operation should only be used when creating a new disk image.

It must not be used to recreate an image containing important filesystem data.

## 36.2 Attaching the image

Attach the image using:

```bash
qemu-system-x86_64 \
    -cdrom build/angelos.iso \
    -m 512M \
    -serial stdio \
    -drive file=angelos-disk.img,format=raw,if=ide
```

The disk is exposed through an IDE-compatible interface.

## 36.3 Preserving filesystem data

The disk image contains persistent filesystem information.

To preserve data across restarts:

- Keep the original disk image.
- Continue using the same image when launching QEMU.
- Do not recreate or overwrite it.
- Do not change the filesystem format without a migration plan.

## 36.4 Testing persistence

A useful test is to create a file, save its contents, reboot, and then read it again.

This verifies that the filesystem is not relying exclusively on temporary in-memory state.

---

# 37. Complete Command Reference

The following table lists the currently implemented shell commands.

## General

| Command | Arguments | Description |
|---|---|---|
| `help` | None | Display help |
| `clear` | None | Clear terminal |
| `info` | None | Display system information |
| `version` | None | Display version |
| `uptime` | None | Display uptime |
| `date` | None | Display date |
| `time` | None | Display time |
| `memory` | None | Display memory information |
| `echo` | Text | Print text |

## Aliases

| Command | Arguments | Description |
|---|---|---|
| `alias` | None | List aliases |
| `alias` | `name=command` | Create alias |
| `unalias` | Name | Remove alias |

## Filesystem

| Command | Arguments | Description |
|---|---|---|
| `crdir` | Name/path | Create directory |
| `list` | None | List current directory |
| `sdir` | Path | Change directory |
| `crf` | Name/path | Create file |
| `src` | Filename | Read file |
| `write` | Filename, text | Write file |
| `rm` | Name/path | Remove node |
| `pwd` | None | Print working directory |
| `edit` | Filename | Open text editor |

## Storage

| Command | Arguments | Description |
|---|---|---|
| `disk` | None | List disks |
| `diskread` | Device, LBA | Read disk sector |

## System

| Command | Arguments | Description |
|---|---|---|
| `reboot` | None | Restart system |
| `shutdown` | None | Shut down system |

---

# 38. Example Workflows

## 38.1 Creating a directory hierarchy

```text
angel> crdir projects
angel> sdir projects
angel> crdir angelos
angel> sdir angelos
angel> pwd
/projects/angelos
```

## 38.2 Creating and reading a file

```text
angel> crf notes.txt
angel> write notes.txt My first AngelOS file
angel> src notes.txt
```

Expected contents:

```text
My first AngelOS file
```

## 38.3 Editing a file

```text
angel> edit notes.txt
```

Inside the editor:

```text
My first AngelOS file
This file was edited using AngelOS.
```

Press Ctrl+S to save and Ctrl+Q to exit.

Back in the shell:

```text
angel> src notes.txt
```

## 38.4 Navigating directories

```text
angel> sdir ..
angel> pwd
/projects
```

## 38.5 Listing files

```text
angel> list
```

## 38.6 Returning to root

```text
angel> sdir /
angel> pwd
/
```

## 38.7 Creating an alias

```text
angel> alias ll=list
angel> ll
```

## 38.8 Inspecting memory

```text
angel> memory
```

## 38.9 Inspecting storage

```text
angel> disk
angel> diskread 0 0
```

## 38.10 Checking uptime

```text
angel> uptime
```

---

# 39. Current Implementation Limitations

AngelOS is an actively developed hobby operating system.

Its existing functionality provides a foundation for further development, but several major operating system features are not yet complete.

## 39.1 Process management

There is no complete process creation, scheduling, or termination system.

## 39.2 User-space execution

The shell and other current kernel functionality execute in the kernel environment.

There is no complete user-space application execution model.

## 39.3 System calls

A complete system call interface has not yet been implemented.

## 39.4 Multitasking

The PIT provides timer interrupts, but a complete preemptive scheduler is not currently implemented.

## 39.5 Filesystem capacity

The filesystem has a fixed limit of 128 nodes.

File contents are limited to 511 usable bytes.

## 39.6 Storage interfaces

The current storage implementation focuses on ATA PIO.

DMA, AHCI, and NVMe are not currently implemented.

## 39.7 Graphics

AngelOS does not yet have a complete graphical user interface or window manager.

## 39.8 Networking

A complete networking stack is not currently available.

## 39.9 USB

USB host-controller support is not currently implemented.

## 39.10 Application environment

There is no complete executable loader, standard library, or independent application environment.

---

# 40. Future Development

The future direction of AngelOS is to evolve from a basic monolithic kernel with an interactive shell into a more complete operating system.

The following areas represent major development goals.

## 40.1 Advanced kernel architecture

Potential developments:

- Improved kernel subsystem interfaces.
- Kernel module infrastructure.
- Process management.
- Thread management.
- Kernel scheduling.
- User-mode execution.
- System call handling.
- Improved exception reporting.
- Kernel synchronization primitives.

## 40.2 Advanced memory management

Potential developments:

- Independent address spaces.
- User-space page tables.
- Page-level protection.
- Demand paging.
- Copy-on-write.
- Shared memory.
- Memory-mapped files.
- Improved physical allocation strategies.

## 40.3 Advanced filesystem

Potential developments:

- Dynamic file allocation.
- Larger files.
- More directory entries.
- Improved disk layout.
- File permissions.
- File metadata.
- Directory indexing.
- Filesystem consistency checks.
- Recovery mechanisms.
- Filesystem version migration.

## 40.4 Improved shell

Potential developments:

- Command pipelines.
- Input redirection.
- Output redirection.
- Environment variables.
- Shell scripts.
- Application execution.
- Persistent command history.
- Improved completion.
- Background tasks.

## 40.5 Advanced text editor

Potential developments:

- Arbitrary cursor movement.
- Text insertion at any position.
- Text selection.
- Copy and paste.
- Search and replace.
- Dynamic document allocation.
- Larger documents.
- Syntax highlighting.
- Improved terminal rendering.

## 40.6 Advanced hardware support

Potential developments:

- PCI device enumeration.
- USB controller drivers.
- AHCI.
- NVMe.
- APIC.
- I/O APIC.
- Improved ACPI support.
- Framebuffer graphics.
- Mouse input.
- Audio devices.

## 40.7 Networking

A networking subsystem could eventually provide:

- Ethernet drivers.
- MAC address management.
- ARP.
- IPv4.
- IPv6.
- ICMP.
- UDP.
- TCP.
- DNS.
- Socket interfaces.

## 40.8 Graphical environment

A future graphical environment could include:

- Framebuffer initialization.
- Graphics primitives.
- Font rendering.
- Mouse input.
- Window management.
- Desktop environment.
- Graphical terminal.
- Graphical applications.

## 40.9 Application execution

A complete userspace environment could introduce:

- Executable loading.
- ELF parsing.
- Process creation.
- Application memory isolation.
- System libraries.
- Standard input and output.
- System call wrappers.
- Application permissions.

---

# 41. Contributing

AngelOS is an ongoing operating system development project.

Contributions and technical discussion are welcome.

Potential areas for development include:

- Kernel architecture.
- x86-64 assembly.
- Memory management.
- Interrupt handling.
- Hardware drivers.
- ATA storage.
- Filesystem design.
- Shell functionality.
- Editor development.
- Testing.
- Debugging.
- Documentation.

When modifying the kernel, existing functionality should be preserved whenever possible.

Changes to core subsystems should be tested independently before being integrated into the main initialization sequence.

Filesystem changes require particular care because incompatible disk format modifications can destroy existing data.
