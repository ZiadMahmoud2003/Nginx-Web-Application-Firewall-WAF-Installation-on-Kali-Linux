# Nginx with ModSecurity WAF on Kali Linux

This repository contains a comprehensive, step-by-step guide to installing and configuring Nginx with the ModSecurity Web Application Firewall (WAF) on a Kali Linux environment. The setup utilizes the OWASP Core Rule Set (CRS) to provide robust protection against common web attacks such as Cross-Site Scripting (XSS), SQL Injection (SQLi), and Local File Inclusion (LFI).

The guide is based on compiling ModSecurity and its Nginx connector from source, ensuring compatibility and modularity with the Nginx web server.

---

### **Table of Contents**

1.  [Prerequisites](#prerequisites)
2.  [Installation Guide](#installation-guide)
    - [Step 1: Install Dependencies](#step-1-install-dependencies)
    - [Step 2: Compile & Install ModSecurity Core](#step-2-compile--install-modsecurity-core)
    - [Step 3: Compile Nginx Connector Module](#step-3-compile-nginx-connector-module)
3.  [Configuration](#configuration)
    - [Step 4: Configure Nginx to Load the Module](#step-4-configure-nginx-to-load-the-module)
    - [Step 5: Configure ModSecurity & OWASP CRS](#step-5-configure-modsecurity--owasp-crs)
    - [Step 6: Enable WAF in Nginx Server Block](#step-6-enable-waf-in-nginx-server-block)
4.  [Verification and Testing](#verification-and-testing)
5.  [Troubleshooting Common Issues](#troubleshooting-common-issues)

---

### **1. Prerequisites**

-   A Kali Linux system (or any Debian-based system).
-   Nginx installed and running.
-   `sudo` privileges.

### **2. Installation Guide**

#### **Step 1: Install Dependencies**

Update your system and install all the necessary build tools and libraries required for compiling the software from source.

```bash
sudo apt update
sudo apt install -y build-essential git libcurl4-openssl-dev libpcre3-dev libxml2-dev zlib1g-dev libssl-dev automake autoconf libtool libyajl-dev pkg-config libpcre2-dev
