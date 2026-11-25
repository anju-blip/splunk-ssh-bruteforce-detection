Project 01: Building a Splunk Dashboard for SSH Brute-Force Detection
What is this project?
In this lab, I set up a mini Security Operations Center (SOC) to see what a real SSH brute-force attack looks like from the defender's point of view. I used a Kali Linux machine to launch the attack against an Ubuntu server, and then used Splunk to catch the attacker red-handed in the system logs and visualize the attack on a dashboard.

## The Setup (My Lab Environment)
*   **The Attacker:** Kali Linux (using the `hydra` tool)
*   **The Victim:** Ubuntu Server (running an SSH server)
*   **The Defender:** Splunk Enterprise (Free Edition)
*   **Infrastructure:** VirtualBox (Both VMs running on an isolated Host-Only network for safety)

## How I Executed the Attack
1. First, I set up the Ubuntu server to be deliberately vulnerable. I installed OpenSSH and created a test user named `labuser` with a terribly weak password: `password123`.
2. From my Kali machine, I used `hydra` and a custom short wordlist to blast the SSH service with login attempts. 
3. Within seconds, Hydra successfully guessed the password, proving how dangerous weak credentials are.

## How I Detected It (The Splunk Magic)
As soon as the attack happened, Ubuntu started logging hundreds of "Failed password" events in `/var/log/auth.log`. 

I configured Splunk to monitor this specific log file. To actually make sense of the noise, I wrote a custom SPL (Splunk Processing Language) query. I used Regex to pull the attacker's IP address out of the raw log text, and then charted it over time:

```spl
index=* "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| timechart count by src_ip
