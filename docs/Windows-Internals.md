# Windows Internals

## Overview

Windows Internals refers to the core architecture and components that make the Windows operating system function. Understanding these internals is essential for SOC analysts because attackers frequently abuse legitimate Windows features to execute malware, maintain persistence, escalate privileges, and evade detection.

By understanding how Windows works under the hood, defenders can better identify abnormal behavior and investigate security incidents.

---

# Windows Architecture

Windows operates using two primary execution modes:

- User Mode
- Kernel Mode

These modes separate normal applications from the operating system's core components, improving security and system stability.

```
                 Windows Operating System

        +---------------------------------------+
        |              User Mode                |
        |---------------------------------------|
        | Chrome                               |
        | Microsoft Word                       |
        | PowerShell                           |
        | Command Prompt                       |
        | Visual Studio Code                   |
        +---------------------------------------+

                 System Call Boundary

        +---------------------------------------+
        |             Kernel Mode              |
        |---------------------------------------|
        | Windows Kernel                       |
        | Memory Manager                       |
        | Process Scheduler                    |
        | Device Drivers                       |
        | File System                          |
        +---------------------------------------+
```

---

# User Mode

User Mode is where normal applications execute. Programs running in User Mode have limited privileges and cannot directly access hardware or modify critical parts of the operating system.

If an application crashes in User Mode, Windows usually continues running because the crash is isolated from the operating system.

Examples include:

- Google Chrome
- Microsoft Edge
- Microsoft Word
- PowerShell
- Visual Studio Code
- Notepad

---

# Kernel Mode

Kernel Mode is the most privileged execution level in Windows.

Core operating system components execute here, including:

- Windows Kernel
- Device Drivers
- Memory Manager
- Process Scheduler
- Hardware Abstraction Layer (HAL)

Code running in Kernel Mode has unrestricted access to system memory and hardware resources.

Because of this, bugs or malicious code executing in Kernel Mode can crash the entire operating system or compromise the system completely.

---

# Why SOC Analysts Care

Many advanced attacks attempt to move from User Mode into Kernel Mode because Kernel Mode provides complete control over the operating system.

Examples include:

- Vulnerable drivers
- Rootkits
- Kernel exploits
- Driver-based malware

Understanding this distinction helps analysts recognize why some attacks are significantly more dangerous than others.
# Windows Processes

## What is a Process?

A process is an instance of a program that is currently running on a Windows system.

When a user opens an application, Windows creates a process and allocates the resources required for it to execute. These resources include memory, CPU time, handles, and threads.

For example:

- Opening Google Chrome creates one or more `chrome.exe` processes.
- Opening Notepad creates a `notepad.exe` process.
- Opening Visual Studio Code creates a `Code.exe` process.

Every running application on Windows exists as one or more processes.

---

## Process Components

Each process contains several important elements:

- **Process ID (PID):** A unique number assigned to every running process.
- **Parent Process ID (PPID):** The ID of the process that started the current process.
- **Threads:** Individual units of execution within a process.
- **Memory:** The RAM allocated to the process.
- **Handles:** References to system resources such as files, registry keys, and network connections.

---

## Parent and Child Processes

Most Windows processes are created by another process.

For example:

```
explorer.exe
│
├── cmd.exe
│      └── powershell.exe
│             └── python.exe
```

This relationship is known as a **process tree**.

SOC analysts use process trees to understand how programs were launched and whether the sequence of events appears legitimate.

---

## Why Processes Matter in Cybersecurity

Attackers rarely create entirely new software. Instead, they often abuse trusted Windows processes to execute malicious commands or hide their activity.

Common examples include:

- `powershell.exe`
- `cmd.exe`
- `rundll32.exe`
- `regsvr32.exe`
- `mshta.exe`
- `wscript.exe`
- `cscript.exe`

These programs are legitimate Windows utilities, but they are frequently used in malicious attacks. Analysts must therefore evaluate **how** they are used rather than assuming they are malicious simply because they are running.

---

## Information Collected About a Process

During an investigation, analysts commonly collect:

- Process Name
- Process ID (PID)
- Parent Process
- Command Line
- User Account
- Creation Time
- Executable Path
- Digital Signature
- Network Connections
- Child Processes

Together, this information helps reconstruct attacker activity and determine whether a process is legitimate or suspicious.
# What is a Process?

A process is an instance of a program that is currently executing in memory.

When a user launches an application, Windows creates a process and allocates the necessary resources required for execution. These resources include CPU time, memory, threads, handles, and access to system resources.

Examples include:

- Opening Notepad creates `notepad.exe`
- Opening Google Chrome creates one or more `chrome.exe` processes
- Opening Visual Studio Code creates `Code.exe`

Although a program exists as a file on disk, it becomes a process only when Windows loads and executes it.
## Process ID (PID)

Every process running on Windows is assigned a unique numerical identifier called a Process ID (PID).

The PID allows Windows and security tools to uniquely identify and manage running processes. Even if multiple instances of the same application are running, each process receives a different PID.

SOC analysts frequently use PIDs when correlating events across logs, identifying suspicious activity, and tracing process execution.
## Parent Process

Most Windows processes are created by another process.

The process that launches a new process is known as the **parent process**, while the newly created process is referred to as the **child process**.

Windows records this relationship using the Parent Process ID (PPID), allowing analysts to reconstruct how processes were launched during an investigation.
## Process Trees

A process tree is a hierarchical representation of the relationships between running processes. It shows how one process creates another, forming parent-child relationships.

Process trees help analysts visualize the sequence of process execution and identify suspicious behavior.

Example of a normal process tree:

```text
explorer.exe
    └── chrome.exe
```

Example of a suspicious process tree:

```text
WINWORD.EXE
    └── powershell.exe
            └── cmd.exe
                    └── malware.exe
```

Analyzing process trees allows SOC analysts to determine whether a process execution chain is expected or potentially malicious.
# Threads

## What is a Thread?

A thread is the smallest unit of execution within a process.

While a process provides the resources required to run a program, threads perform the actual work. A single process may contain one or multiple threads that execute tasks concurrently.

For example, a web browser can use separate threads to render web pages, download files, play videos, and respond to user input simultaneously.

---

## Why Threads Matter

Threads improve application performance by allowing multiple operations to occur at the same time within a single process.

Although users often interact with a single application, that application may be executing dozens or even hundreds of threads in the background.

Understanding threads helps analysts appreciate how Windows schedules work and why a single process can perform many different tasks simultaneously.
# Windows Services

## What is a Windows Service?

A Windows Service is a background process that performs specific functions for the operating system or installed applications.

Services typically start automatically during system boot and continue running without requiring user interaction.

Unlike regular applications, services operate in the background and are responsible for tasks such as networking, printing, security, updates, and hardware management.

Examples include:

- Windows Defender
- Windows Update
- Print Spooler
- DHCP Client
- DNS Client

Windows relies on services to provide essential functionality and maintain system stability.
## Service Control Manager (SCM)

The Service Control Manager (SCM) is a core Windows component responsible for managing Windows services.

Its responsibilities include:

- Starting services
- Stopping services
- Restarting services
- Monitoring service status
- Managing service startup configuration

Nearly every Windows service is controlled through the SCM.
## Service Startup Types

Each Windows service has a startup type that determines when it is executed.

| Startup Type | Description |
|--------------|-------------|
| Automatic | Starts during system boot. |
| Automatic (Delayed Start) | Starts shortly after boot to reduce startup load. |
| Manual | Starts only when required. |
| Disabled | Cannot be started until enabled. |

Understanding startup types helps analysts identify services that automatically execute when Windows starts.
## Services and Cybersecurity

Windows Services are frequently abused by attackers to establish persistence.

Common techniques include:

- Creating malicious services.
- Modifying legitimate services.
- Replacing service executables.
- Configuring services to start automatically.

SOC analysts monitor service creation and modification events because unauthorized changes may indicate malicious activity.
---

# Windows Registry

## What is the Windows Registry?

The Windows Registry is a hierarchical database used by Windows to store configuration settings for the operating system, hardware, installed applications, and user preferences.

Instead of storing configuration in separate files, Windows centralizes much of this information in the Registry, allowing the operating system and applications to retrieve settings quickly.

The Registry is critical to Windows operation, and incorrect modifications can cause applications or the operating system to malfunction.

---

## Registry Structure

The Windows Registry is organized into logical sections called **Registry Hives**.

The primary Registry Hives include:

| Registry Hive | Purpose |
|---------------|---------|
| **HKEY_CLASSES_ROOT (HKCR)** | Stores file associations and COM object information. |
| **HKEY_CURRENT_USER (HKCU)** | Stores settings specific to the currently logged-in user. |
| **HKEY_LOCAL_MACHINE (HKLM)** | Stores system-wide configuration settings. |
| **HKEY_USERS (HKU)** | Contains settings for all user profiles on the system. |
| **HKEY_CURRENT_CONFIG (HKCC)** | Stores information about the current hardware configuration. |

---

## Registry Keys and Values

The Registry is organized similarly to a file system.

```
HKEY_LOCAL_MACHINE
│
└── Software
      │
      └── Microsoft
              │
              └── Windows
```

- **Keys** are similar to folders.
- **Subkeys** are folders within other keys.
- **Values** store the actual configuration data.

This hierarchical structure makes it easier to organize and retrieve system settings.

---

## Why the Registry Matters

Windows relies heavily on the Registry for system operation.

The Registry stores information such as:

- Installed software
- Startup applications
- User preferences
- Network configuration
- Device settings
- Security policies
- Service configuration

Nearly every Windows component interacts with the Registry during normal operation.

---

# Registry and Cybersecurity

The Windows Registry is one of the most common locations abused by attackers.

Malware often modifies Registry keys to:

- Achieve persistence.
- Execute automatically during system startup.
- Disable security features.
- Hide malicious activity.
- Store configuration information.

Because of this, Registry modifications are frequently monitored by Endpoint Detection and Response (EDR) solutions and SIEM platforms.

---

## Common Registry Persistence Locations

Attackers frequently abuse the following Registry locations:

| Registry Path | Purpose |
|---------------|---------|
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` | Starts programs when the current user logs in. |
| `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` | Starts programs for all users during system startup. |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce` | Executes a program once after user logon. |
| `HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce` | Executes a program once during startup for all users. |

SOC analysts routinely inspect these locations during incident response because they are commonly used to maintain persistence.

---

# Key Takeaways

- The Windows Registry is a centralized configuration database.
- Registry information is organized into hives, keys, and values.
- Windows and applications rely heavily on the Registry.
- Attackers frequently abuse Registry keys to establish persistence.
- Monitoring Registry modifications is an important part of threat detection and incident response.
---

# Windows File System (NTFS)

## What is a File System?

A file system is the method an operating system uses to organize, store, retrieve, and manage files on a storage device.

Without a file system, Windows would not know where files are located or how they should be accessed.

Modern Windows systems primarily use the **New Technology File System (NTFS)**.

---

## NTFS (New Technology File System)

NTFS is the default file system used by modern Windows operating systems.

It provides advanced features that improve reliability, security, and performance compared to older file systems such as FAT32.

Key features of NTFS include:

- File and folder permissions
- Encryption (EFS)
- Compression
- Journaling
- Large file support
- Access Control Lists (ACLs)

These features make NTFS suitable for enterprise environments and modern security requirements.

---

## Important Windows Directories

Understanding common Windows directories helps SOC analysts quickly locate files during investigations.

| Directory | Purpose |
|-----------|---------|
| `C:\Windows` | Contains the Windows operating system files. |
| `C:\Windows\System32` | Stores core Windows executables, DLLs, and drivers. |
| `C:\Program Files` | Default installation directory for 64-bit applications. |
| `C:\Program Files (x86)` | Default installation directory for 32-bit applications. |
| `C:\Users` | Contains user profiles, Desktop, Documents, Downloads, and AppData. |
| `C:\Temp` | Temporary storage used by applications and installers. |

---

## Hidden and System Files

Windows marks certain files as **Hidden** or **System** to reduce the risk of accidental modification.

Attackers may abuse these attributes to conceal malicious files from users.

For this reason, investigators often enable the display of hidden files during forensic analysis.

---

## File Permissions

NTFS uses **Access Control Lists (ACLs)** to determine who can access files and folders.

Permissions include:

- Read
- Write
- Modify
- Execute
- Full Control

Proper file permissions help protect sensitive data from unauthorized access.

---

# File System and Cybersecurity

Attackers frequently abuse the Windows file system to store malware, scripts, and persistence mechanisms.

Common attacker behaviors include:

- Dropping malware into user-writable directories.
- Hiding files using Hidden or System attributes.
- Replacing legitimate executables.
- Storing malicious scripts in temporary folders.

SOC analysts examine file paths, timestamps, permissions, hashes, and digital signatures to determine whether a file is legitimate.

---

## Common Suspicious Locations

Although malware can exist anywhere, analysts often investigate files located in:

- `C:\Users\Public`
- `C:\Users\<Username>\AppData`
- `C:\ProgramData`
- `C:\Temp`
- `%TEMP%`
- `%APPDATA%`

Files executing from these locations are not always malicious, but they warrant additional investigation.

---

# Key Takeaways

- NTFS is the primary file system used by Windows.
- The file system organizes how data is stored and accessed.
- Important Windows directories frequently appear during investigations.
- NTFS permissions help secure files and folders.
- Attackers commonly abuse writable directories to store malware and maintain persistence.
---

# Why Windows Internals Matter to SOC Analysts

A strong understanding of Windows Internals allows SOC analysts to investigate security incidents more effectively by understanding how the operating system behaves under normal and malicious conditions.

During an investigation, analysts routinely examine:

- Running processes and process trees
- Parent-child process relationships
- Windows Services
- Registry modifications
- File system activity
- User accounts
- Network connections
- Windows Event Logs

Many attacker techniques rely on abusing legitimate Windows components rather than exploiting vulnerabilities. Understanding these components enables analysts to distinguish normal system behavior from malicious activity.

Examples include:

- PowerShell executing malicious scripts
- Attackers creating Windows Services for persistence
- Malware modifying Registry Run keys
- Suspicious processes spawning command shells
- Malware hiding within legitimate Windows directories

Knowledge of Windows Internals forms the foundation for malware analysis, digital forensics, threat hunting, detection engineering, and incident response.

---

# Summary

Throughout this document, we explored the core components that make the Windows operating system function.

Topics covered include:

- Windows Architecture
- User Mode vs Kernel Mode
- Windows Processes
- Process IDs (PID)
- Parent Processes
- Process Trees
- Threads
- Windows Services
- Service Control Manager (SCM)
- Service Startup Types
- Windows Registry
- Registry Hives
- Registry Persistence
- NTFS File System
- Windows Directory Structure
- File Permissions
- Hidden and System Files

Understanding these concepts provides the foundation required to analyze Windows systems, investigate security incidents, and understand how attackers abuse legitimate operating system functionality.

This knowledge will support future topics including Sysinternals tools, Sysmon, Windows Event Forwarding (WEF), Wazuh, Sigma rules, detection engineering, and threat hunting.