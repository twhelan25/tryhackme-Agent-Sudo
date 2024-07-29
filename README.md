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
Command explanation:
-A let's use specify the user agent "R"
-L will follow redirects

We get a different response:

![agent R response](https://github.com/user-attachments/assets/a9312aa2-68ff-464e-a19b-d852219dc0e5)

This reponse gives us the hint that since there are 25 user-agents, and 26 letters in the alphabet, and agent R is one of them, that the user agent is most likely one of the other letters. Let's try the same command replacing "R" with "A", "B" and so on.

We get the same result until we try user-agent "C":

![agent C ](https://github.com/user-attachments/assets/802c4f48-6b83-4045-af41-c9a0f1ca7ae3)

This gives us the answer to question about the user name, chris. It also alerts us to the fact that agent C has a weak password, so we should attempt a dictionary attack with hydra and the wordlist rockyou.txt. Looking back our nmap scan, the password will most likely be for ftp or ssh. I attempted anonymous login to ftp: ftp $ip, but it prompted for a password, so let's try the password attack on the ftp server:
```bash
hydra -l chris -P /usr/share/wordlists/rockyou.txt $ip ftp
```
After about a minute, I hydra revealed the ftp password. Let's go ahead and log into the ftp server as chris:

![ftp](https://github.com/user-attachments/assets/680a6191-3213-4c78-8a7a-516c51a7aa33)

We're in! Let's use the get command to get this image files onto our machine:
```bash
get <file name>
```
First, let's cat To_agentJ.txt:


![To_agentJ txt](https://github.com/user-attachments/assets/ae98bb71-b5fa-4334-8ece-d8915c72bcaf)
Ok, I tried exiftool and strings on cute-alien.jpg but didn't yield anything. Then I tried steghide, but it prompts for a password, so let's attack it with stegcracker:

![stegcracker](https://github.com/user-attachments/assets/e9a29a66-dd5d-450b-8d44-23411b21025f)
