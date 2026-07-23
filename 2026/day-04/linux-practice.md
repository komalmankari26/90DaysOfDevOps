<h1>Real Time Output of Commands I Practiced</h1>
1. View Current Hostname
hostnamectl - Displays detailed hostname and system information.

<img width="1920" height="396" alt="Screenshot (259)" src="https://github.com/user-attachments/assets/98ca5c4a-9fae-4471-a600-9ae2a9888136" />


2. Change Hostname-
   `sudo hostnamectl` set-hostname devops-server - Changes the system hostname.
3. Check system uptime-
   `uptime` - Shows how long the system has been running and load averages.
4. Find CPU Consuming process-
   `ps aux --sort=-%cpu | head `- Lists processes consuming the most CPU.
5. Display process tree-
   `pstree -p` - Shows parent-child relationship between running processes.
6. List running services-
   `systemctl list-units --type=service --state=running`
   <img width="1920" height="1080" alt="Screenshot (261)" src="https://github.com/user-attachments/assets/d1922c0b-ca60-47b1-8dfa-2e1a1383a7e8" />
<img width="1920" height="1080" alt="Screenshot (262)" src="https://github.com/user-attachments/assets/ff03cc54-b0de-49ac-a73d-759571664b74" />
<img width="1920" height="1080" alt="Screenshot (263)" src="https://github.com/user-attachments/assets/e7209a19-6c70-4d91-bc1c-5b21fd8e3873" />
7.Check SSH Service status-
`systemctl status ssh`
8.SSH Logs-
`journalctl -u ssh -n 20`
<img width="1920" height="247" alt="Screenshot (266)" src="https://github.com/user-attachments/assets/b2918405-2572-481b-b371-d849df5290ca" />
9.View Error logs
`journalctl -p err -b`
<img width="1920" height="165" alt="Screenshot (267)" src="https://github.com/user-attachments/assets/816b9781-bf2e-4852-b36f-b7434ec3319d" />
10.Check Disk storage
   `df -h`
<img width="1920" height="396" alt="Screenshot (268)" src="https://github.com/user-attachments/assets/c21bae24-f786-4d97-9ab2-4e40fba0b273" />



<img width="1920" height="531" alt="Screenshot (265)" src="https://github.com/user-attachments/assets/a63b520f-9a02-4751-b079-8ca47179b9a1" />


