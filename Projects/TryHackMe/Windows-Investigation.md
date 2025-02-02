# TryHackMe Room: Windows Investigation

## Project Overview
The **Investigating Windows** room on TryHackMe focuses on analyzing a simulated Windows environment to uncover potential security issues. 
This simulated exercise covers critical Windows features, event logs, and methods for detecting malicious activity, providing hands-on practice with basic forensic investigation techniques.

---

## Objectives
- Understand the role of **Windows Event Logs** in monitoring and detecting suspicious activity.
- Learn how to use tools like **Event Viewer** and other included Windows system tools to analyze system behaviors.
- Gain practical knowledge in detecting malware and other security incidents on Windows machines.

---

## Skills Developed
- **Forensic Analysis:** Investigated Windows logs and processes to identify anomalies and potential threats.
- **Malware Detection:** Used key indicators to detect suspicious files and malicious processes in a Windows environment.

---

## Preface
- A Windows machine has been hacked, it's our job to go investigate this Windows machine and find clues to what the hacker might have done.   

  We log onto the system via TryHackMe's AttackBox to simulate the work environment.  
  However, upon logging onto the system, we notice something particularly interesting...
  Two strange pop-ups at start up?
  
![sus](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/mimz.png)

![sus2](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/mimz2.png)

Anyways, on to the challenges!
  
  

---

### Questions

#### What's the version and year of the Windows machine?

- Using the command prompt and typing "systeminfo" yields the following results:
   - This could also be accomplished via Start > Settings > System > About.

![systeminfo](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q1.png)

- The Windows machine is running Windows Server 2016.

#### Which user logged in last?

- Using EventViewer, we can filter the results from Windows Logs > Security under Event ID 4624 for successful logins.

![q2.1](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q2.1.png)

![q2.2](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q2.2.png)

![q2.3](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q2.3.png)

![q2.4](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q2.4.png)

- The last user to have logged on was Administrator.

#### When did John log onto the system last?

- While in EventViewer, we can further refine the results to see the last successful logon for John.

![q3](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q3.png)

- However, this can also be done via command prompt with "net user John".

![q3.1](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q3.1.png)

- Both results will show that John last logged on at 03/02/2019 5:48:32 PM.

#### What IP does the system connect to when it first starts?

- This could be seen in the instances that were opened upon start up, but they were relatively quick to pass by.
- So let's verify and check the startup.

- We can do this in one of two ways:

   - View System Information > Software Environment > Startup Programs

![q4](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q4.png)

   - Check the Registry; Registry Editor > HKEY_LOCAL_MACHINE > SOFTWARE > Microsoft > Windowws > CurrentVersion > Run

![q4.1](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q4.1.png)

- The system connects to 10.34.2.3 on startup.
   
#### What two accounts had administrative privileges (other than the Administrator user)?

- This can be determined using command prompt with 'net localgroup "Administrators"'.
  
![q5](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q5.png)

- The other two accounts with administrative privileges were Jenny and Guest.

#### When did Jenny last log on?

- Utilizing command prompt again, we enter "net user Jenny".

![q6](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q6.png)

- She apparently has never logged on...? We'll come back to this later.

#### What's the name of the scheduled task that is malicous? What file wass the task trying to run daily? Over what port?

- To view scheduled tasks, we open, well, Task Scheduler. <br> From there, we can view the Task Scheduler Library to see all tasks scheduled to run.
- 
![tasksc](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/tasksc.png)

- While there are a few suspect processes, the answer we were tasked wwith finding is "Clean file system." <br> Notably, the task isn't actually cleaning anything, but rather is running a PowerShell script as indicated by the .ps1 file. <br> More importantly, the action listed executes "nc.ps1 -l 1348" -- indicating a netcat listener on port 1348.
  
#### At what date did the compromise take place?

- At this point, there are many things that should have set off many alarms, evidence of possible IoCs:
  - A user, Jenny, was created and has never logged on; see previous question.
  - There are various suspect tasks scheduled to run.
    
![q7.1](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q7.1.png)

- These all occur 03/02/2019.
  - Additionally, the scheduled task "GameOver's" event action is: C:\TMP\mim.exe sekurlsa::LogonPasswords > C:\TMP\o.txt <br> Executing a program called "mim.exe," skims passwords, and places them in a file, "o.txt." 

#### During the compromise, at what time did Windows first assign special privileges to a new logon?

- With the information above, we can go back to EventViewer and narrow our focus to 03/02/2019, around 04:00:00 PM to 05:00:00 PM with an Event ID of 4672 to view when special privileges are assigned to a new logon.

![q9.1](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q9.1.png)

![q9.2](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q9.2.png)

- According to the hint provided, the time ends in 00:00:49 PM, which correlates with 03/02/2019 4:04:49 PM. 

#### What tool was used to get Windows passwords?

- If we pry further and investigate the C:\TMP folder, we can see the following:

![q10](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q10.png)

![q10.1](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q10.1.png)

- This just about confirms our suspicions, the attacker is utilizing Mimikatz, an open-source cybersecurity tool designed to extract plaintext passwords, hashes, PINs, and Kerberos tickets from memory on Windows systems.

#### What was the attackers external control and command servers IP?

- For this, we check the hosts file located at C:\Windows\System32\drivers\etc, which serves as a local DNS which allows for the mapping of hostnames to IP addresses on the local machine. 

![q11](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q11.png)

![q11.1](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q11.1.png)

- We see that the IP assigned to google.com is unfamiliar and listed as 76.32.97.132.

#### What was the extension name of the shell uploaded via the servers website?

- This question required a bit of research, but it is essentially asking for the name of the file extension of the shell uploaded by the server's website. <br> As this is a Window's machine, the web server is use is most likely to be IIS.
- By checking C:\inetpub, the default folder for Microsofft Internet Information Services (IIS).

![q13](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q13.png)

- We see that in the wwwroot folder, there are two file extensions: .gif and .jsp. <br> The latter seems suspicious, and via a quick google search, we find that a .jsp file can use template data, custom elements, scripting languages, and server-side Java objects to return dynamic content to a client.

#### What was the last port the attacker opened?

- Since this is dealing with ports, a reasonable tool to utilize is the Windows Firewall. <br> From here, we'll check out the Inbound Rules.

![q14](https://raw.githubusercontent.com/jason-ly/jason-ly.github.io/main/Projects/Assets/TryHackMe/InvestigatingWindows/q14.png)

- This particular rule, "Allow outside connections for development," is without a group and has port 1337 open. <br> Very hacker-ish. 

---

### Further Considerations
- Although this exercise was completed, I would like to further attempt to refine my skills and utilize PowerShell commands in a future iteration.
- For now, I'll continue to tackle parts 2 and 3 of Windows investigations related to this one.

---

## Conclusion
This project demonstrated my ability to:
- Ability to analyze event logs, detect suspicious processes, and investigate malware in Windows environments using tools like Event Viewer and Sysinternals.
  - Basic Windows forensics. 
- Incident Response Skills -- Practical experience in identifying security incidents and tracing malicious activities.

--- 

[View Full Portfolio](https://jason-ly.github.io/)
