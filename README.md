<h1>EDR With LimaCharlie</h1>

<h2>Description</h2>
Based on the blog post <a href="https://blog.ecapuano.com/p/so-you-want-to-be-a-soc-analyst-intro?sd=pf"> So you want to be a SOC Analyst? by ERIC CAPUANO</a>. <br />
Set up a homelab environment to get hands-on experience with reading logs and setting up detection and response alerts in LimaCharlie which will be set up on a Windows virtual machine (VM). A Kali Linux VM will be acting as an adversary by using Sliver to create a payload that will be used on our vulnerable Windows machine.
<br />

<h2>Platforms Used</h2>
- <b>Windows</b> <br />
- <b>Kali Linux</b>

<h2>Utilities Used</h2>
- <b>Powershell</b> <br />
- <b>Sliver C2</b> <br />
- <b>LimaCharlie</b> <br />
- <b>Pypykatz</b>

<h2>Lab walk-through:</h2>

<!-- <p align="center"> -->

<b>Part 1:</b> <br />
Part 1 of the blog post goes through setting up the VMs for the Windows and Ubuntu servers. In my case, I had both a Windows VM and a Kali Linux VM from a previous lab and will be using them for this lab as well.
For my Windows VM, I ensured that Windows Defender was completely turned off. <br />
<img src="https://i.imgur.com/e57ZQcH.png" width="80%" alt="turning off windows defender"/>

<br />

I opened my Windows command prompt and began to install sysmon. <br />
<img src="https://i.imgur.com/3k78yz9.png" width="80%" alt="installing sysmon"/> <br />
The blog mentions that this lab does not directly use sysmon but we will be pushing the sysmon logs into LimaCharlie.

<br />

After creating an account with LimaCharlie and adding a sensor, I was prompted to run the installation key in the command prompt to complete the LimaCahrlie installation. <br />
<img src="https://i.imgur.com/5ReQmCc.png" width="80%" alt="installing LimaCharlie"/> 
<br />
<img src="https://i.imgur.com/AGk5US2.png" width="80%" alt="installing LimaCharlie"/>

<br />

To push the sysmon logs to LimaCharlie, I navigated to "Artifact Collections" in the LimaCharlie GUI and added an artifact rule. <br />
<img src="https://i.imgur.com/jVVdmJv.png" width="80%" alt="pushing sysmon logs to LimaCharlie"/>

<br />

I switched to my Kali machine to begin installing Sliver (a Command & Control (C2) framework by BishopFox). <br />
<img src="https://i.imgur.com/OhokKZC.png" width="80%" alt="installing sliver"/><br />

I created a working directory (/opt/sliver).

<b>Part 2:</b> <br />
I navigated to the working directory I just made and launched Sliver. I then used the Sliver shell to generate a C2 session payload using my Kali machine's IP. <br />
<img src="https://i.imgur.com/KLpXm2D.png" height="80%" width="80%" alt="launching Sliver"/> <br />
I took note of the name of our newly saved implant (DISTURBING_CHIME.exe).

<br />

Now it was time to get the payload onto the Windows machine. I did this my starting up a temporary web server with the command <b>pyhton3 -m http.server 80</b>. On my Windows machine, I navigated to <b>http://[MyKaliVM-IP]:80</b>. 
<br />
<i>Side note: I had a bit of trouble getting my Windows VM browser to load the directory of my Kali VM. My mistake was that I did not ensure that the two machines were on the same network AND had different IP addresses. Both machines kept resolving to the same IP whenever using NAT or Nat Network. I statically set my Kali machine's IP address and was able to download the payload onto the Windows machine just fine after that.</i>

<br/>

In the Kali VM running the Sliver shell, I started the HTTP listener. In the Windows VM, I executed the payload <br />
<img src="https://i.imgur.com/57jmmEg.png" height="80%" width="80%" alt="starting http listener"/> <br />

I can confirm that the payload exexuted properly due to the session check-in in the Sliver shell. <br />
<img src="https://i.imgur.com/JUUyAMC.png" height="80%" width="80%" alt="c2 session started"/> 

<br />

I then began interacting with the C2 session and ran some commands to gain information about the victim machine. In the second screenshot below, I can see that the defensive tool Sysmon64.exe is highlighted in red by Sliver and that the implant we are currently using is highlighted in green <br />
<img src="https://i.imgur.com/7ooZwzG.png" height="80%" width="80%" alt="c2 session interaction"/> <br />
<img src="https://i.imgur.com/YWkKtbw.png" height="80%" width="80%" alt="c2 session interaction"/>

<br />

I went back to LimaCharlie on my Windows VM and navigated to the processes tab. Here I noticed that my implant, DISTURBING_CHIME.exe is listed along with other more benign processes. I also noticed that all the other processes in this tab had a green checkmark, meaning that those processes are signed. <br />
<img src="https://i.imgur.com/ORunvHU.png" height="80%" width="80%" alt="checking processes in LimaCharlie"/>

<br />

After investigating the DISTURBING_CHIME.exe process, I was able to see the source IP address when looking at the network connections. <br />
<img src="https://i.imgur.com/3bGE9AO.png" height="80%" width="80%" alt="checking processes in LimaCharlie"/>

<br />

Now naviagting to the File System tab, I entered the path to the payload and was met with the hash for the executable and an option to scan it with VirusTotal. This scan came out with no items found because the payload was just created. <br />
<img src="https://i.imgur.com/kR3GOLV.png" height="80%" width="80%" alt="checking file system in LimaCharlie"/>
<br />
<img src="https://i.imgur.com/43upymG.png" height="80%" width="80%" alt="scanning hash with VirusTotal"/>

<br />

Next, I navigated to the Timeline tab. I found that here, I could see when I executed the payload on the Windows VM and established a network connection to the C2 on my Kali VM. <br />
<img src="https://i.imgur.com/ANDd6BL.png" height="80%" width="80%" alt="checking timeline in LimaCharlie"/>

<br />

<b>Part 3:</b> <br />
On my Kali machine, I carried out an attack that would allow me to pull credentials from the Windows machine.
<img src="https://i.imgur.com/a67fsEw.png" height="80%" width="80%" alt="dumping credentials using lsass.dmp"/>
<br />
<img src="https://i.imgur.com/Pdi7gnq.png" height="80%" width="80%" alt="dumping credentials using lsass.dmp"/>

<br />

I decided to look a little deeper into how this technique works. I found a tool called pypykatz that allowed me to pull the credentials from the dump file I extracted. <br />
<img src="https://i.imgur.com/pN3yGgd.png" height="80%" width="80%" alt="installing pypykatz"/>
<img src="https://i.imgur.com/bJC2YKe.png" height="80%" width="80%" alt="using pypykatz"/>

<br />

I then checked into LimaCharlie for the logs for the lsass dump. <br />
<img src="https://i.imgur.com/jhbHdmx.png" height="80%" width="80%" alt="checking events in LimaCharlie"/>

<br />

LimaCharlie offers detection and response. Based on the information at hand, I created a detection and response rule. <br />
<img src="https://i.imgur.com/7zyDPIG.png" height="80%" width="80%" alt="creating detection and response alert in LimaCharlie"/> <br />
The rule calls for detection of any event involving SENSITIVE_PROCESS_ACCESS and target process ending in lsass.exe. In this lab in particular, the response section tells LimaCharlie to create an alert labeled "report".

<br />
I then tested the rule. <br />
<img src="https://i.imgur.com/m8WsUaH.png" height="80%" width="80%" alt="testing detection and response alert in LimaCharlie"/> 

<br />

It was time to test the detection and response rule by redoing my credentials attack. After running through the attack again, I naviagted back to LimaCharlie and to it's Detection tab where I found alerts under the category "LSASS access" which is the rule I created.
<img src="https://i.imgur.com/bjDIfZn.png" height="80%" width="80%" alt="checking events in LimaCharlie"/>
