# WRITEUP

HackTheBox Sherlock: Operation Blackout 2025: Phantom Check 

Scenario:
- Talion suspects that the threat actor carried out anti-virtualization checks to avoid detection in sandboxed environments. Your task is to analyze the event logs and identify the specific techniques used for virtualization detection. Byte Doctor requires evidence of the registry checks or processes the attacker executed to perform these checks.

# STEPS

First we will download the provided .zip file and unpack it with p7zip:

![image](https://github.com/user-attachments/assets/6c211048-384a-499c-9d29-2b65090f6670)

We are given 2 `.evtx` files (Windows Event Log files)

![image](https://github.com/user-attachments/assets/7ab336ce-fd43-4406-a2fc-06f6861559c6)

# QUESTION 1 "Which WMI class did the attacker use to retrieve model and manufacturer information for virtualization detection?"

To view the contents of our .evtx files, I will be using evtx_dump.py

Let's take a look in `Windows-Powershell-Operational.evtx`

![image](https://github.com/user-attachments/assets/5c75241e-557e-451d-9067-26dcb242d009)

The common cmdlet for retrieving info from WMI classes in PowerShell is `Get-WmiObject`

We can search for this in the event log and find the use of "Win32_ComputerSystem" to get model details.

![image](https://github.com/user-attachments/assets/ec8db6cb-03e9-4a3f-acc8-db8fa2ec371b)

# QUESTION 2 "Which WMI query did the attacker execute to retrieve the current temperature value of the machine?"

Let's use the same `Get-WmiObject` search to see...

![image](https://github.com/user-attachments/assets/603e36ce-6326-42c6-9a2a-15a870a35f9d)

The query used was `SELECT * FROM MSAcpi_ThermalZoneTemperature`

# QUESTION 3 "The attacker loaded a PowerShell script to detect virtualization. What is the function name of the script?"

Let's use `evtx_dump` on the `Windows-Powershell-Operational.evtx` and search for anything with event ID 4104, which denotes an executed script.

We will search through the 4104s to find anything related to virtualization detection. 

Aha! This looks like the one:

![image](https://github.com/user-attachments/assets/3799381f-46f8-464a-89d9-2a251b25bfc3)

# QUESTION 4 "Which registry key did the above script query to retrieve service details for virtualization detection?"

Typical path format for registry keys look like the following: `HKEY_LOCAL_MACHINE\Subkey\Subkey\Subkey.....`

This script seems to query the following registry key:

![image](https://github.com/user-attachments/assets/db13eb05-a95d-44de-bf9a-b03e85d911bd)

# QUESTION 5 "The VM detection script can also identify VirtualBox. Which processes is it comparing to determine if the system is running VirtualBox?"

Looking through the script's conditionals, we can see a section that looks like it is fingerprinting VirtualBox using the following process:

![image](https://github.com/user-attachments/assets/a68b4e79-6ba2-44db-96f9-a7fc0a8ea886)

# QUESTION 6 "The VM detection script prints any detection with the prefix 'This is a'. Which two virtualization platforms did the script detect?"

![image](https://github.com/user-attachments/assets/8a383fd7-33d4-42c6-9fe5-1658f97046bb)

# END

https://labs.hackthebox.com/achievement/sherlock/2192113/935
