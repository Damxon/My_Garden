---
title: Network Foundations
draft: false
tags:
  - Basics
  - Networking
---
**SKILL ASSESSMENT**

`ip route get <ip adress>` -> *gives the route from your machine to target*

`ifconfig -a` -> *shows all the network connections* 
	lo : is basically between router 
	
`netstat -tvlp4 / -tvlnp4` ->  *n option shows network IP:PORT* 

netcat for FTP 

- Bypass the request filtering found on the target machine's HTTP service, and submit the flag found in the response. The flag will be in the format: HTB{...}
		-nc <target IP> 21
		- USER Anonymous [ctrl+V][enter][enter]
		- PASS a
		- PASV (son ikinci hane ile 256 carpilip son hane ile topla)
		- nc -v <IP> <islende cikan PORT numarasini yaz>
		- bu sekilde islemlere devam et
		- RETR <dokuman adi> [ctrl+V]
		- nc -v <IP> 80
		- GET / HTTP/1.1
		- Host: <IP>
		- User-Agent: ... 
	HTB{...} DONE.