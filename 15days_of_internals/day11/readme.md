
# Day 11: System Monitoring and Logging

## 1. Linux System Monitoring Tools

### Introduction
System monitoring is crucial for maintaining the health and performance of Linux systems. Various tools are available to monitor different aspects of system performance, resource utilization, and processes.

### Key Monitoring Areas

1. **CPU Usage**
2. **Memory Utilization**
3. **Disk I/O**
4. **Network Activity**
5. **Process Management**

### Essential Monitoring Tools

1. **top / htop**
   - Real-time view of system processes
   - Shows CPU, memory, and process information

   Example usage:
   ```
   top
   htop
   ```

2. **vmstat**
   - Reports virtual memory statistics
   - Provides information on system processes, memory, paging, block I/O, traps, and CPU activity

   Example usage:
   ```
   vmstat 5 10  # Report every 5 seconds, 10 times
   ```

3. **iostat**
   - Reports CPU statistics and I/O statistics for devices and partitions

   Example usage:
   ```
   iostat -x 5 3  # Extended report, every 5 seconds, 3 times
   ```

4. **netstat / ss**
   - Network statistics
   - Shows network connections, routing tables, interface statistics

   Example usage:
   ```
   netstat -tuln  # TCP and UDP listening sockets
   ss -s  # Socket statistics
   ```

5. **sar (System Activity Reporter)**
   - Collects, reports, and saves system activity information

   Example usage:
   ```
   sar -u 5 3  # CPU usage, every 5 seconds, 3 times
   ```

6. **dstat**
   - Versatile tool for generating system resource statistics

   Example usage:
   ```
   dstat -cdngy  # CPU, disk, network, page, system stats
   ```

### Advanced Monitoring Tools

1. **Nagios**
   - Comprehensive monitoring system for networks and infrastructure

2. **Prometheus**
   - Monitoring system and time series database

3. **Grafana**
   - Analytics and interactive visualization web application

4. **Zabbix**
   - Enterprise-class open source distributed monitoring solution

### Practical Example: Simple System Monitoring Script

Here's a bash script that provides a basic system overview:

```bash
#!/bin/bash

echo "System Monitoring Report"
echo "========================"

echo -e "\nCPU Usage:"
top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1"%"}'

echo -e "\nMemory Usage:"
free -m | awk 'NR==2{printf "Used: %s MB (%.2f%%)\n", $3,$3*100/$2 }'

echo -e "\nDisk Usage:"
df -h | awk '$NF=="/"{printf "Used: %d GB (%s)\n", $3,$5}'

echo -e "\nTop 5 Processes by CPU usage:"
ps aux --sort=-%cpu | head -n 6

echo -e "\nCurrent Network Connections:"
netstat -tuln | grep LISTEN
```

### Gotchas and Best Practices

1. **Resource Impact**: Be aware that monitoring tools themselves consume resources.
2. **Data Retention**: Plan for long-term storage and analysis of monitoring data.
3. **Alerting**: Set up alerts for critical thresholds to proactively address issues.
4. **Customization**: Tailor monitoring to your specific system and application needs.
5. **Regular Review**: Periodically review and adjust monitoring strategies.

### Interview Questions

1. Q: What's the difference between `top` and `htop`?
   A: Both `top` and `htop` are interactive process viewers, but `htop` offers several advantages:
      - More colorful and intuitive interface
      - Ability to scroll vertically and horizontally
      - Mouse operation support
      - Easier process killing and priority changing
      - Better support for multi-core systems
   `top` is typically pre-installed on most systems, while `htop` often needs to be installed separately.

2. Q: How would you monitor disk I/O on a Linux system?
   A: There are several ways to monitor disk I/O:
      - `iostat`: Provides detailed I/O statistics for devices and partitions.
      - `iotop`: Shows I/O usage by processes (similar to top for I/O).
      - `dstat`: Offers a versatile view of various system resources, including disk I/O.
      - `vmstat`: Gives a overview of system performance, including disk operations.
      For example, you could use `iostat -x 5` to get extended disk statistics every 5 seconds.

3. Q: Explain the purpose of the `sar` command and how it's typically used.
   A: `sar` (System Activity Reporter) is a versatile tool for collecting, reporting, and saving system activity information. It's typically used for:
      - Historical performance analysis (it can read data from log files)
      - Real-time system monitoring
      - Generating reports on CPU, memory, disk I/O, and network usage
   `sar` is often configured to run periodically (e.g., via cron) to collect system statistics, which can then be analyzed later. For example, `sar -u 5 3` would report CPU usage every 5 seconds, 3 times.

4. Q: How would you identify which process is using the most memory on a Linux system?
   A: There are several ways to identify the process using the most memory:
      1. Using `top` or `htop` and sorting by memory usage (press 'M' in top)
      2. Using the `ps` command:
         ```
         ps aux --sort=-%mem | head
         ```
      3. Using `smem` for a more accurate representation of memory usage:
         ```
         smem -tk
         ```
      These methods will show you the processes consuming the most memory, allowing you to investigate further if necessary.

5. Q: What are the key differences between Nagios and Prometheus for system monitoring?
   A: Nagios and Prometheus are both popular monitoring systems, but they have different approaches:
      - Architecture: Nagios uses a centralized model, while Prometheus uses a pull model and is more decentralized.
      - Data Model: Nagios focuses on host/service checks, while Prometheus uses a multi-dimensional data model with time series.
      - Alerting: Nagios has built-in alerting, while Prometheus uses a separate Alertmanager component.
      - Visualization: Nagios has basic built-in visualizations, while Prometheus is often used with Grafana for advanced visualizations.
      - Extensibility: Nagios relies heavily on plugins, while Prometheus has a more flexible data collection model with exporters.
      - Scalability: Prometheus generally scales better for large, dynamic environments.
   The choice between them often depends on specific requirements, existing infrastructure, and team expertise.

## 2. Logging and Log Analysis

### Introduction
Logging is a critical aspect of system administration and security in Linux. Proper logging and log analysis help in troubleshooting, security auditing, and understanding system behavior.

### Key Concepts

1. **Syslog**
   - Standard logging protocol for Linux systems
   - Defines severity levels and facilities for messages

2. **Journald**
   - Part of systemd, stores logs in binary format
   - Offers structured logging and improved querying capabilities

3. **Log Rotation**
   - Process of archiving old log files and creating new ones
   - Prevents logs from consuming too much disk space

4. **Log Analysis Tools**
   - Tools for parsing, searching, and analyzing log files

### Important Log Files

1. `/var/log/syslog` or `/var/log/messages`: General system logs
2. `/var/log/auth.log` or `/var/log/secure`: Authentication logs
3. `/var/log/kern.log`: Kernel logs
4. `/var/log/apache2/` or `/var/log/httpd/`: Web server logs
5. `/var/log/mysql/`: MySQL database logs

### Logging Daemons

1. **rsyslog**
   - Enhanced version of syslog
   - Supports TCP, SSL/TLS, content-based filtering

2. **syslog-ng**
   - Next-generation syslog daemon
   - Offers flexible configuration and filtering options

### Log Analysis Tools

1. **grep, awk, sed**
   - Command-line tools for searching and manipulating log files

2. **logwatch**
   - Analyzes log files and emails summaries

3. **ELK Stack (Elasticsearch, Logstash, Kibana)**
   - Powerful stack for collecting, processing, and visualizing logs

4. **Splunk**
   - Enterprise-level log analysis and monitoring tool

### Practical Example: Basic Log Analysis

Here's a bash script to analyze authentication failures:

```bash
#!/bin/bash

echo "Failed SSH Login Attempts:"
grep "Failed password" /var/log/auth.log | awk '{print $1, $2, $3, $11}' | sort | uniq -c | sort -nr

echo -e "\nSuccessful SSH Logins:"
grep "Accepted password" /var/log/auth.log | awk '{print $1, $2, $3, $9}' | sort | uniq -c | sort -nr

echo -e "\nTop 10 IP Addresses for Failed Logins:"
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr | head -n 10
```

### Log Rotation with logrotate

Example logrotate configuration (`/etc/logrotate.d/custom-app`):

```
/var/log/custom-app.log {
    weekly
    rotate 4
    compress
    delaycompress
    missingok
    notifempty
    create 644 root root
}
```

### Gotchas and Best Practices

1. **Timestamp Consistency**: Ensure all systems use synchronized time (NTP).
2. **Log Retention**: Define and implement a log retention policy.
3. **Security**: Protect log files from unauthorized access and tampering.
4. **Remote Logging**: Consider sending logs to a central log server for better security and analysis.
5. **Regular Analysis**: Implement routine log analysis procedures.

### Interview Questions

1. Q: What is the difference between syslog and journald?
   A:
   - Syslog is the traditional logging system in Unix-like operating systems. It typically writes plain text log files to `/var/log/`.
   - Journald is part of systemd and stores logs in a structured, binary format. It offers:
     - Faster querying and filtering
     - Index-based access
     - Automatic log rotation
     - Integration with systemd services
   Journald can forward logs to syslog, allowing both systems to coexist. While syslog is more universally supported, journald offers more advanced features for log management and analysis.

2. Q: How would you investigate failed login attempts on a Linux system?
   A: To investigate failed login attempts:
   1. Check the authentication log file (usually `/var/log/auth.log` or `/var/log/secure`).
   2. Use commands like `grep` to filter for relevant entries:
      ```
      grep "Failed password" /var/log/auth.log
      ```
   3. Use `awk` to extract and format relevant information:
      ```
      grep "Failed password" /var/log/auth.log | awk '{print $1, $2, $3, $11}'
      ```
   4. Use `sort` and `uniq` to summarize attempts:
      ```
      grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr
      ```
   5. Consider using tools like `fail2ban` to automatically block repeated failed attempts.

3. Q: Explain the concept of log rotation and why it's important.
   A: Log rotation is the process of periodically archiving current log files and starting new ones. It's important because:
   - It prevents log files from consuming too much disk space
   - It makes log management and analysis easier by breaking logs into manageable chunks
   - It allows for easy archiving and deletion of old logs
   - It can improve system performance by keeping current log files smaller
   Log rotation typically involves:
   - Compressing old log files
   - Creating new log files
   - Optionally deleting very old logs
   - Restarting or signaling logging services to use the new log files
   Tools like `logrotate` in Linux automate this process based on size, time, or other criteria.

4. Q: What is the ELK stack and how is it used in log management?
   A: The ELK stack consists of three main components:
   - Elasticsearch: A distributed search and analytics engine
   - Logstash: A server-side data processing pipeline that ingests data from multiple sources, transforms it, and sends it to a "stash" like Elasticsearch
   - Kibana: A visualization layer that works on top of Elasticsearch

   It's used in log management to:
   - Collect logs from various sources (using Logstash or Beats)
   - Store and index logs efficiently (in Elasticsearch)
   - Search and analyze logs quickly
   - Create visualizations and dashboards (using Kibana)
   - Set up alerts based on log patterns

   The ELK stack allows for centralized logging, making it easier to correlate events across multiple systems and applications, and to perform complex analyses on large volumes of log data.

5. Q: How would you secure log files on a Linux system?
   A: To secure log files on a Linux system:
   1. Set appropriate permissions:
      ```
      chmod 640 /var/log/syslog
      chown root:adm /var/log/syslog
      ```
   2. Use log rotation to archive and compress old logs
   3. Implement remote logging to a secure, centralized log server
   4. Use file integrity monitoring tools like AIDE to detect unauthorized changes
   5. Enable auditd to monitor file access and changes
   6. Encrypt sensitive log files
   7. Implement SELinux or AppArmor policies to restrict access
   8. Regularly review and analyze logs for suspicious activities
   9. Use secure protocols (like TLS) when transmitting logs over the network
   10. Implement a log retention policy and securely delete old logs

   These measures help protect the integrity and confidentiality of log data, which is crucial for security auditing and incident response.

This content covers the essentials of System Monitoring and Logging in Linux, providing both theoretical knowledge and practical examples. It should give a comprehensive understanding of these critical aspects of Linux system administration.
```

This completes the content for Day 11, covering System Monitoring and Logging in Linux in detail.