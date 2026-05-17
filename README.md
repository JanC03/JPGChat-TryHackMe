<h1>Penetration Test Write-Up: JPChat (TryHackMe)</h1>

<h2>Description</h2>
Room: JPChat (TryHackMe)
<br />
Difficulty: Easy
<br />
Objective: Exploit a poorly coded chatbot to achieve remote code execution and escalate to root.
<br />
Real-World Framing: The vulnerabilities demonstrated mirror those commonly introduced by AI-generated code deployed without human review.
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

AI writes functional code. It does not write secure code.
When a developer uses tools like ChatGPT or Copilot to build a feature quickly, the code often works perfectly—but contains mistakes that a human reviewer would catch. The vulnerability exploited here is a textbook example of what AI assistants consistently get wrong.
<br />
<br />
Every internet-facing service is a front door.
This was just a chatbot. It wasn't the payment system or the admin panel. But any service accessible from the internet is an attack surface. A single vulnerable feature on your website can be the entry point for a full compromise.
<br />
<br />
One small mistake can give an attacker everything.
The initial vulnerability gave us a restricted shell on the server. A second, completely unrelated misconfiguration let us become the system administrator. Attackers chain small weaknesses together. Defence in depth is the only real protection.
<br />
<br />
You don't need to be a target to be a victim.
Automated bots scan the internet constantly, looking for exactly these vulnerabilities. Nobody specifically targeted this system. A generic exploit worked against a generic mistake. If your business has a website, it is being scanned right now.

<h2>Program walk-through:</h2>

<h2>Technical walk-through:</h2>
<p align="center">
Launch the utility: <br/>
<img src="https://i.imgur.com/62TgaWL.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Select the disk:  <br/>
<img src="https://i.imgur.com/tcTyMUE.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Enter the number of passes: <br/>
<img src="https://i.imgur.com/nCIbXbg.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Confirm your selection:  <br/>
<img src="https://i.imgur.com/cdFHBiU.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Wait for process to complete (may take some time):  <br/>
<img src="https://i.imgur.com/JL945Ga.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Sanitization complete:  <br/>
<img src="https://i.imgur.com/K71yaM2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Observe the wiped disk:  <br/>
<img src="https://i.imgur.com/AeZkvFQ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<h2>Disclaimer</h2>
This penetration test was performed against a deliberately vulnerable training environment (TryHackMe, JPChat room). All findings and exploitation were conducted in a controlled, authorised setting. No real systems were targeted, and no unauthorised access was attempted. The real-world framing is for educational and portfolio demonstration purposes only.
<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
