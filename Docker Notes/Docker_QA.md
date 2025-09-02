# 100 Docker Interview Questions & Answers

## Part 1: Docker Fundamentals & Core Concepts
**1. `Scenario`: A new team member is confused about why the company is using Docker instead of traditional Virtual Machines (VMs) for a new microservices project. How would you explain the primary benefits of using containers over VMs in this context?**

**`Answer`**: I would explain that while both provide isolation, Docker containers are much more lightweight. They share the host OS kernel, whereas each VM requires a full guest OS. For a microservices project, this means we can run many more containers on a single host, leading to faster startup times, better resource utilization (CPU, memory), and a more efficient, portable, and consistent deployment process across different environments.

<br>

**2. `Scenario`: You are explaining the Docker architecture to a junior developer. They ask what the "Docker daemon" is and what it does. How do you respond?**

**`Answer`**: The Docker daemon, or dockerd, is the **persistent background service** that **manages Docker objects**. It's the **"brain" of Docker**. It listens for API requests from the Docker CLI (the commands you type, like docker run) and handles all the heavy lifting: building and managing images, running and stopping containers, and configuring networks and volumes.

<br>

**3. `Scenario`: An application is running perfectly on a developer's machine but fails in the staging environment. How does Docker solve this classic "it works on my machine" problem?**

**`Answer`**: Docker solves this by **packaging the application**, its code, runtime, system tools, libraries, and all other dependencies **into a single, isolated unit called a container**. This containerized application **runs the exact same way regardless of the underlying environment** (developer's laptop, staging, or production server), ensuring consistency and eliminating environment-specific issues.

<br>

**4. `Scenario`: What is the difference between a Docker image and a Docker container?**

**`Answer`**: A Docker image is **a read-only template with instructions for creating a container**. It's like a blueprint or a class in object-oriented programming. A Docker container is **a runnable instance of an image**. You can have multiple running containers of the same image, just like you can create multiple objects from a single class.

<br>

**5. `Scenario`: You need to run a Linux-based container on a Windows machine for development. What underlying technology makes this possible?**

**`Answer`**: This is made possible by the **Windows Subsystem for Linux (WSL)**. WSL is **a compatibility layer developed by Microsoft that allows a Linux environment**, including its command-line tools and applications, **to run directly on Windows** without the overhead of a traditional virtual machine.

<br>

**6. `Scenario`**: You are asked to explain the difference between a Type 1 and a Type 2 hypervisor.**

**`Answer`**: A Type 1 hypervisor, also known as a "**bare-metal**" hypervisor, runs directly on the host's physical hardware. 

A Type 2 hypervisor, or a "**hosted**" hypervisor, runs as a software application on top of an existing operating system.

<br>

**7. `Scenario`: How does OS-level virtualization (containerization) differ from traditional hardware virtualization?**

**`Answer`**: OS-level virtualization, or containerization, uses a single host OS kernel and creates multiple isolated user-space instances called containers. In contrast, traditional hardware virtualization uses a hypervisor to create complete virtual machines, each with its own full guest operating system. This makes containers much more lightweight and efficient.

<br>

**Q. Why are my Jenkins builds not triggering automatically when I push code to GitHub while running Jenkins on localhost?**

**`Answer:`** I realized the issue is with webhooks. GitHub or GitLab can’t reach my Jenkins running on localhost:8080, because my laptop isn’t publicly accessible. That’s why the build doesn’t trigger automatically. 

On localhost, I can either:

- use Poll SCM in Jenkins,

- or use a tunneling tool like ngrok to expose Jenkins,

- or host Jenkins on a public server like EC2 where webhooks can call Jenkins directly.

<br>

**Q. Why does the docker build command fail inside Jenkins with an error like “docker not found”?**

**`Answer:`** Jenkins itself is running inside a Docker container, and by default that container doesn’t have access to the host’s Docker engine. So when Jenkins tries to run docker build, it can’t find Docker.

To solve this, I can mount the Docker socket from my host into the Jenkins container, for example:
```docker
docker run -d \
  -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

This way Jenkins can talk to the host’s Docker engine and successfully run docker build.

<br>

**Q. So what’s the core issue with localhost Jenkins compared to a proper server like EC2?**

**`Answer:`** The main issue is **accessibility**. On localhost, GitHub can’t reach Jenkins via webhooks, and Jenkins in Docker can’t access host Docker unless I explicitly give it permission. On a proper server like EC2 with a public IP, webhooks work fine and Docker is more straightforward to integrate with Jenkins.

<br>

**Q. What is the Docker socket, why do we mount it, and what happens after mounting it into the Jenkins container, and if I follow all this, will my build be executed on localhost but automatic trigger won’t happen because of localhost?**

**`Answer:`** <br> The Docker socket (**`/var/run/docker.sock`**) is like a communication bridge between the **`Docker CLI`** and the **`Docker Engine`** (the daemon). 

Normally, when I run **`docker build`** or **`docker run`** on my host, those commands talk to the Docker Engine through this socket file.

When Jenkins runs inside a container, it doesn’t have direct access to the host Docker Engine. That’s why **`docker build`** fails.

So, I mount the Docker socket from my host into the Jenkins container using:
```docker
-v /var/run/docker.sock:/var/run/docker.sock
```

After mounting:

- Jenkins container shares the same socket file as my host.

- Now, when Jenkins executes **`docker build`**, it’s actually sending the command to the host’s Docker Engine through that socket.

- This allows Jenkins (inside a container) to build and run containers on the host system.

- It’s like giving Jenkins a “remote control” to the Docker Engine running on my host.

<br>

But — because Jenkins is running on localhost, GitHub or GitLab webhooks cannot reach it. That means the b**uild won’t trigger automatically**. 

On localhost, I can only:

- run builds manually,

- use Poll SCM (scheduled checks),

- or use tunneling tools like ngrok to expose Jenkins.

> So yes — the build will execute on localhost if I mount the socket, but  automatic triggers won’t work unless Jenkins is publicly accessible (like on EC2).

<br>

**Q. What is Poll SCM, and how does it work in this scenario?**

**`Answer:`** 

Poll SCM is a Jenkins feature where Jenkins **periodically checks (polls) the Git repository for changes** instead of waiting for a webhook.

In my case, since Jenkins is running on localhost:8080, **GitHub webhooks can’t reach it because my machine isn’t publicly accessible**. That means automatic triggers won’t work.

So what I can do is **enable Poll SCM in my pipeline/job**. For example, I can set a schedule like **`H/5 * * * *`**, which means **Jenkins will check my repo every 5 minutes**.

The workflow in this scenario would be:

- I push changes to GitHub.

- Since GitHub can’t notify Jenkins directly, Jenkins won’t know immediately.

- But within 5 minutes (based on my Poll SCM schedule), Jenkins will check the repo.

- If it sees any new commits, it triggers the pipeline automatically.

- So, Poll SCM is like Jenkins saying: “Let me keep checking GitHub myself since GitHub can’t call me.”

- It’s not as efficient as webhooks, but on localhost it’s a practical workaround to get automatic-ish triggers.

