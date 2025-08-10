
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
````

#### **Step 2: Compile & Install ModSecurity Core**

The ModSecurity WAF engine must be compiled and installed before it can be used by Nginx.

```bash
# Navigate to a source directory
cd /usr/local/src

# Clone the ModSecurity repository from GitHub
sudo git clone --depth 1 -b v3/master --single-branch [https://github.com/SpiderLabs/ModSecurity](https://github.com/SpiderLabs/ModSecurity)

# Navigate into the ModSecurity directory
cd ModSecurity

# Initialize and update git submodules (dependencies)
sudo git submodule init
sudo git submodule update

# Run the build script, configure, compile, and install
sudo ./build.sh
sudo ./configure
sudo make
sudo make install
```

#### **Step 3: Compile Nginx Connector Module**

Compile the dynamic connector module that links Nginx to the ModSecurity library. This module must match your installed Nginx version.

```bash
# Check your installed Nginx version
nginx -v
# Example Output: nginx version: nginx/1.26.3

# Download the exact matching Nginx source code
cd /usr/local/src
sudo wget [https://nginx.org/download/nginx-1.26.3.tar.gz](https://nginx.org/download/nginx-1.26.3.tar.gz)

# Extract the Nginx source
sudo tar -xvzf nginx-1.26.3.tar.gz

# Clone the ModSecurity-Nginx connector from GitHub
sudo git clone [https://github.com/SpiderLabs/ModSecurity-nginx.git](https://github.com/SpiderLabs/ModSecurity-nginx.git)

# Navigate into the Nginx source directory
cd nginx-1.26.3

# Configure Nginx to compile the connector as a dynamic module
sudo ./configure --with-compat --add-dynamic-module=../ModSecurity-nginx

# Compile only the dynamic modules
sudo make modules

# Find the Nginx modules path
nginx -V 2>&1 | grep "modules-path"
# Example Output: --modules-path=/usr/lib/nginx/modules

# Create the directory and copy the compiled module
sudo mkdir -p /usr/lib/nginx/modules/
sudo cp objs/ngx_http_modsecurity_module.so /usr/lib/nginx/modules/
```

### **3. Configuration**

#### **Step 4: Configure Nginx to Load the Module**

Tell Nginx to load the new dynamic module using an absolute path to prevent loading errors.

```bash
sudo nano /etc/nginx/modules-enabled/50-mod-http-modsecurity.conf
```

Add the following line to the file:
`load_module /usr/lib/nginx/modules/ngx_http_modsecurity_module.so;`

#### **Step 5: Configure ModSecurity & OWASP CRS**

Set up the ModSecurity configuration file and install the OWASP Core Rule Set (CRS).

```bash
# Copy recommended ModSecurity configuration files
sudo cp /usr/local/src/ModSecurity/modsecurity.conf-recommended /etc/nginx/modsecurity.conf
sudo cp /usr/local/src/ModSecurity/unicode.mapping /etc/nginx/

# Enable ModSecurity's blocking mode
sudo sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/nginx/modsecurity.conf

# Download and configure the OWASP CRS
cd /etc/nginx
sudo git clone [https://github.com/coreruleset/coreruleset.git](https://github.com/coreruleset/coreruleset.git)
cd coreruleset
sudo cp crs-setup.conf.example crs-setup.conf

# Include the CRS rules in the main ModSecurity configuration
sudo nano /etc/nginx/modsecurity.conf
```

Add these lines to the end of the file:

```
Include /etc/nginx/coreruleset/crs-setup.conf
Include /etc/nginx/coreruleset/rules/*.conf
```

#### **Step 6: Enable WAF in Nginx Server Block**

Activate ModSecurity for your default virtual host by adding the directives inside the `server` block.

```bash
sudo nano /etc/nginx/sites-enabled/default
```

Replace the entire content with this clean configuration:

```nginx
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	# Add ModSecurity directives here
	modsecurity on;
	modsecurity_rules_file /etc/nginx/modsecurity.conf;
	
	root /var/www/html;

	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		try_files $uri $uri/ =404;
	}
}
```

### **4. Verification and Testing**

1.  **Test Nginx Configuration:**

    ```bash
    sudo nginx -t
    ```

    You should see: `nginx: configuration file /etc/nginx/nginx.conf test is successful`.

2.  **Restart Nginx Service:**

    ```bash
    sudo systemctl restart nginx
    ```

3.  **Test for XSS, SQLi, and LFI:**
    Monitor the Nginx error log and use your browser to send these payloads. You should see a `403 Forbidden` page in the browser and a corresponding alert in the terminal logs.

    ```bash
    # Monitor the logs in a separate terminal
    sudo tail -f /var/log/nginx/error.log
    ```

      * **XSS Test:** `http://127.0.0.1/?q=<script>alert(1)</script>`
      * **SQLi Test:** `http://127.0.0.1/?id=1' or '1'='1`
      * **LFI Test:** `http://127.0.0.1/?file=../../../../etc/passwd`

### **5. Troubleshooting Common Issues**

  - **`dlopen() ... No such file or directory`:** The `load_module` directive is pointing to the wrong location. Ensure you are using the absolute path (`/usr/lib/nginx/modules/...`) as shown in **Step 4**.
  - **`unknown directive "You"`:** You have copied and pasted prose or comments into an Nginx configuration file. Review the file and remove or comment out any invalid lines.
  - **`Unable to connect` or `Active: inactive (dead)`:** The Nginx service has stopped. If `nginx -t` is successful, try `sudo systemctl start nginx`. If it still fails, check the logs with `sudo journalctl -xeu nginx.service`.
  - **`rules loaded: 0`:** The ModSecurity configuration file is not correctly including the OWASP Core Rule Set. Verify that the `Include` directives in `modsecurity.conf` are correct and that the `coreruleset` directory exists at the specified path.

