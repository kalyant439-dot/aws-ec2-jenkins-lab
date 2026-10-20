# Jenkins Setup on AWS EC2 (Ubuntu)

A beginner-friendly hands-on lab documenting how I launched an Ubuntu EC2 instance, connected to it from Windows using SSH, accessed Jenkins, and completed the initial unlock step.

> **Lab status:** Jenkins was running and the initial administrator password was accepted. The next screen offered **Install suggested plugins**. Plugin installation and administrator account creation should be documented after completion.

## Architecture

```text
Windows laptop (SSH client / web browser)
       | SSH (TCP 22) with private key
       v
AWS EC2 instance (Ubuntu, us-east-2 / Ohio)
       | Jenkins service
       v
Jenkins web UI (TCP 8080)
```

## Prerequisites

- AWS account and an Ubuntu EC2 instance
- EC2 key pair (`.pem` private key) downloaded to your computer
- Public IPv4 address of the instance
- Windows OpenSSH client (available in recent Windows versions)
- Jenkins installed and running on the EC2 instance

**Note:** This guide documents the connection and Jenkins unlock process. The exact Jenkins package installation commands used earlier were not verified in the screenshots, so they are not presented as completed steps.

## 1. Launch or identify the Ubuntu EC2 instance

1. Open the **AWS Console → EC2 → Instances**.
2. Select the Ubuntu instance and confirm **Instance state: Running** and that status checks pass.
3. In **Details**, note its **Public IPv4 address**, **Key pair name**, and **Region**.
4. For this lab, the AWS region was **us-east-2 (Ohio)**.

> EC2 public IP addresses can change when an instance is stopped and started unless an Elastic IP is assigned. Use the current address shown in AWS rather than copying an old example.

## 2. Configure the EC2 security group

Go to **EC2 → Security Groups → select the instance's security group → Edit inbound rules**.

| Purpose | Protocol | Port | Recommended source |
|---|---|---|---|
| SSH from your laptop | TCP | 22 | Your current public IP `/32` |
| Jenkins web interface (temporary lab access) | TCP | 8080 | Your current public IP `/32` |
| Browser-based EC2 Instance Connect (if used) | TCP | 22 | AWS-managed EC2 Instance Connect prefix list for the region |

For **us-east-2**, the managed prefix list name is `com.amazonaws.us-east-2.ec2-instance-connect`. This is different from an ordinary EC2 service address range.

Avoid opening SSH or Jenkins to `0.0.0.0/0` for routine practice. If the connection still fails, verify public IPv4 assignment, subnet routing, network ACLs, SSH service, and key-pair matching.

## 3. Locate the SSH private key on Windows

The downloaded key in this lab was named:

```text
aws-ec2login.pem
```

It was stored in the Windows **Downloads** folder. Keep the `.pem` file private: **never upload it to GitHub**.

## 4. Connect from Windows Command Prompt

Open **Command Prompt** or **PowerShell** on Windows and run the command below, replacing `<EC2_PUBLIC_IP>` with the instance's current public IPv4 address:

```powershell
ssh -i "$env:USERPROFILE\Downloads\aws-ec2login.pem" ubuntu@<EC2_PUBLIC_IP>
```

The command above is for **PowerShell**. In **Command Prompt (CMD)**, use:

```cmd
ssh -i "%USERPROFILE%\Downloads\aws-ec2login.pem" ubuntu@<EC2_PUBLIC_IP>
```

When connecting for the first time, verify the server host-key fingerprint before accepting it. Successful login shows a prompt similar to:

```text
ubuntu@ip-172-31-x-x:~$
```

**Important:** Linux commands such as `sudo` and `apt` must be run *after* logging into the Ubuntu server, not at a Windows `C:\Users\...>` prompt.

## 5. Open Jenkins

In a web browser, visit:

```text
http://<EC2_PUBLIC_IP>:8080
```

If Jenkins is installed and running and the security group permits your IP, the **Unlock Jenkins** page should appear.

For troubleshooting on Ubuntu, you can check the Jenkins service with:

```bash
sudo systemctl status jenkins
```

If needed, inspect recent service logs:

```bash
sudo journalctl -u jenkins --no-pager -n 50
```

## 6. Retrieve the initial Jenkins administrator password

In the **Ubuntu SSH terminal**, run:

```bash
sudo cat /var/lib/jenkins***********************************
```

Copy the output privately, paste it into the **Administrator password** field on the **Unlock Jenkins** page, and click **Continue**.

**Never paste this password into GitHub, a screenshot, or a public issue.**

## 7. Install suggested plugins

After Jenkins accepts the password, the **Customize Jenkins / Getting Started** screen displays:

- **Install suggested plugins** (recommended for this beginner lab)
- **Select plugins to install** (manual selection)

Choose **Install suggested plugins** and wait for installation to finish.

**Next steps to complete and verify:** Create the first administrator user, save the credentials in a password manager, review the Jenkins URL configuration, and reach the Jenkins dashboard. Do not claim these steps are complete until verified.

## Troubleshooting lessons from this lab

| Symptom | Explanation | What helped / what to check |
|---|---|---|
| `sudo is not recognized` | Linux command was run in Windows CMD | Connect to Ubuntu with SSH first |
| `apt is not recognized` | `apt` is Ubuntu's package manager, not a Windows command | Run it only in the Ubuntu terminal |
| `Identity file ... not accessible` | Incorrect filename or path to `.pem` | Check Downloads and use the full quoted path |
| `Permission denied (publickey)` | Server rejected SSH authentication | Confirm username `ubuntu`, correct key pair, and private-key path |
| EC2 Instance Connect fails | Browser-based SSH has its own network and configuration requirements | Check port 22 rules, regional prefix list, public IP, routes, and instance SSH configuration |
| Jenkins says password is incorrect | Wrong or incomplete unlock secret | Read `initialAdminPassword` on the actual EC2 Ubuntu server |
| Jenkins page won't load | Service/network issue | Check service status, TCP 8080 rules, and current public IP |

## Security checklist

- [ ] Do **not** commit `.pem` keys, passwords, access keys, or other secrets.
- [ ] Restrict SSH (22) and Jenkins (8080) to trusted IP addresses.
- [ ] Use a dedicated Jenkins administrator account with a strong password.
- [ ] For any non-lab deployment, use HTTPS (typically behind a reverse proxy) rather than exposing Jenkins over plain HTTP.
- [ ] Apply operating system and Jenkins security updates.
- [ ] Stop or terminate unused EC2 resources to manage AWS costs.

## Suggested repository layout

```text
aws-ec2-jenkins-lab/
├── README.md
├── screenshots/
│   └── README.md
└── .gitignore
```

Suggested `.gitignore` entries:

```gitignore
*.pem
*.key
.env
.env.*
secrets/
```

Screenshots can be added to `screenshots/` **only after** hiding public IPs if desired, account identifiers, tokens, passwords, browser personal data, and any other sensitive information.

## What I learned

- How EC2 security groups control inbound SSH and Jenkins access.
- The difference between Windows CMD and an Ubuntu SSH terminal.
- How to authenticate to EC2 with a `.pem` key.
- How to retrieve the initial Jenkins unlock password from the server.
- How to reach the Jenkins plugin installation step.

