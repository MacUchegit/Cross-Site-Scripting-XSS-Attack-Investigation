![Untitled](https://github.com/user-attachments/assets/4d3d2eba-c800-4ad3-8909-8e6e85ad2bf0)
----

# 🚨 Detecting and Investigating a Cross-Site Scripting (XSS) Attack with Let’s Defend SIEM

## Introduction

Cybersecurity threats are like digital landmines—silent, sneaky, and sometimes devastating. One such crafty attacker’s favorite is the XSS (Cross-Site Scripting) attack. Today, I'm taking you behind the scenes of a real-world investigation I conducted using the Let’s Defend SIEM platform to detect and analyze a potential XSS attack.  

Whether you're a cybersecurity enthusiast or just curious about how these things work, buckle up—this will be an insightful ride!  

### 🔍 So, What Is an XSS Attack?  

![1744810379019](https://github.com/user-attachments/assets/963f70b7-44ce-41f0-805e-e2db2a4971b5)

Cross-Site Scripting (XSS) is a type of injection attack where an attacker injects malicious JavaScript code into web pages that other users view. If the site isn’t properly sanitizing user input, that code runs in the user's browser—stealing cookies, session tokens, or even redirecting them to fake login pages.  

Think of it like someone writing malicious code into a comment box on a site—and that code hijacks your browser just because you happened to read it. Nasty stuff.  

### 🧭 The Alert That Started It All  
My investigation kicked off with a suspicious alert from the Let’s Defend SIEM dashboard:  

![1744980927253](https://github.com/user-attachments/assets/b0cd5318-2714-4a4d-8799-2e5b1d07e5e0)

```
Event ID: 116  
Event Time: Feb 26, 2022, 06:56 PM  
Rule Triggered: SOC166 - JavaScript Code Detected in Requested URL  
Hostname: WebServer1002  
Source IP: 112.85.42.13  
Destination IP: 172.16.17.17  
HTTP Method: GET  
User-Agent: Mozilla Firefox 40.1  
Requested URL:  
Alert Trigger Reason: JavaScript code detected in URL  
Device Action: Allowed 😬  
```  

Immediately, alarm bells were ringing. Why would someone put raw JavaScript into a search query unless they were testing for vulnerabilities?  

### 🧩 Step 1: Understanding the Trigger  

![1744808730529](https://github.com/user-attachments/assets/cbe1c7e1-a10f-401c-8c3a-8f90caa31fd4)

Following the incident response playbook, I started by answering one important question:  

**Why was this alert triggered?**  

The rule name, *JavaScript Code Detected in Requested URL*, suggests someone tried injecting code—very typical of an XSS probe. The suspicious part of the URL was this:  

```
?q=<$script>javascript:$alert(1)<$/script>  
```  

This is a textbook attempt to run JavaScript in the browser. `alert(1)` is a common payload used to test for XSS vulnerabilities. When the browser executes it, a pop-up appears—signaling that the injection worked.  

Based on this, I suspected a reflected XSS attack was in play.  

### 🌐 Step 2: Diving into the HTTP Traffic  

![1744809101129](https://github.com/user-attachments/assets/1ec1f3cb-7d20-4850-ae6a-94a305058dce)

Next stop: the Log Management tab.  

I filtered logs using the attacker’s IP address: `112.85.42.13`. Sure enough, I discovered more suspicious requests:  

![1744809214097](https://github.com/user-attachments/assets/a97d44f2-ed28-4466-ae68-3f05d86ef431)

```
Request URL:  
https://172.16.17.17/search/?q=<$script>$for((i)in(self))eval(i)(1)<$/script>  
```  

💡 **Breaking It Down (For Everyone):** This isn’t just gibberish—it’s JavaScript designed to run inside a browser. The attacker is trying to use `eval()`, a dangerous function that executes code strings. If this gets executed, the attacker could run anything they want on the victim’s browser.  

That’s not just shady—it’s full-blown malicious.  

### ⚖️ Step 3: Is the Traffic Malicious?  

![1744809251924](https://github.com/user-attachments/assets/fc6820ee-bd4a-4e1f-abda-02964e39da12)

**Absolutely.**  

Everything points to this being a malicious probe. Multiple crafted JavaScript payloads were sent in the URLs—definitely not regular user behavior. This wasn't a misclick or an over-enthusiastic developer testing their code. It was a deliberate attempt to exploit a weakness.  

### 🎯 Step 4: Identifying the Type of Attack  

![1744809303106](https://github.com/user-attachments/assets/9fcaedc0-0444-4f83-9462-8334667b0ed0)

At this point, it's crystal clear: This was a Cross-Site Scripting (XSS) attempt. The attacker was injecting JavaScript into a search query URL to see if the site would blindly execute it.  

### � Step 5: Could It Be a Planned Simulation?  

![1744809359189](https://github.com/user-attachments/assets/d7a02029-ae63-4849-8676-ac93e8724f9e)

Sometimes, alerts come from internal red-team tests or automated simulation tools like Verodin or AttackIQ. To rule that out, I searched the internal mailbox using indicators like the hostname, IP, and user-agent.  

**Result?** ❌ No signs of a simulation. This was not a drill.  

### ↔️ Step 6: What's the Direction of Traffic?  

![1744809390793](https://github.com/user-attachments/assets/4350bbad-1819-4dde-b4c5-3666b52dee53)

Analyzing the source IP (`112.85.42.13`) revealed that it was external. That means someone outside the organization was trying to interact with an internal web server—classic setup for reconnaissance or attack attempts.  

### ✅ Step 7: Was the Attack Successful?  

![1744809418109](https://github.com/user-attachments/assets/b63ac36d-3ad0-4f33-95c0-2cfd236a8133)

I returned to the logs and dug deeper into the HTTP responses. I noticed a consistent pattern of this status code:  

```
HTTP Status: 302 Found  
```  
![1744809507722](https://github.com/user-attachments/assets/98ac87f2-ddfc-4c49-bdcf-0a3977e52e48)

This is a redirect response—it means the request was received, but the server told the client to look elsewhere.  

🛡️ **Why That’s a Good Thing:** If the attack had succeeded, we’d likely see a `200 OK` response, meaning the server accepted and possibly executed the malicious code. But a `302`? That’s more like a polite “Nope, not today.”  

So, no compromise. The attack was not successful. 🎉  

### 📁 Step 8: Recording the Artifacts  

![1744809822356](https://github.com/user-attachments/assets/cbbdf0c9-0de4-4e4b-9101-f303bc825362)

Recording artifacts is crucial. Every piece of evidence—including the malicious IP address `112.85.42.13`, the payloads, user-agent string, and timestamp—was carefully documented.  

**Why does this matter?**  
- ✅ Helps with threat intelligence  
- ✅ Assists in future investigations  
- ✅ Provides proof in audits or legal matters  

Good documentation = smarter defense. This way, if this IP resurfaces in later incidents, we’ve already got a head start.  

### 📈 Step 9: Tier 2 Escalation? Not Needed  

![1744809851580](https://github.com/user-attachments/assets/ce73d789-76dc-4153-bf1c-875678103150)

Since the attack was unsuccessful and the traffic didn’t breach any sensitive systems or execute any code, escalation wasn’t necessary. But we documented everything—just in case there’s a pattern later.  

### 📝 Step 10: Final Notes and Closing the Case  
I left a detailed analysis note in the system, summarizing the investigation, and then closed the alert as a **True Positive**. It was a real threat—but thankfully, it didn’t succeed.  

![1744809884834](https://github.com/user-attachments/assets/7e2d811c-e74d-4838-87ac-2ebff54b8f46)

### 🧠 Final Thoughts: Why XSS Attacks Are a Big Deal  
XSS attacks are often underestimated because they seem “simple.” But don’t be fooled.  

A successful XSS attack can:  
- Steal user credentials or session cookies  
- Perform actions on behalf of the victim  
- Redirect users to phishing or malware pages  
- Deface your site  

They exploit trust—trust between the user and the web application. And trust, once broken, is hard to win back.  

### 💡 Key Takeaway  
Always sanitize user input. Always validate output. And always, always monitor for sketchy behavior like raw `<script>` tags in your URLs.  
