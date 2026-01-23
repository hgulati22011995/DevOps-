# Linux Commands

- [Linux Commands](#linux-commands)
  - [User Account and Identity Management](#user-account-and-identity-management)
    - [/usr/bin/chsh](#usrbinchsh)


## User Account and Identity Management
<details>
<summary><b>/usr/bin/chsh</b></summary>
<br>

### /usr/bin/chsh

**`/usr/bin/chsh`** is the actual executable file for the <mark><b>change shell</b></mark> command on a Linux system. To understand it properly, it helps to start from what a shell really is. A <u><b>shell</b></u> is simply the program that <mark><b>sits between you and the operating system kernel</b></mark>; when you type commands in a terminal, the shell reads them, interprets what you want to do, and asks the kernel to do the actual work. Common shells are **`bash`**, **`zsh`**, **`fish`**, and a few others. Your login shell is the shell that automatically starts when you log in to the system, either through a terminal, SSH, or a graphical session that opens a terminal.

Now, **`/usr/bin/chsh`** is the full filesystem path to the program that changes this login shell for a user. The reason it lives in **`/usr/bin`** is historical and practical: this <mark><b>directory holds essential user-level programs</b></mark> that must be available on almost every Linux system. When you type just **`chsh`** in the terminal, your shell looks through directories listed in the **`PATH`** environment variable and eventually finds **`/usr/bin/chsh`** and runs it. Writing the full path explicitly is just a direct way of saying “run this exact program from this exact location,” bypassing any ambiguity.

Internally, what chsh does is quite straightforward but carefully controlled. Linux keeps <mark><b>user account information</b></mark> in a file called **`/etc/passwd`**. Each user entry in that file has several fields, and the <mark><b>last field specifies the user’s login shell</b></mark>, for example **`/bin/bash`**. When you run **`/usr/bin/chsh`**, the program checks who you are, verifies that the new shell you want is allowed (allowed shells are listed in **`/etc/shells`**, which is simply <mark><b>a whitelist to prevent unsafe or invalid programs</b></mark> from being used as shells), and then updates only that shell field in **`/etc/passwd`**. Because modifying this file affects how users log in, **`chsh`** is designed with permission checks so that <mark><b>you can normally change only your own shell</b></mark>, and <mark><b>changing another user’s shell requires administrative privileges</b></mark>.

Here is a single, concrete example that shows how it is used in practice. Suppose your current shell is **`bash`**, and you want to switch to **`zsh`**. You would run:

```bash

/usr/bin/chsh -s /bin/zsh

```

At that moment, **`chsh`** verifies that **`/bin/zsh`** exists and that it appears in **`/etc/shells`**. If everything is valid, it updates your login shell entry. Nothing visibly changes in your current terminal session, because the shell that is already running cannot replace itself. The effect appears the next time you log in or open a new terminal: the system now starts **`/bin/zsh`** as your shell instead of **`bash`**.

So, in essence, **`/usr/bin/chsh`** is not about running commands interactively; it is about changing which command interpreter Linux launches for you at login, by safely updating the system’s user account configuration.


</details>
<br>