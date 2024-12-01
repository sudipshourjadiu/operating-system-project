### **Project Report: Deploying a Secure Web Server Using Nginx**

#### **Project Overview**

This project focused on deploying a web server using **Nginx** to host multiple websites with virtual hosting. The server was configured to support secure connections via HTTPS using **SSL/TLS** certificates provided by **Certbot**. To enhance security, **firewalls** (using **ufw**) and **Fail2ban** were implemented to mitigate unauthorized access and brute-force attacks. During the implementation, some issues with Fail2ban were encountered and addressed.

---

### **Objectives**

1. Deploy a web server using **Nginx**.
2. Configure the server to host multiple websites using virtual hosting.
3. Enable HTTPS using **SSL/TLS** certificates generated by Certbot.
4. Implement security measures, including:
   - Configuring a firewall using **ufw**.
   - Setting up **Fail2ban** to automatically block IPs based on suspicious activities.
5. Identify and troubleshoot issues related to Fail2ban.

---

### **System Design and Configuration**

#### **1. Web Server Deployment with Nginx**

- **Installation**:
  Installed Nginx on the operating system:
  ```bash
  sudo apt update
  sudo apt install nginx
  ```

- **Virtual Hosting**:
  Configured Nginx to host multiple websites by creating separate configuration files in `/etc/nginx/sites-available/` and enabling them via symbolic links in `/etc/nginx/sites-enabled/`.  
  Example configuration for `example.com`:
  ```nginx
  server {
      listen 80;
      server_name example.com www.example.com;
      root /var/www/example.com;

      location / {
          index index.html;
      }
  }
  ```

- **Validation and Restart**:
  Verified the configurations and restarted Nginx:
  ```bash
  sudo nginx -t
  sudo systemctl restart nginx
  ```

---

#### **2. HTTPS Support with SSL/TLS**

- **Installation of Certbot**:
  Installed Certbot to generate free SSL certificates from Let's Encrypt:
  ```bash
  sudo apt install certbot python3-certbot-nginx
  ```

- **Generating Certificates**:
  Secured the website with SSL/TLS:
  ```bash
  sudo certbot --nginx -d example.com -d www.example.com
  ```

- **Automating Certificate Renewal**:
  Verified Certbot’s renewal process:
  ```bash
  sudo certbot renew --dry-run
  ```

- **Redirection**:
  Updated the Nginx configuration to redirect HTTP traffic to HTTPS:
  ```nginx
  server {
      listen 80;
      server_name example.com www.example.com;
      return 301 https://$host$request_uri;
  }
  ```

---

#### **3. Firewall Configuration with UFW**

- **Installing UFW**:
  Installed and enabled the uncomplicated firewall:
  ```bash
  sudo apt install ufw
  sudo ufw enable
  ```

- **Allowing Required Ports**:
  Allowed HTTP (port 80), HTTPS (port 443), and SSH (port 22) traffic:
  ```bash
  sudo ufw allow http
  sudo ufw allow https
  sudo ufw allow ssh
  ```

- **Verification**:
  Checked the active rules:
  ```bash
  sudo ufw status
  ```

---

#### **4. Security with Fail2ban**

- **Installation**:
  Installed Fail2ban for intrusion prevention:
  ```bash
  sudo apt install fail2ban
  ```

- **Custom Configuration for Nginx**:
  Configured Fail2ban to monitor Nginx logs for unauthorized access attempts:
  - Created a custom jail configuration in `/etc/fail2ban/jail.d/nginx-http-auth.conf`:
    ```ini
    [nginx-http-auth]
    enabled = true
    port = http,https
    filter = nginx-http-auth
    logpath = /var/log/nginx/access.log
    maxretry = 3
    bantime = 3600
    findtime = 600
    ```
  - Updated the Nginx filter in `/etc/fail2ban/filter.d/nginx-http-auth.conf` to block repeated HTTP 401 responses:
    ```ini
    [Definition]
    failregex = ^<HOST> - .* "(GET|POST|HEAD).*" 401
    ignoreregex =
    ```

- **Testing and Monitoring**:
  Tested Fail2ban with simulated log entries and verified bans:
  ```bash
  sudo fail2ban-regex /var/log/nginx/access.log /etc/fail2ban/filter.d/nginx-http-auth.conf
  sudo fail2ban-client status nginx-http-auth
  ```

---

### **Issues Encountered**

1. **Fail2ban Not Banning IPs**:
   - **Problem**:
     Fail2ban detected matching log entries but failed to ban IPs.
   - **Solution**:
     - Verified log permissions and ensured the `fail2ban` user could access `/var/log/nginx/access.log`.
     - Checked the `iptables` rules to ensure Fail2ban was updating them correctly.
     - Corrected the `banaction` in the jail configuration to:
       ```ini
       banaction = iptables-multiport
       ```

2. **Regex Misconfiguration**:
   - **Problem**:
     Initial regex for detecting HTTP 401 responses was overly complex and failed to match logs consistently.
   - **Solution**:
     Simplified the `failregex` pattern to match only HTTP 401 response logs.

3. **Firewall Integration**:
   - **Problem**:
     There was a lack of clarity about integrating Fail2ban with UFW.
   - **Solution**:
     Configured Fail2ban to use UFW for banning IPs by setting:
     ```ini
     banaction = ufw
     ```

---

### **Results**

1. **Hosting**:
   - The Nginx server successfully hosts multiple websites with virtual hosting.
   - HTTPS is enabled for all hosted websites using Let's Encrypt certificates.

2. **Security**:
   - UFW blocks all unnecessary traffic while allowing essential ports.
   - Fail2ban actively monitors access logs and bans IPs exhibiting unauthorized behavior, such as repeated HTTP 401 responses.

3. **Issue Resolution**:
   - All identified issues with Fail2ban were addressed, ensuring a functional and secure deployment.

---

### **Conclusion**

This project demonstrates the successful deployment of a secure web hosting environment using Nginx, Certbot, UFW, and Fail2ban. Despite initial challenges with Fail2ban, the issues were resolved through careful troubleshooting and configuration adjustments. This setup is now robust, scalable, and secure, making it suitable for hosting multiple websites in a production environment.

Let me know if you’d like further refinements or additional sections!
