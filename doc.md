# Ansible Configuration Management (Automating Projects 7–10)

## Project Overview
This project demonstrates the use of **Ansible Configuration Management** to automate repetitive server administration tasks previously performed manually.

Using **Ansible**, **Jenkins**, **GitHub Webhooks**, and a **Jenkins-Ansible Bastion Host**, this project introduces Infrastructure Automation through declarative YAML playbooks.

### Objectives
- Install and configure Ansible on an EC2 instance
- Use Jenkins as an automation server for Ansible builds
- Set up GitHub webhook integration with Jenkins
- Create and manage Ansible inventory files
- Write reusable Ansible playbooks
- Automate software installation across multiple servers
- Apply Git branching and pull request workflow
- Test and verify Ansible automation

---

## Architecture
This project uses a **Jump Server (Bastion Host)** architecture.

- **Jenkins-Ansible Server** acts as the control node
- **NFS Server**
- **Web Servers**
- **Database Server**
- **Load Balancer**

---

## Technologies Used
- AWS EC2
- Ansible
- Jenkins
- Git & GitHub
- YAML
- Visual Studio Code
- SSH Agent

---

# Step 1 — Install and Configure Ansible on EC2

## Rename Jenkins Server
Update your Jenkins EC2 instance name tag:

```bash
Jenkins-Ansible
```
![](/Ansible-Configuration-Management-11/images/Rename.png)
## Create GitHub Repository
Create a repository:

```bash
ansible-config-mgt
```
![](/Ansible-Configuration-Management-11/images/git.png)
## Install Ansible
```bash
sudo apt update
sudo apt install ansible -y
```
![](/Ansible-Configuration-Management-11/images/update-ansible.png)
## Verify Installation
```bash
ansible --version
```
![](/Ansible-Configuration-Management-11/images/version.png)
## Configure Jenkins Freestyle Project
Create a Jenkins job:

```bash
ansible
```
![](/Ansible-Configuration-Management-11/images/enkins-ansible.png)

Configure:

- GitHub repository source
- GitHub webhook trigger
- Post-build artifact archiving:

```bash
**
```
![](/Ansible-Configuration-Management-11/images/webhook.png)
![](/Ansible-Configuration-Management-11/images/post.png)

Verify artifacts:

```bash
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```
![](/Ansible-Configuration-Management-11/images/arch.png)
### Note
Assign an Elastic IP to avoid updating webhooks whenever the server restarts.

---

# Step 2 — Prepare Development Environment

Install **Visual Studio Code**.

Clone repository:

```bash
git clone <ansible-config-mgt-repo-link>
```

---

# Step 3 — Begin Ansible Development

Create a feature branch:

```bash
git checkout -b pro11/feature
```
![](/Ansible-Configuration-Management-11/images/branch.png)
Create project structure:

```bash
mkdir playbooks
mkdir inventory
```

Create:

```bash
touch playbooks/common.yml
touch inventory/dev
touch inventory/staging
touch inventory/uat
touch inventory/prod
```

Project structure:

```bash
ansible-config-mgt/
├── inventory/
│   ├── dev
│   ├── staging
│   ├── uat
│   └── prod
└── playbooks/
    └── common.yml
```

---

# Step 4 — Configure Ansible Inventory

Start SSH Agent:

```bash
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```

Verify:

```bash
ssh-add -l
```

SSH into Jenkins-Ansible:

```bash
ssh -A ubuntu@public-ip
```

## Configure `inventory/dev`

```ini
[nfs]
<NFS-Private-IP> ansible_ssh_user=ec2-user

[webservers]
<Web1-Private-IP> ansible_ssh_user=ec2-user
<Web2-Private-IP> ansible_ssh_user=ec2-user

[db]
<DB-Private-IP> ansible_ssh_user=ec2-user

[lb]
<LB-Private-IP> ansible_ssh_user=ubuntu
```
![](/Ansible-Configuration-Management-11/images/inventory.png)
---

# Step 5 — Create Common Playbook

Edit:

```bash
vi playbooks/common.yml
```

Add:

```yaml
---
- name: Update web, nfs and db servers
  hosts: webservers,nfs,db
  become: yes

  tasks:
    - name: Ensure wireshark is latest version
      yum:
        name: wireshark
        state: latest

- name: Update LB server
  hosts: lb
  become: yes

  tasks:
    - name: Update apt repository
      apt:
        update_cache: yes

    - name: Ensure wireshark is latest version
      apt:
        name: wireshark
        state: latest
```
![](/Ansible-Configuration-Management-11/images/playbook.png)

# Step 6 — Push Code to GitHub

## Commit Changes

```bash
git status
git add .
git commit -m "Added Ansible inventory and common playbook"
git push origin pro11feature
```

## Create Pull Request
- Open PR in GitHub
- Review changes
- Merge into `master`

Pull latest code:

```bash
git checkout master
git pull
```

Jenkins automatically archives new build artifacts after merge.

---

# Step 7 — Run First Ansible Test

Navigate to repository:

```bash
cd ansible-config-mgt
```

Run playbook:

```bash
ansible-playbook -i inventory/dev playbooks/common.yml
```
![](/Ansible-Configuration-Management-11/images/install.png)
---

## Verification

Check Wireshark installation on target servers:

```bash
which wireshark
```

Or:

```bash
wireshark --version
```

Expected result:

- Wireshark installed successfully on all managed nodes.

---
![](/Ansible-Configuration-Management-11/images/wireshark-v.png)

## Challenges Encountered
- SSH key forwarding setup
- Inventory configuration troubleshooting
- Jenkins webhook integration
- Managing different package managers (yum vs apt)

---

## Project Outcome
Successfully automated common server configuration tasks using **Ansible**, reducing manual administration effort and improving repeatability.

---

## Author
**Abraham Nwachukwu**  
DevOps / Cloud Engineer

---