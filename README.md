<h1>Memory Forensic</h1>

A project done when going through "Concept And Techniques For Malware Analysis" module during my university
 
<h2>Description</h2>

A memory forensic case study 

<h2>Environment and Tools Used</h2>

- Conduct the forensic in <b>Kali Linux</b> within a virtual environment
- <b>Volatility</b> are used to analyse the digital evidence of assignment1.vmem file where I have used volatile3 plugins to conduct memory forensic and any malicious document on the target for the investigation.
- <b>CyberChef</b> to decode a encoded command script.
- <b>LibreOffice</b> to open potential malicious office document to get a better overview of the incident
- <b>VirusTotal</b> for cross-referencing, analyses suspicious files to detect types of malware and malicious content using antivirus engines and website scanners.

<h2>Background</h2>

You are working as a Digital Forensics Analyst as part of the Incident Response team in Advanced
Cores Enterprises. You received an alert from your colleagues in the Security Operations Centre that
they detected suspicious network activities originating from the computer belonging to one of the staff
from the Finance department.
As part of your Incident Response Procedure, you have acquired the memory image from the
suspected computer. You have been tasked to perform an analysis on the memory image,
assignment1.vmem , and write a report to document your findings.

<h2>Evidence Acquisition and Analysis</h2>

<p align="left">
<img src="https://imgur.com/YtTF7hU.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
• Utilizing windows.imageinfo plugin in the volatility3 utility framework,target system and device is a windows system running on an x64 architecture.
<br />
<br />
<img src="https://imgur.com/r5mXoir.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	Firstly, used “psscan” plugins to get all the processes at time of when the memory image file was acquired. 
<br />
•	At first glance, suspicious looking process EXCEL.EXE caught my attention. Hence, by using “grep”, to focus that process and find out on the parent process.
<br />
<br />
<img src="https://imgur.com/HFJK8eu.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br /> 
•	To find more clues, I utilized the “cmdline” plugins and I found out that EXCEL.EXE process is used to create a file named latest_finance_accounts.xls. It became more suspicious as the victim works in the finance department hence it may be a document that is embedded with malicious macros. 
<br />
•	At this point of time, I made a hypothesis that the victim received an Outlook email (Outlook.exe was captured in image 1) with the “latest_finance_account” file inside which contained a malicious macro. Once the victim opens the file, it will run the script and execute the payload.
<br />
<br />
<img src="https://imgur.com/jriHl0e.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	I used “pstree” to get the child processes of it and the result shows that the excel.exe spawns a child process powershell.exe which seems even more suspicious. Furthermore, the powershell comes with a whole line of encoded script hence I conduct further investigation.
<br />
<br />
<img src="https://imgur.com/hjb1kL2.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	The powershell.exe spawns another powershell.exe which spawns Excelsvc.exe. As all of the previous clues stream down to this process, I chose this process as my potential malicious target that infected the machine.
<br />
<br />
<img src="https://imgur.com/Ug2frNT.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	Not much information gotten when used cmdline, getsids, handles plugins on the target process.
<br />
<br />
<img src="https://imgur.com/vsiexi2.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	Potential C2 IP address and port as it’s from the suspected process. IP address: 192.168.221.135
<br />
<br />
<img src="https://imgur.com/Xyx4DsM.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	“Malfind” plugin is able to capture result for the target process which helps to support my hypothesis
<br />
<br />
<img src="https://imgur.com/7QpZ2tj.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	To confirm whether the process is malicious, I dumped and copied the file to virustotal. 
<br />
<br />
<img src="https://imgur.com/QTawYdR.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	According to VirusTotal which can concludes that the process is malicious.
<br />
<br />
<img src="https://imgur.com/jBYGBCN.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	As previously found the suspicious encoded powershell script, I copied into cyberchef and decode the script.
<br />
•	Looking at the malicious script, it seems to be trying to download a payload from the attacker C2 server and output the file as the excelsvc.exe to stay undetected.
<br />
•	Based on the url link, the IP address matches the previous ip address.
<br />
<br />
<img src="https://imgur.com/AuNt12f.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	Conducted a quick triage using “strings” commands and multiple keyword related to cryptography was captured,maybe a ransomware incident
<br />
<br />
<img src="https://imgur.com/OXyknBd.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	Dump the excel file
<br />
<br />
<img src="https://imgur.com/nB7tyvV.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	Using oletools (oleid.py), the excel file contain malicious macro.
<br />
<br />
<img src="https://imgur.com/pV0KWms.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<img src="https://imgur.com/W4Cdtc8.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	Olevba.py captured the malicious macro and I proceed to conduct a script analysis on the VBA script. According to the VBA, the script will run automatically when the document is open if the macro is enabled.
<br />
<br />
<img src="https://imgur.com/DB8G74k.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<img src="https://imgur.com/jaHBG2D.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
•	Replace the declared variables with understandable names and decode all the necessary line to get a rough idea of what does the script does. 
<br />
•	As a result of the script analysis, the script will run automatically once the document is open and “powershell L -W 1 -C powershell - \n nc (encoded string from cell(1583,245))” command will be run.
<br />
<br />
<img src="https://imgur.com/ZEzKPQq.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<img src="https://imgur.com/1BNRA1B.png" height="80%" width="80%" alt="IMAGE NOT AVAILABLE"/>
<br />
<h2>Summary</h2>
• In conclusion, based on the content found in the excel sheet. It is highly possible that the victim receive a phishing email with a malicious document attached and the malicious macro was executed when the victim opens the document while under the assumption that it’s the latest_finance_account document.
</p>
