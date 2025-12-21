# Ansible Introduction

<br>
<br>

- [Ansible Introduction](#ansible-introduction)
  - [Why Ansible Exists](#why-ansible-exists)
  - [What Ansible Really Is](#what-ansible-really-is)
  - [Push-Based Working Model](#push-based-working-model)
  - [How Ansible Connects to Servers](#how-ansible-connects-to-servers)
  - [Real-World Use Cases](#real-world-use-cases)
  - [Ansible vs Terraform (Practical Difference)](#ansible-vs-terraform-practical-difference)
  - [Ansible vs Chef](#ansible-vs-chef)
  - [Basic Ansible Setup](#basic-ansible-setup)
  - [Typical Real-World Workflow](#typical-real-world-workflow)
  - [Declarative Mindset](#declarative-mindset)
  - [When Ansible Makes Sense (and When It Doesn’t)](#when-ansible-makes-sense-and-when-it-doesnt)
  - [Final Practical Takeaway](#final-practical-takeaway)


## Why Ansible Exists

- Ansible exists because managing servers manually does not scale. That’s the real reason. Not theory, not buzzwords.

- In real life, when you work as a system admin or DevOps engineer, you don’t manage one server. You manage many. Sometimes 10, sometimes 50, sometimes hundreds. Some servers are missing updates, some have services stopped, some have wrong versions of tools installed. Logging into each server using SSH and fixing things one by one is slow and error-prone.

- Ansible solves this exact problem. It lets me control many servers from one place and make them behave the same way.

---

<br>
<br>

## What Ansible Really Is

- Ansible is <mark><b>a configuration management</b></mark> and <mark><b>automation tool</b></mark>, but instead of remembering that definition, think of it this way: I tell Ansible what I want, and Ansible makes it happen on all servers.

- For example, I don’t say “run this command, then check output, then run another command”. I say “nginx should be installed and running”, and Ansible handles the rest. This mindset is very important.

---

<br>
<br>

## Push-Based Working Model

- One very important thing about Ansible is that it works on a push model. There is one machine where Ansible is installed. This machine is called the control node or master. From this machine, Ansible connects to all other servers and pushes changes to them.

- The other servers do not need Ansible installed. They only need SSH access. This makes Ansible very simple compared to older tools like Chef or Puppet, which require agents running on every server.

---

<br>
<br>

## How Ansible Connects to Servers

- Ansible <mark><b>connects to servers using SSH</b></mark>. Nothing special. No custom protocol. No background service. Just SSH.

- In most real setups, <mark><b>key-based authentication is used</b></mark>. The Ansible control node has a private key, and all target servers trust that key. Once this is set up, Ansible can connect to any server without asking for a password.

- This is also why understanding SSH is mandatory before using Ansible seriously.

---

<br>
<br>

## Real-World Use Cases

- In real projects, Ansible is used for very practical tasks. Installing software like Docker or nginx. Starting or restarting services. Copying configuration files. Deploying applications. Updating operating systems. Managing dev and production environments separately.

- If there are multiple servers and consistency matters, Ansible fits naturally.

---

<br>
<br>

## Ansible vs Terraform (Practical Difference)

- A very common confusion is Ansible vs Terraform. The difference is simple when you look at real usage.

- <mark><b>Terraform</b></mark> is used <u><b>to create infrastructure</b></u>. Virtual machines, networks, load balancers, cloud resources.

- <mark><b>Ansible</b></mark> is <u><b>used after those machines already exist</b></u>. It configures them. Installs packages. Manages services. Deploys applications.

- Yes, Ansible can create servers, and Terraform can run scripts, but in real-world DevOps, <mark><b>Terraform builds</b></mark> and <mark><b>Ansible configures</b></mark>.

---

<br>
<br>

## Ansible vs Chef

- Another comparison people ask about is Ansible vs Chef.

- <mark><b>Chef</b></mark> works on a <mark><b>pull model</b></mark>. Servers pull configuration from a central server and require an agent running on them. This increases complexity.

- <mark><b>Ansible</b></mark> uses a <mark><b>push model</b></mark>. No agent. Just SSH. This simplicity is the main reason Ansible is more widely used today.

---

<br>
<br>

## Basic Ansible Setup

- A basic Ansible setup is very straightforward. You need one Linux machine where Ansible is installed. This is your control node. You need target servers that are reachable over SSH. That’s it.

- Everything runs from the control node. Nothing runs in the background on target servers.

---

<br>
<br>

## Typical Real-World Workflow

- When starting with Ansible in a real environment, the workflow almost always looks the same. First, install Ansible on the control node. Then set up SSH access. Then define servers in the inventory file. After that, test connectivity using a simple ping. Once connectivity works, run ad-hoc commands. Finally, move to playbooks for automation.

![alt text](<diagrams/Ansible Workflow.png>)

---

<br>
<br>

## Declarative Mindset

- Ansible is often called declarative, but in practice this just means you focus on the desired state, not the steps. You describe what the system should look like, and Ansible brings it to that state.

- This avoids configuration drift and keeps servers consistent.

---

<br>
<br>

## When Ansible Makes Sense (and When It Doesn’t)

- Ansible is not a silver bullet. If you have only one server, Ansible may be unnecessary. If SSH access is not possible, Ansible will not work. If infrastructure is changing constantly, Terraform should come first.

- Ansible shines when infrastructure is stable and configuration needs to be consistent.

---

<br>
<br>

## Final Practical Takeaway

- The most important thing to remember is this: Ansible is not about YAML files or modules. It is about solving one real problem — managing many servers without touching them one by one.

- If you keep this mindset, everything else in Ansible will make sense naturally.
