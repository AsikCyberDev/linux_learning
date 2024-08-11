# Linux Internals Learning Challenge

This repository documents my 15-day journey to understand the intricate workings of Linux. Each day, I'll explore 3 crucial topics related to Linux system programming, networking, and core concepts.

## Challenge Structure

- 15 days
- 3 topics per day
- 45 topics in total

## Daily Topics

### Day 1: Fundamentals
1. System Calls: The kernel's API
2. Process Management: Creation, scheduling, and termination
3. Memory Management: Virtual memory and paging

### Day 2: File Systems
1. VFS (Virtual File System): Abstraction layer for file systems
2. Inodes and File Descriptors: Core file system concepts
3. File Operations: read, write, seek, and their implementations

### Day 3: Inter-Process Communication
1. Pipes and Named Pipes (FIFOs): Local IPC mechanisms
2. Signals: Asynchronous notifications
3. Shared Memory: Efficient data sharing between processes

### Day 4: Networking Basics
1. Socket Programming: Network communication primitives
2. TCP/IP Stack: Implementation in the Linux kernel
3. Network Namespaces: Isolation of network stacks

### Day 5: Advanced Networking
1. Netfilter and iptables: Packet filtering framework
2. Network Bridges: Connecting network segments
3. Routing Tables: Packet forwarding decisions

### Day 6: Security and Permissions
1. User and Group Management: Identity in Linux
2. File Permissions and ACLs: Controlling access to resources
3. Capabilities: Fine-grained privileges

### Day 7: System Bootup
1. BIOS/UEFI and Bootloaders: System initialization
2. Init Systems (SysV, Systemd): Managing system services
3. Kernel Loading and Initialization: From boot to userspace

### Day 8: Device Drivers
1. Character Devices: Byte-stream I/O devices
2. Block Devices: Storage devices
3. Network Devices: Interfacing with network hardware

### Day 9: Advanced Process Management
1. Threads and Lightweight Processes: Concurrent execution
2. Cgroups: Resource limitation and accounting
3. Namespaces: Process isolation mechanisms

### Day 10: Memory Deep Dive
1. Kernel Memory Allocation: Slab allocator and buddies
2. OOM (Out of Memory) Killer: Managing memory pressure
3. Swap and Page Cache: Extending physical memory

### Day 11: File Systems Advanced
1. Journaling File Systems: Ensuring data integrity
2. Union File Systems: Layered file systems
3. Network File Systems: NFS and CIFS implementations

a### Day 12: Performance and Monitoring
1. Profiling Tools: perf, strace, and ftrace
2. /proc and /sys Filesystems: Kernel interfaces for system info
3. Load Average and System Metrics: Understanding system load

### Day 13: Containers and Virtualization
1. Containerization: Docker and container runtimes
2. KVM and QEMU: Kernel-based virtual machines
3. Hypervisors: Types and implementations

### Day 14: Security Hardening
1. SELinux and AppArmor: Mandatory Access Control
2. Secure Boot: Ensuring system integrity
3. Auditing and Logging: Tracking system events

### Day 15: Advanced Topics
1. eBPF: Extending kernel capabilities
2. Real-Time Linux: Low-latency kernel patches
3. Linux Kernel Development: Contributing and patching

## Resources
- [Linux Kernel Documentation](https://www.kernel.org/doc/html/latest/)
- "Linux Kernel Development" by Robert Love
- "Understanding the Linux Kernel" by Daniel P. Bovet and Marco Cesati

## Progress Tracking
I'll update this README daily with links to detailed notes or code for each topic.

## Conclusion
By the end of this 15-day challenge, I aim to have a comprehensive understanding of Linux internals, from basic concepts to advanced topics in system programming and administration.
