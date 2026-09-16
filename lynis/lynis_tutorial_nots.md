# Linux Security Auditing with Lynis: Setup, Scan, and Hardening Guide

In this article, we will walk through how to scan our Linux system using Lynis and how to remediate the security vulnerabilities it detects.

---

## What is Lynis?

Lynis is an open-source **system auditing and security hardening** tool. It runs from the command line and analyzes your system's security configurations, services, users, and many other components, providing you with a detailed report.

## Installation

```bash
git clone https://github.com/CISOfy/lynis
cd lynis
sudo ./lynis audit system 
```

Note: Lynis requires sudo (root) privileges to read system settings and files.

You can also scan Docker containers, not just your host system:

```bash
sudo lynis audit docker_file_name
```

Common Parameters

<img width="772" height="606" alt="Ekran Görüntüsü 2026-09-16 19-51-37" src="https://github.com/user-attachments/assets/d9b27499-f07a-455d-a7e3-aac18a093ff8" />

Creating a Custom Profile
Before scanning, Lynis creates a custom profile. This allows you to tailor the security scan to your system's role.

For example, Lynis's default profile is written for a general-purpose operating system. However, your system could be a web server, database server, or personal computer. With profiles, you can tell Lynis: "I intentionally left this port or service open."

Benefits:

Avoid unnecessary warnings.

Reduce scan time (scan only relevant services instead of everything).

Running the Scan
```bash
sudo lynis audit system
```
The scan takes a few minutes. Learning to read the output is key to using this tool effectively.

## Reading the Scan Output
Initialization: OS is detected, profiles are checked, language and location are determined.

System Information: Displays OS and program versions.

Tool Scanning: Checks available tools and system binaries.

Plugin 1: Contains comprehensive tests. Checks boot, services, kernel, memory, processes, users, groups, authentication, Kerberos, shells, file systems, USB devices, and storage.

Plugin 2: Usually not needed. It only activates if additional plugins are required.

Tip: You can develop and use your own custom plugin.
<img width="696" height="440" alt="image" src="https://github.com/user-attachments/assets/8419bada-7cb7-4c39-b6fc-b5940cf9d9ae" />


## Interpreting the Results
At the bottom of the report, you'll find the Result section. This contains warnings and suggestions.

### Example warning:
[WARNING] Firewall is not installed

Suggestions are provided right below. Reading and researching these suggestions is the most important step in improving your system security.

<img width="769" height="769" alt="image" src="https://github.com/user-attachments/assets/b5f864df-a012-4562-a977-494f04e85e21" />

## Hardening Index
In the Details section, you'll find the Hardening Index. This scores your security out of 100.

On a first scan, you'll typically get a score between 65-75.

You can increase this score by applying the suggestions.

## Software Components
Right below, Lynis checks whether the following components are installed:

Firewall

Malware Scanner

Intrusion Detection/Prevention Software

If you don't have a firewall, make sure to install one. It's critical for packet filtering.

Conclusion
### Lynis is a powerful tool for quickly auditing your Linux systems and remediating security vulnerabilities. By regularly monitoring the Hardening Index and suggestions, you can continuously improve your system security.
