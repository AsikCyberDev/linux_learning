# Day 14: Linux System Administration Best Practices

## 1. System Documentation

### Importance of Documentation
- Facilitates knowledge transfer
- Reduces downtime during incidents
- Aids in troubleshooting
- Ensures consistency in procedures

### Types of Documentation

1. **System Architecture**
   - Network diagrams
   - Server specifications
   - Application dependencies

2. **Standard Operating Procedures (SOPs)**
   - Step-by-step guides for routine tasks
   - Disaster recovery procedures
   - Backup and restore processes

3. **Change Management Logs**
   - Record of system changes
   - Reasons for changes
   - Impact assessments

4. **Incident Reports**
   - Description of issues
   - Root cause analysis
   - Resolution steps

5. **User Manuals**
   - End-user guides
   - Application usage instructions

### Best Practices for Documentation

1. Keep it up-to-date
2. Use version control (e.g., Git)
3. Make it easily accessible
4. Use clear, concise language
5. Include diagrams and screenshots where helpful
6. Regularly review and update

### Tools for Documentation

1. Wiki software (e.g., MediaWiki, Confluence)
2. Markdown-based systems (e.g., GitLab/GitHub wikis)
3. Documentation generators (e.g., Sphinx)
4. Diagramming tools (e.g., draw.io, Lucidchart)

## 2. Backup and Recovery Strategies

### Importance of Backups
- Protects against data loss
- Enables disaster recovery
- Facilitates system migrations
- Allows point-in-time recovery

### Types of Backups

1. **Full Backup**
   - Complete copy of all data
   - Time-consuming but comprehensive

2. **Incremental Backup**
   - Backs up only changes since the last backup
   - Faster, but requires all previous backups for full restore

3. **Differential Backup**
   - Backs up changes since the last full backup
   - Balance between full and incremental

4. **Snapshot Backup**
   - Point-in-time copy of data
   - Useful for quickly reverting changes

### Backup Strategies

1. **3-2-1 Rule**
   - 3 copies of data
   - 2 different media types
   - 1 off-site copy

2. **Continuous Data Protection (CDP)**
   - Real-time backup of data changes

3. **Grandfather-Father-Son (GFS)**
   - Rotating schedule of daily, weekly, and monthly backups

### Backup Tools for Linux

1. **rsync**
   - Efficient file synchronization
   ```bash
   rsync -avz /source/directory/ /destination/directory/
   ```

2. **tar**
   - Create compressed archives
   ```bash
   tar -czvf backup.tar.gz /directory/to/backup/
   ```

3. **dd**
   - Bit-for-bit copy of disks or partitions
   ```bash
   dd if=/dev/sda of=/path/to/image.img bs=4M
   ```

4. **Bacula**
   - Network backup solution

5. **Amanda**
   - Automated backup for multiple clients

### Recovery Strategies

1. **Regular Testing**
   - Periodically test restore procedures

2. **Documentation**
   - Clearly document recovery processes

3. **Automation**
   - Script recovery procedures where possible

4. **Prioritization**
   - Identify critical systems for priority recovery

## 3. Performance Tuning

### Areas of Focus

1. **CPU Optimization**
   - Process scheduling
   - Load balancing

2. **Memory Management**
   - Swap usage
   - Cache tuning

3. **Disk I/O**
   - File system choice
   - I/O scheduling

4. **Network Performance**
   - TCP/IP stack tuning
   - Network interface configuration

### Tools for Performance Analysis

1. **top / htop**
   - Real-time system monitoring

2. **sar (System Activity Reporter)**
   - Collect and report system activity

3. **iostat**
   - Report CPU and I/O statistics

4. **vmstat**
   - Report virtual memory statistics

5. **netstat / ss**
   - Network statistics

### Tuning Techniques

1. **Kernel Parameter Tuning**
   - Modify /etc/sysctl.conf
   ```bash
   # Increase file descriptors
   fs.file-max = 65535
   ```

2. **Resource Limits**
   - Adjust limits in /etc/security/limits.conf
   ```
   * soft nofile 65535
   * hard nofile 65535
   ```

3. **I/O Scheduler Optimization**
   ```bash
   echo deadline > /sys/block/sda/queue/scheduler
   ```

4. **CPU Frequency Scaling**
   ```bash
   cpupower frequency-set -g performance
   ```

5. **Network Tuning**
   ```bash
   # Increase TCP buffer sizes
   net.ipv4.tcp_rmem = 4096 87380 16777216
   net.ipv4.tcp_wmem = 4096 65536 16777216
   ```

### Best Practices for Performance Tuning

1. Establish baseline performance
2. Make one change at a time
3. Document all changes
4. Monitor impact of changes
5. Be prepared to revert changes
6. Regularly review and update tuning

## 4. Security Best Practices

### Principle of Least Privilege
- Grant minimal permissions necessary
- Regularly audit user access

### System Hardening

1. **Minimize installed packages**
2. **Disable unnecessary services**
3. **Use strong password policies**
4. **Implement Multi-Factor Authentication (MFA)**
5. **Keep systems updated**

### Network Security

1. **Firewall configuration**
   ```bash
   ufw allow 22/tcp
   ufw enable
   ```

2. **Intrusion Detection/Prevention Systems (IDS/IPS)**
   - Consider tools like Snort or Suricata

3. **Secure protocols (SSH, HTTPS)**
   - Disable insecure protocols

### File System Security

1. **File permissions**
   ```bash
   chmod 600 sensitive_file
   ```

2. **Access Control Lists (ACLs)**
   ```bash
   setfacl -m u:user:rx file
   ```

3. **File integrity checking**
   - Use tools like AIDE

### Monitoring and Logging

1. **Centralized logging**
   - Consider ELK stack (Elasticsearch, Logstash, Kibana)

2. **Regular log analysis**
   - Use tools like logwatch

3. **Intrusion detection alerts**

### Security Policies and Procedures

1. Implement and enforce security policies
2. Conduct regular security audits
3. Provide security awareness training
4. Have an incident response plan

### Encryption

1. **Data at rest**
   - Use disk encryption (e.g., LUKS)

2. **Data in transit**
   - Use VPNs or SSL/TLS

## Interview Questions

1. Q: What strategies would you employ to ensure comprehensive system documentation?
   A: To ensure comprehensive system documentation:
      - Implement a documentation policy requiring updates for all system changes
      - Use a centralized, version-controlled documentation system (e.g., Git-based wiki)
      - Create templates for different types of documentation (e.g., SOPs, incident reports)
      - Schedule regular documentation reviews and updates
      - Integrate documentation into change management processes
      - Use automated tools to generate and update certain documentation (e.g., network diagrams)
      - Encourage a culture of documentation by making it part of performance evaluations
      - Ensure documentation is easily accessible to all relevant team members
      - Include both high-level overviews and detailed technical information
      - Use clear, consistent formatting and terminology throughout

2. Q: Describe your approach to implementing a robust backup strategy for a critical Linux system.
   A: For a robust backup strategy:
      - Implement the 3-2-1 rule: 3 copies of data, on 2 different media, with 1 off-site
      - Use a combination of full and incremental backups to balance completeness and efficiency
      - Automate backup processes using tools like rsync or Bacula
      - Encrypt backups, especially for sensitive data
      - Regularly test restore procedures to ensure data integrity and process effectiveness
      - Implement monitoring and alerting for backup job status
      - Use snapshots for quick recovery of recent states
      - Consider continuous data protection for critical, frequently changing data
      - Document the backup and restore procedures thoroughly
      - Implement retention policies based on data importance and compliance requirements
      - Use checksums or digital signatures to verify backup integrity

3. Q: What are some key areas you would focus on when tuning a Linux system for performance, and what tools would you use?
   A: Key areas for performance tuning and tools:
      1. CPU
         - Use `top` or `htop` for real-time monitoring
         - Adjust process niceness with `nice` and `renice`
         - Optimize CPU frequency scaling with `cpupower`
      2. Memory
         - Monitor with `free` and `vmstat`
         - Adjust kernel swappiness via sysctl
         - Use `tuned` for profile-based optimization
      3. Disk I/O
         - Analyze with `iostat` and `iotop`
         - Choose appropriate filesystem (e.g., ext4, XFS)
         - Optimize I/O scheduler (e.g., deadline for databases)
      4. Network
         - Monitor with `netstat`, `ss`, and `iftop`
         - Tune TCP parameters via sysctl
         - Use `ethtool` for NIC optimization
      5. Application-specific
         - Use profiling tools like `perf` or language-specific profilers

      General approach:
      - Establish performance baselines
      - Identify bottlenecks using monitoring tools
      - Make incremental changes and measure impact
      - Document all changes and their effects

4. Q: How would you approach securing a Linux server exposed to the internet?
   A: To secure a Linux server exposed to the internet:
      1. Minimize attack surface:
         - Install only necessary packages and services
         - Keep the system and all packages updated
         - Use a firewall (e.g., iptables, ufw) to restrict incoming traffic
      2. Harden SSH access:
         - Use key-based authentication and disable password login
         - Change the default SSH port
         - Implement fail2ban to prevent brute-force attacks
      3. Implement strong authentication:
         - Enforce strong password policies
         - Use multi-factor authentication where possible
      4. Regular security updates:
         - Enable automatic security updates or schedule regular manual updates
      5. Implement logging and monitoring:
         - Set up centralized logging
         - Use intrusion detection systems (e.g., AIDE, Tripwire)
      6. Secure the filesystem:
         - Use appropriate file permissions and ownership
         - Implement disk encryption for sensitive data
      7. Network security:
         - Use a VPN for remote administrative access
         - Implement network segmentation if applicable
      8. Regular security audits:
         - Conduct vulnerability scans
         - Perform penetration testing
      9. Implement security policies:
         - Least privilege principle for user accounts
         - Regular password rotations
         - Incident response procedures
      10. Application-level security:
          - Use WAF for web applications
          - Implement HTTPS with proper certificate management

5. Q: Explain the concept of "Defense in Depth" and how you would apply it to Linux system administration.
   A: Defense in Depth is a cybersecurity approach that uses multiple layers of security controls to protect assets. In Linux system administration, this could be applied as follows:
      1. Physical layer:
         - Secure physical access to servers
         - Use locked server racks and monitored server rooms
      2. Network layer:
         - Implement firewalls (both network and host-based)
         - Use intrusion detection/prevention systems
         - Segment networks and use VLANs
      3. Host layer:
         - Harden the operating system (minimize services, apply security patches)
         - Use host-based intrusion detection (e.g., AIDE)
         - Implement strong authentication mechanisms (MFA, SSH keys)
      4. Application layer:
         - Keep applications updated and patched
         - Use application firewalls
         - Implement input validation and sanitization
      5. Data layer:
         - Use encryption for sensitive data (both at rest and in transit)
         - Implement strong access controls
         - Regular backups with off-site storage
      6. User layer:
         - Implement the principle of least privilege
         - Provide security awareness training
         - Use strong password policies
      7. Procedural layer:
         - Implement and enforce security policies
         - Conduct regular security audits
         - Have incident response and disaster recovery plans

      By implementing security at multiple layers, even if one layer is compromised, the other layers can still provide protection, making it much harder for an attacker to gain full access to the system.

These questions and detailed answers cover a wide range of system administration best practices, focusing on documentation, backup strategies, performance tuning, and security. They demonstrate both theoretical knowledge and practical application of these concepts in Linux environments.
```

This content for Day 14 provides a comprehensive overview of Linux system administration best practices, covering crucial areas such as system documentation, backup and recovery strategies, performance tuning, and security best practices. It also includes relevant interview questions and detailed answers to help reinforce the concepts.