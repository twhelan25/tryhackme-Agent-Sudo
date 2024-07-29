![intro](https://github.com/user-attachments/assets/f0320ee6-9e24-44cf-bf4d-98c32338a829)

# tryhackme-Agent-Sudo
This is a walkthrough for the tryhackme CTF Agent Sudo. I will not provide any flags or passwords as this is intended to be used as a guide.

## Scanning

First off, let's store the target IP as a variable for easy access.

Command: export ip=xx.xx.xx.xx

Next, let's run an nmap scan on the target IP:
```bash
nmap -A -v $ip -D RND:10 -oN nmap.txt
```

Command break down:

-A: This flag enables aggressive scanning. It combines various scan types (like OS detection, version detection, script scanning, and traceroute) into a single scan.

-v: increases verbosity, providing more detailed output during the scan.

—$ip: provides the target IP we stored as the variable $ip.

-D RND:10 Nmap can send additional packets to confuse network intrusion detection systems (IDS) or hide the true source of the scan by randomly selecting up to 10 decoy IP addresses.

-oN nmap.txt: This option specifies normal output that should be saved to a file named “nmap.txt.

This scan reveals three open ports:

![nmap](https://github.com/user-attachments/assets/eccc0012-bd2f-408e-99c1-5a3f32620794)

First, let's investigate the websever on port 80.
We are greeted with a message from Agent R:

![agent R](https://github.com/user-attachments/assets/6277fcae-9082-4506-a524-d4d8036d2b0c)

The User-Agent is a crucial part of the HTTP headers sent with every request. It helps servers identify the application, operating system, vendor, and version of the requesting user agent. In this case we can assume that agent R is one of the code names. Let's try spoofing as user-agent R on this url. 
```bash
curl -A "R" -L $ip
```



