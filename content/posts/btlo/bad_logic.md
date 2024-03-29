---
title: Bad Logic
url: /writeups/btlo/bad_logic
categories: ["Writeups"]
tags: ["BTLO", "Digital Forensics"]
date: 2021-06-05
summary: "Our system administrators have lost access to an internet facing Windows Server, Managed Defence believe our systems has been compromised and have collected some assets and left them on the Desktop - Please help them!"
featuredImage: images/btlo/official/nhRUjTBiOmnf7PZGsnMR.png
featuredImagePreview: false
lightgallery: true
---

## Walkthrough

From the initial **README.txt**, we learn that KAPE has been run, which is used to find the most interesting artefacts for forensics investigation. A memory dump and a pcap are also available.

### Pcap analysis

The capture is very big and takes some time to load. From **Statistics > Capture File Properties**, we see the capture was run during 7 days and 15h, which explains the big file size:  

{{< image src="images/btlo/bad_logic/ffa1c96a695c2107f650efc46e67bd29.png" caption="Packet capture properties" >}}

#### HTTP traffic

When reviewing HTTP requests, we notice some weird ones from the IP **178.62.72[.]123**, like this one:  

{{< image src="images/btlo/bad_logic/f4ffe7360c117384b86533e6b1a06159.png" caption="First malicious IP" >}}

It appears that an attacker is exploiting a vulnerability on port **7001** on the host **172.31.4.99** to perform some command execution, in this case a ping to **advertyzing.co[.]uk**.

According to the user agent, the request is sent from python-requests 2.20.

To better understand the request, we can URL decode it twice using **Cyberchef**:

{{< image src="images/btlo/bad_logic/f4fe8c5c9d8f5e19c99a6c91094eb6a0.png" caption="Cyberchef results" >}}


{{< admonition type=question title="What is the malicious domain used by the threat actor?" open=false >}}
**advertyzing.co[.]uk**
{{< /admonition >}}

By searching the GET request in a search engine, we find some articles about a vulnerability for Oracle WebLogic (CVE-2020-14882).

{{< admonition type=question title="Which application did the threat actor exploit, what port does this run on and which CVE did the threat actor utilise?" open=false >}}
**WebLogic,7001,CVE-2020-14882**
{{< /admonition >}}

After the ping, the vulnerability is used again (the attacker must have received the ping on the domain **advertyzing.co[.]uk** confirming that the code execution worked) to download **nc.exe** using **certutil.exe**, a common [LOLBins](https://lolbas-project.github.io/#).

{{< admonition type=question title="The threat actor has made good use of ‘Living off the land’ binaries (LOLBins). Which windows executable did they use to download a malicious file from their server?" open=false >}}
**Certutil.exe**
{{< /admonition >}}

{{< admonition type=question title="What was the name of the malicious file they downloaded using this windows executable?" open=false >}}
**nc.exe**
{{< /admonition >}}

We also find a request for a ncat reverse shell which executes powershell:

{{< image src="images/btlo/bad_logic/b7fa1fb1318ac476028dc50a71bf1871.png" caption="Reverse shell" >}}

We could start by reviewing powershell events retrieved from the KAPE analysis.

By further investigating HTTP requests, the IP **95.181.232.7** is also interacting with the weblogic website a few hours after the initial compromission:

{{< image src="images/btlo/bad_logic/f91e51059e6853e4635ac590b02935e8.png" caption="Second malicious IP" >}}

{{< admonition type=question title="Confirm the two IP addresses utilized by the threat actor:" open=false >}}
**178.62.72.123,95.181.232.7**
{{< /admonition >}}

If we review wireshark statistics, those two IPs are among those which interacted a lot with the victim IP.

The IP **178.62.72.123** is the top one which received packets from the victim:

{{< image src="images/btlo/bad_logic/4cd96a6f5d35c7ceb4f150aa331d01aa.png" caption="Network conversations - Packets received" >}}

Whereas the IP **95.181.232.7** is the second one which sent most packets to the victim:

{{< image src="images/btlo/bad_logic/0aa1a15650565d8bfc699844bad2b690.png" caption="Network conversations - Packets sent" >}}

### KAPE analysis

The two following directories are related to **KAPE**:  
- Module_Options: results of programs (for example **JLECmd** for jump lists) that have been configured to be run when the tool was executed
- Target_Options: Directory where files and directories are copied based on those specified to be recovered when the tool was executed

As we noticed a reverse shell serving a powershell command line, let's start by reviewing powershell's artefacts.

#### Powershell history

First let's review the PSRealine command history as mentioned in this [article](https://0xdf.gitlab.io/2018/11/08/powershell-history-file.html).

In the administrator's history, we get some interesting command, the tool [LaZagne](https://github.com/AlessandroZ/LaZagne) is used to dump credentials:

{{< image src="images/btlo/bad_logic/43eb315eb7bc9e11f724ddef2715597c.png" caption="Powershell history" >}}

{{< admonition type=question title="What is the name of the password dumping tool used by the threat actor?" open=false >}}
**laZagne**
{{< /admonition >}}

{{< admonition type=question title="What is the name of the text file the TA echo'ed out to?" open=false >}}
**password_extract.txt**
{{< /admonition >}}

#### KAPE's live response

By reviewing artefacts found by KAPE in `<C:\Users\BTLOTest\Desktop\MD-Artefacts\Module_Options\LiveResponse>` directory, we can see dns entries to subdomains related to **kryptex[.]org**, which is a cryptocurrency miner:

{{< image src="images/btlo/bad_logic/79327895f0d935dc9d3864a40a0da777.png" caption="DNS cache" >}}

In the same directory, we also find scheduled tasks related to this software:

{{< image src="images/btlo/bad_logic/38a79c174ff7da4c4ecdee41692d21a2.png" caption="Scheduled tasks" >}}

{{< admonition type=question title="The threat actor has used an off-the-shelf cryptominer, what is the name of the executable?" open=false >}}
**kryptex.exe**
{{< /admonition >}}

We also confirm the IP of the victim from the **ipconfig.txt** file:

{{< image src="images/btlo/bad_logic/3ac81ef223e6d60921d952002979239a.png" caption="Ipconfig" >}}

### Host artefacts

We were able to access artefacts on the host itself because the investigation is done on the compromised host, which won't likely be the case in a real world investigation. As those files were not recovered, we could have extracted them from the memory dump or find other artefacts that would point to them.

#### Kryptex

In the configuration file for the cryptocurrency software installed in `<C:\Users\Administrator\AppData\Roaming\Kryptex>`, we can find an email address which is most likely linked to the threat actor:

{{< image src="images/btlo/bad_logic/3063464feb2e8d27a4c8b0d387555be4.png" caption="Kryptex configuration file" >}}

#### Oracle Web Logic

According to the pcap analysis, the exploited service is **WebLogic**, we can review the related files to see if any legitimate files were used by the attacker.

By browsing the related directories in we fin in `<C:\Oracle\Middleware\Oracle_Home\user_projects\domains\base_domain\bin>` files that have been modified at **3/4/2021**, the same date of the attack from the pcap and a different date than other files, which were all modified the last time the **3/1/2021**:

{{< image src="images/btlo/bad_logic/fea23fff44f529a7159138aa6dce27c0.png" caption="Oracle Web Logic modified files" >}}

According to Oracle's [documentation](https://docs.oracle.com/cd/E13222_01/wls/docs90/server_start/overview.html), the script **startWebLogic.cmd** is the startup script for the WebLogic server.

It appears that the attacker added a call to **StartWebLogic.bat** which then executes **StartWebLogic.exe** with parameters similar to **ncat** (these are the only files with a **.bat** and **.exe** extension in the directory which is also suspicious):

{{< image src="images/btlo/bad_logic/99788a541525421b8479b1c525e1a34a.png" caption="startWebLogic.cmd" >}}

{{< image src="images/btlo/bad_logic/ab85dc93d5622bebcdec70e78c3d990a.png" caption="StartWebLogic.bat" >}}

If we get the hash of the executable and search it in [VirusTotal](https://www.virustotal.com/gui/file/e8fbec25db4f9d95b5e8f41cca51a4b32be8674a4dea7a45b6f7aeb22dbc38db/details), the file is recognized as **nc.exe**.

{{< admonition type=question title="The threat actor has attempted an unusual way of persisting by editing a key file. Which configuration file have they altered?" open=false >}}
**startWebLogic.cmd**
{{< /admonition >}}

### Memory dump analysis

Unfortunately, as specified in the **README.txt**, the installed version of **Volatility** won't work because the dump is too big.

We can  perform some strings analysis on it, but it is more difficult to get the context of the data returned.
As the search takes some time, it's better to do it when you already have some information and you can use specific keywords to have better results. 

For example, if we perform a search for "certutil", as we know it was used as a LOLbin:

{{< image src="images/btlo/bad_logic/b500cfdbfc059c944b9d9ccfe0973c09.png" caption="Certutil string occurences" >}}

We find again that it was used to download **nc.exe**, but we also see other domain that could be good to further investigate:

{{< image src="images/btlo/bad_logic/d4a85fa41e82691c439a278099406220.png" caption="Suspicious domain 1" >}}

{{< image src="images/btlo/bad_logic/fca557e71827d6b2f4ae7c2a2bebb0c4.png" caption="Suspicious domain 2" >}}

## Final Notes

A lot of tools are available for this investigation and I only detailed places where I found interesting information.

For example I searched specific event IDs in logs at: `<C:\Users\BTLOTest\Desktop\MD-Artefacts\Target_Options\C\Windows\System32\winevt\logs>`, reviewed jump lists at: `<C:\Users\BTLOTest\Desktop\MD-Artefacts\Target_Options\C\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations>`, reviewed registry hives and many other places.

This investigation was a very good learning experience as you have so much places to look into and many tools to try. 

This is one of the lab I like to come back to try new things, and if I find new interesting stuff, I will update this write up accordingly.

I hope you enjoyed the read !

## Resources

- <https://www.sans.org/posters/eric-zimmermans-results-in-seconds-at-the-command-line-poster>

- <https://www.sans.org/posters/windows-forensic-analysis>

- <https://lolbas-project.github.io>

- <https://0xdf.gitlab.io/2018/11/08/powershell-history-file.html>

- <https://github.com/AlessandroZ/LaZagne>

- <https://docs.oracle.com/cd/E13222_01/wls/docs90/server_start/overview.html>

- <https://www.virustotal.com/gui/file/e8fbec25db4f9d95b5e8f41cca51a4b32be8674a4dea7a45b6f7aeb22dbc38db/details>
