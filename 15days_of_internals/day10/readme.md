
# Day 10: Security and Access Control

## 1. Linux Security Modules (LSM)

### Introduction
Linux Security Modules (LSM) is a framework that allows the Linux kernel to support various security models without favoring any specific security implementation. LSM provides a set of hooks in the kernel, which security modules can use to implement additional security checks.

### Key Concepts

1. **Security Hooks**
   - Points in the kernel where security checks can be performed
   - Cover operations like file access, network operations, and process management

2. **Mandatory Access Control (MAC)**
   - Access control enforced by the system, not by file owners
   - Implemented by security modules like SELinux and AppArmor

3. **Discretionary Access Control (DAC)**
   - Traditional Unix permissions (user, group, others)
   - Controlled by file owners

4. **Security Modules**
   - SELinux (Security-Enhanced Linux)
   - AppArmor
   - Smack (Simplified Mandatory Access Control Kernel)
   - TOMOYO

### SELinux (Security-Enhanced Linux)

1. **Features**
   - Fine-grained access control
   - Policy-based security
   - Type enforcement and role-based access control

2. **Modes**
   - Enforcing: Enforces the policy, denying access and logging actions
   - Permissive: Logs policy violations without enforcing
   - Disabled: SELinux is turned off

3. **Policy Types**
   - Targeted: Default policy that confines specific services
   - MLS (Multi-Level Security): For systems with varying security clearances

### AppArmor

1. **Features**
   - Path-based access control
   - Easier to configure than SELinux
   - Profile-based security model

2. **Modes**
   - Enforce: Enforces the defined policy
   - Complain: Logs policy violations without enforcing

3. **Profiles**
   - Define what resources a program can access
   - Can be in enforce or complain mode individually

### Practical Example: AppArmor Profile

Here's a simple AppArmor profile for a hypothetical application:

```
#include <tunables/global>

/usr/bin/myapp {
  #include <abstractions/base>
  #include <abstractions/user-tmp>

  /usr/bin/myapp mr,
  /var/log/myapp.log w,
  /etc/myapp.conf r,
  network tcp,
}
```

This profile allows the application to:
- Read and execute itself
- Write to its log file
- Read its configuration file
- Use TCP networking

### Gotchas and Best Practices

1. **Performance Impact**: Security modules can impact system performance; tune appropriately.
2. **Learning Curve**: LSMs like SELinux have a steep learning curve; invest time in training.
3. **Regular Audits**: Regularly audit and update security policies.
4. **Testing**: Thoroughly test applications with security modules enabled.
5. **Documentation**: Maintain clear documentation of security policies and configurations.

### Interview Questions

1. Q: What is the main difference between SELinux and AppArmor?
   A: The main difference lies in their approach to access control:
      - SELinux uses a label-based system where every process, file, and system object has a security context. It provides fine-grained control based on these contexts.
      - AppArmor uses a path-based approach, where access rules are defined based on file paths. It's generally considered easier to configure but may be less granular than SELinux.

2. Q: Explain the concept of MAC (Mandatory Access Control) and how it differs from DAC (Discretionary Access Control).
   A:
      - MAC is a type of access control where the system enforces the security policy, and users cannot override these restrictions. It's based on security labels or contexts assigned to subjects (processes) and objects (files, resources).
      - DAC is the traditional Unix permission model where the owner of a resource controls its permissions. Users can modify permissions for resources they own.
      MAC provides stronger security by preventing users from accidentally or intentionally weakening the security policy, while DAC offers more flexibility but can lead to security vulnerabilities if not managed carefully.

3. Q: What are the main components of an SELinux policy?
   A: The main components of an SELinux policy include:
      - Types (or domains for processes)
      - Roles
      - Users
      - Rules (allow, deny, auditallow, dontaudit)
      - Contexts (user:role:type:level)
      - Booleans (for runtime policy adjustments)
      These components work together to define what actions are allowed or denied in the system based on the security context of processes and resources.

4. Q: How would you troubleshoot an application that's being blocked by AppArmor?
   A: To troubleshoot an application blocked by AppArmor:
      1. Check AppArmor logs in `/var/log/audit/audit.log` or `/var/log/syslog` for denied actions.
      2. Use `aa-status` to check the status of AppArmor and the profile in question.
      3. Consider switching the profile to complain mode using `aa-complain`.
      4. Use `aa-logprof` to analyze the logs and suggest profile updates.
      5. Manually edit the profile if necessary, adding required permissions.
      6. Test the updated profile in complain mode before switching back to enforce mode.

5. Q: What is the purpose of the LSM framework in the Linux kernel?
   A: The LSM (Linux Security Modules) framework serves several purposes:
      - It provides a modular approach to implementing security models in the Linux kernel.
      - It allows different security models to coexist without favoring any particular implementation.
      - It offers a set of hooks at critical points in the kernel where security decisions need to be made.
      - It enables the development and use of various security modules (like SELinux, AppArmor) without requiring extensive changes to the core kernel code.
      - It allows systems to be configured with different security policies based on their specific requirements.

## 2. User Authentication and Authorization

### Introduction
User authentication and authorization are crucial aspects of Linux security, controlling who can access a system and what they can do once they're in.

### Key Concepts

1. **Authentication**
   - Process of verifying a user's identity
   - Typically involves username and password

2. **Authorization**
   - Determining what actions an authenticated user is allowed to perform
   - Based on user permissions and group memberships

3. **PAM (Pluggable Authentication Modules)**
   - Framework for integrating multiple low-level authentication schemes
   - Configurable authentication mechanism

4. **Sudo**
   - Allows users to run programs with the security privileges of another user

5. **LDAP (Lightweight Directory Access Protocol)**
   - Protocol for accessing and maintaining distributed directory information services

### Authentication Mechanisms

1. **Local Authentication**
   - `/etc/passwd` and `/etc/shadow` files
   - Local user accounts and passwords

2. **LDAP Authentication**
   - Centralized user management
   - Often used in enterprise environments

3. **Kerberos**
   - Network authentication protocol
   - Provides strong authentication for client/server applications

4. **Two-Factor Authentication (2FA)**
   - Combines something you know (password) with something you have (token)
   - Enhances security by requiring two forms of identification

### PAM (Pluggable Authentication Modules)

1. **Configuration**
   - Located in `/etc/pam.d/`
   - Separate configuration files for different services

2. **Module Types**
   - auth: Authentication
   - account: Account management
   - password: Password management
   - session: Session management

3. **Common PAM Modules**
   - pam_unix.so: Traditional Unix authentication
   - pam_ldap.so: LDAP authentication
   - pam_google_authenticator.so: Google Authenticator for 2FA

### Practical Example: Configuring Sudo

Here's an example of configuring sudo to allow a user to run specific commands:

1. Run `visudo` to edit the sudo configuration
2. Add the following line:

```
username ALL=(ALL) /usr/bin/apt-get, /sbin/reboot
```

This allows 'username' to run `apt-get` and `reboot` commands with sudo privileges.

### User and Group Management

1. **User Management Commands**
   - `useradd`: Add a new user
   - `usermod`: Modify user account
   - `userdel`: Delete user account

2. **Group Management Commands**
   - `groupadd`: Create a new group
   - `groupmod`: Modify group information
   - `groupdel`: Delete a group

3. **Password Management**
   - `passwd`: Change user password
   - `chage`: Change user password expiry information

### Gotchas and Best Practices

1. **Strong Passwords**: Enforce strong password policies.
2. **Principle of Least Privilege**: Grant users only the permissions they need.
3. **Regular Audits**: Regularly audit user accounts and permissions.
4. **SSH Key Authentication**: Use SSH keys instead of passwords where possible.
5. **Centralized Authentication**: Consider using LDAP or similar for centralized user management in larger environments.

### Interview Questions

1. Q: Explain the difference between authentication and authorization.
   A:
      - Authentication is the process of verifying the identity of a user or system. It answers the question "Who are you?" This typically involves providing credentials like a username and password.
      - Authorization is the process of determining what actions an authenticated user is allowed to perform. It answers the question "What are you allowed to do?" This is based on the user's permissions, roles, or group memberships.

2. Q: What is PAM and how does it enhance system security?
   A: PAM (Pluggable Authentication Modules) is a flexible mechanism for authenticating users in Linux. It enhances security by:
      - Allowing different authentication methods to be used without changing application code
      - Providing a way to stack multiple authentication mechanisms
      - Enabling centralized configuration of authentication policies
      - Supporting additional checks like time-based access restrictions or resource limits
      - Allowing for easy integration of new authentication technologies

3. Q: How would you set up two-factor authentication (2FA) for SSH logins?
   A: To set up 2FA for SSH logins:
      1. Install Google Authenticator PAM module: `sudo apt-get install libpam-google-authenticator`
      2. Run `google-authenticator` as the user to generate a secret key and QR code
      3. Edit `/etc/pam.d/sshd` to include the line: `auth required pam_google_authenticator.so`
      4. Edit `/etc/ssh/sshd_config` to set `ChallengeResponseAuthentication yes`
      5. Restart the SSH service: `sudo systemctl restart sshd`
      Now, users will need both their password and a time-based code from their authenticator app to log in.

4. Q: What is the purpose of the `/etc/shadow` file and how does it improve security compared to storing passwords in `/etc/passwd`?
   A: The `/etc/shadow` file stores encrypted user passwords and related information. It improves security by:
      - Restricting access to password hashes (only readable by root)
      - Storing additional password-related information (e.g., expiration date, last change date)
      - Allowing for the use of more secure hashing algorithms
      - Separating password information from other user account details
      This separation prevents non-privileged users from accessing password hashes, making it more difficult for attackers to attempt offline password cracking.

5. Q: Explain the concept of sudo and how it enhances system security.
   A: sudo (Superuser Do) is a program that allows users to run commands with the security privileges of another user (by default, the superuser). It enhances security by:
      - Allowing fine-grained control over who can run what commands with elevated privileges
      - Providing an audit trail of commands executed with elevated privileges
      - Reducing the need for users to log in as root or know the root password
      - Allowing temporary elevation of privileges without changing the user's environment
      - Supporting time-limited authorization
      sudo helps implement the principle of least privilege, where users only get the minimum privileges necessary to perform their tasks, reducing the risk of accidental or intentional system damage.

This content covers the essentials of Linux Security Modules (LSM) and User Authentication and Authorization, providing both theoretical knowledge and practical examples. It should give a comprehensive understanding of these critical aspects of Linux security.
```

This completes the content for Day 10, covering Security and Access Control in Linux in detail.