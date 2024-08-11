
# Day 12: Containerization and Virtualization

## 1. Docker Fundamentals

### Introduction
Docker is a platform for developing, shipping, and running applications in containers. Containers allow developers to package up an application with all of its dependencies and ship it as one package.

### Key Concepts

1. **Container**
   - Lightweight, standalone, executable package that includes everything needed to run a piece of software
   - Includes the code, runtime, system tools, libraries, and settings

2. **Image**
   - Read-only template used to create containers
   - Often based on another image, with additional customization

3. **Dockerfile**
   - Text file that contains instructions for building a Docker image

4. **Docker Hub**
   - Cloud-based registry service for storing and sharing Docker images

5. **Docker Compose**
   - Tool for defining and running multi-container Docker applications

### Basic Docker Commands

1. **Run a container**
   ```
   docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
   ```

2. **List containers**
   ```
   docker ps [OPTIONS]
   ```

3. **Stop a container**
   ```
   docker stop [OPTIONS] CONTAINER [CONTAINER...]
   ```

4. **Remove a container**
   ```
   docker rm [OPTIONS] CONTAINER [CONTAINER...]
   ```

5. **List images**
   ```
   docker images [OPTIONS] [REPOSITORY[:TAG]]
   ```

6. **Build an image from a Dockerfile**
   ```
   docker build [OPTIONS] PATH | URL | -
   ```

### Practical Example: Creating a Simple Docker Container

1. Create a Dockerfile:

```Dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

2. Build the image:
```
docker build -t my-nginx .
```

3. Run the container:
```
docker run -d -p 8080:80 my-nginx
```

### Docker Compose Example

Create a `docker-compose.yml` file:

```yaml
version: '3'
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
  database:
    image: postgres:12
    environment:
      POSTGRES_PASSWORD: example
```

Run with:
```
docker-compose up -d
```

### Gotchas and Best Practices

1. **Image Size**: Use minimal base images when possible.
2. **Security**: Avoid running containers as root.
3. **Persistence**: Use volumes for data that needs to persist.
4. **Networking**: Understand Docker's networking models.
5. **Cleanup**: Regularly remove unused containers and images.

### Interview Questions

1. Q: What's the difference between a Docker container and a virtual machine?
   A: The main differences are:
      - Containers share the host OS kernel, while VMs have their own OS
      - Containers are more lightweight and start up faster
      - VMs provide stronger isolation but with more overhead
      - Containers are ideal for microservices, while VMs are better for running multiple applications with different OS requirements

2. Q: Explain the concept of Docker layers. How do they contribute to efficiency?
   A: Docker images are built using a layered filesystem:
      - Each instruction in a Dockerfile creates a new layer
      - Layers are cached and reused in subsequent builds
      - Only changed layers need to be rebuilt
      - Layers are shared between images, saving disk space and network bandwidth
      This system makes building, sharing, and updating images more efficient.

3. Q: How would you persist data in a Docker container?
   A: There are several ways to persist data:
      1. Volumes: Docker-managed directories on the host filesystem
         ```
         docker run -v my_volume:/app/data my_image
         ```
      2. Bind mounts: Mount a specific directory from the host
         ```
         docker run -v /host/path:/container/path my_image
         ```
      3. tmpfs mounts: For storing temporary data in memory
         ```
         docker run --tmpfs /app/temp my_image
         ```
      Volumes are generally the preferred method as they're managed by Docker and work well across different operating systems.

4. Q: What is Docker Compose and when would you use it?
   A: Docker Compose is a tool for defining and running multi-container Docker applications:
      - It uses a YAML file to configure application services
      - It allows you to start all services with a single command
      - It's useful for development, testing, and staging environments
      - It simplifies the process of linking containers and defining networks
      You would use Docker Compose when your application requires multiple interconnected services, like a web server, database, and caching layer.

5. Q: How do you handle secrets (like passwords) in Docker?
   A: There are several approaches to handling secrets in Docker:
      1. Docker Secrets (in Swarm mode):
         ```
         docker secret create my_secret my_file.txt
         docker service create --secret my_secret my_image
         ```
      2. Environment variables (less secure):
         ```
         docker run -e PASSWORD=mysecret my_image
         ```
      3. Using external secret management tools like HashiCorp Vault
      4. Mounting secrets at runtime:
         ```
         docker run -v /path/to/secrets:/run/secrets my_image
         ```
      The best practice is to use Docker Secrets or a dedicated secret management tool, avoiding hardcoded secrets in images or environment variables.

## 2. Virtualization with KVM

### Introduction
KVM (Kernel-based Virtual Machine) is a virtualization module in the Linux kernel that allows the kernel to function as a hypervisor. It enables running multiple virtual machines on a single physical host.

### Key Concepts

1. **Hypervisor**
   - Software that creates and runs virtual machines
   - KVM is a Type-1 (bare-metal) hypervisor

2. **Virtual Machine (VM)**
   - Emulation of a computer system
   - Provides functionality of a physical computer

3. **QEMU**
   - Machine emulator often used with KVM
   - Provides device emulation

4. **libvirt**
   - API and management tool for platform virtualization

### Setting Up KVM

1. Check for hardware virtualization support:
   ```
   egrep -c '(vmx|svm)' /proc/cpuinfo
   ```

2. Install KVM and related tools:
   ```
   sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
   ```

3. Add user to libvirt group:
   ```
   sudo adduser $USER libvirt
   ```

### Managing VMs with virsh

1. **List VMs**
   ```
   virsh list --all
   ```

2. **Start a VM**
   ```
   virsh start vm_name
   ```

3. **Stop a VM**
   ```
   virsh shutdown vm_name
   ```

4. **Delete a VM**
   ```
   virsh undefine vm_name
   ```

### Creating a VM with virt-install

```bash
virt-install \
  --name ubuntu20 \
  --ram 2048 \
  --disk path=/var/lib/libvirt/images/ubuntu20.qcow2,size=10 \
  --vcpus 2 \
  --os-type linux \
  --os-variant ubuntu20.04 \
  --network bridge=virbr0 \
  --graphics none \
  --console pty,target_type=serial \
  --location 'http://archive.ubuntu.com/ubuntu/dists/focal/main/installer-amd64/' \
  --extra-args 'console=ttyS0,115200n8 serial'
```

### Gotchas and Best Practices

1. **Resource Allocation**: Carefully manage CPU, memory, and storage allocation.
2. **Networking**: Understand different networking modes (NAT, bridged, etc.).
3. **Snapshots**: Use VM snapshots for backups and before major changes.
4. **Security**: Keep the host system and VMs updated and secured.
5. **Performance Monitoring**: Regularly monitor host and VM performance.

### Interview Questions

1. Q: What is the difference between KVM and other virtualization technologies like VirtualBox or VMware?
   A: Key differences include:
      - KVM is integrated into the Linux kernel, making it more efficient
      - KVM is a Type-1 (bare-metal) hypervisor, while VirtualBox is Type-2 (hosted)
      - KVM generally offers better performance for Linux guests
      - VirtualBox and VMware have more user-friendly interfaces
      - KVM is open-source and free, while VMware has proprietary versions
      - KVM is primarily managed through command-line tools, though GUI tools exist

2. Q: Explain the relationship between KVM, QEMU, and libvirt.
   A:
      - KVM: Kernel module that provides access to hardware virtualization features
      - QEMU: Provides hardware emulation and works with KVM for better performance
      - libvirt: Management layer that provides a common API for managing different virtualization technologies, including KVM/QEMU
   Together, they form a complete virtualization stack: KVM provides the core virtualization infrastructure, QEMU handles hardware emulation, and libvirt provides management capabilities.

3. Q: How would you migrate a KVM virtual machine from one host to another?
   A: There are several ways to migrate a KVM VM:
      1. Offline migration:
         - Shut down the VM
         - Copy the VM's disk image and configuration files to the new host
         - Define the VM on the new host using the copied files
      2. Live migration (requires shared storage):
         ```
         virsh migrate --live vm_name qemu+ssh://destination_host/system
         ```
      3. Using virt-manager GUI tool for live or offline migration

      The choice depends on factors like downtime tolerance, network bandwidth, and storage setup.

4. Q: What are some best practices for securing KVM virtual machines?
   A: Some best practices include:
      - Keep the host system and VMs updated
      - Use SELinux or AppArmor for additional security
      - Implement network segmentation (e.g., separate bridge for VMs)
      - Use virtio-random for better VM entropy
      - Disable unnecessary features and devices
      - Use encrypted storage for sensitive VMs
      - Implement strong access controls for the hypervisor
      - Regularly audit and monitor VM activities
      - Use secure protocols (like SSH) for remote management

5. Q: How does KVM achieve near-native performance for VMs?
   A: KVM achieves near-native performance through several mechanisms:
      - Direct execution: VMs run directly on the host CPU using hardware virtualization extensions
      - Memory management: Uses hardware-assisted paging (e.g., Intel EPT or AMD RVI)
      - I/O virtualization: Utilizes technologies like virtio for efficient I/O operations
      - Kernel integration: Being part of the Linux kernel allows for efficient resource management
      - Paravirtualization: Optimized drivers (virtio) for network and disk operations
      - CPU feature passthrough: Allows VMs to use advanced CPU features
      These features combined allow KVM to minimize virtualization overhead and provide performance close to bare-metal systems.

This content covers the fundamentals of Docker containerization and KVM virtualization, providing both theoretical knowledge and practical examples. It should give a comprehensive understanding of these critical aspects of modern infrastructure management.
```

This completes the content for Day 12, covering Containerization with Docker and Virtualization with KVM in detail.