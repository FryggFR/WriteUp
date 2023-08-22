* NOT THE REAL WRITEUP, I JUST DO SOME TEST*
# ROOM : KIBA

## 1. Enumeration

First of all, i have to gather somes informations about this boxes.
I run nmap scan to get open port, version etc.

`nmap -sC -sV {ip}`

3 ports is open as you can see in the result :

```
RESULT 
```

Port 22 (SSH)
Port 80 (HTTP)
Port 5601 (KIBANA)

## 2. Research of exploit !
We try to scan the port 80 with gobuster but nothing interesting here. The port 5601 is Kibana. Kibana is a dashboard for Elasticsearch.

We found one CVE about kibana : ==CVE-2019-7609==

This CVE can lead to RCE (Remote Code Execution)

(En cours d'écriture..)

