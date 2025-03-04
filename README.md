# web-attacks
Web attacks are malicious activities targeting websites, web applications, and online services. These attacks exploit vulnerabilities in web technologies to steal data, disrupt services, or gain unauthorized access. Below are common types of web attacks and the tools used for them:

---

### **1. SQL Injection (SQLi)**
- **Description:** Attackers insert malicious SQL queries into input fields to manipulate a database. This can allow them to retrieve, modify, or delete data.
- **Tools Used:**
  - SQLmap
  - Havij
  - SQLNinja
  - jSQL Injection  

---

### **2. Cross-Site Scripting (XSS)**
- **Description:** Attackers inject malicious scripts into web pages that execute in users’ browsers. This can steal cookies, session tokens, or redirect users to malicious sites.
- **Tools Used:**
  - XSSer
  - BeEF (Browser Exploitation Framework)
  - XSStrike  

---

### **3. Cross-Site Request Forgery (CSRF)**
- **Description:** Forces a victim’s browser to execute unauthorized actions on a trusted site without their consent.
- **Tools Used:**
  - CSRF PoC Generator
  - Burp Suite  

---

### **4. Distributed Denial of Service (DDoS)**
- **Description:** Attackers flood a website with excessive traffic, overwhelming the server and making the site unavailable.
- **Tools Used:**
  - LOIC (Low Orbit Ion Cannon)
  - HOIC (High Orbit Ion Cannon)
  - Slowloris  
  - MHDDoS  

---

### **5. Man-in-the-Middle (MitM) Attack**
- **Description:** Attackers intercept communication between users and web services to steal sensitive data or inject malicious content.
- **Tools Used:**
  - Ettercap
  - Wireshark
  - Bettercap  

---

### **6. Command Injection**
- **Description:** Attackers inject system commands through vulnerable input fields, gaining control over the web server.
- **Tools Used:**
  - Commix
  - Metasploit  

---

### **7. Directory Traversal**
- **Description:** Exploits improper access control to navigate a website’s directories and access restricted files.
- **Tools Used:**
  - DirBuster
  - Gobuster
  - WFuzz  

---

### **8. Security Misconfiguration Exploitation**
- **Description:** Attackers exploit improperly configured security settings, such as default credentials, exposed admin panels, or weak permissions.
- **Tools Used:**
  - Nikto
  - Wappalyzer
  - Metasploit  

---

### **9. Zero-Day Exploits**
- **Description:** Exploits unknown vulnerabilities before they are patched.
- **Tools Used:**
  - ExploitDB
  - Metasploit  

---

### **10. Phishing Attacks**
- **Description:** Attackers trick users into providing sensitive information by impersonating legitimate websites.
- **Tools Used:**
  - Social-Engineer Toolkit (SET)
  - Evilginx  

---

### **Prevention & Defense**
- Web Application Firewalls (WAF) (e.g., ModSecurity, Cloudflare)
- Input Validation & Sanitization
- Security Headers (e.g., CSP, X-XSS-Protection)
- HTTPS & Secure Authentication Practices  

Would you like details on specific attack types or tools?
