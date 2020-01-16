---
title: Setup
layout: page
root: "."
---

# Get a shell terminal emulator

To connect to Artemis HPC, and follow this lesson, you will need a **'terminal emulator'** program installed on your computer. Often just called a 'terminal', or 'shell terminal', 'shell client', terminal emulators give you a window with a _command line interface_ through which you can send commands to be executed by your computer.

More precisely, these commands are executed by your _shell_, which is a program designed to do just that: execute commands! The most commonly used shell is 'Bash', and we'll generally refer to the shell as a 'Bash shell', and to scripts as 'Bash scripts'. There are other shells, and each has its own set of extra commands or syntaxes it can accept -- its own _scripting language_.

<figure>
  <img src="{{ page.root }}/fig/s_old_term.jpeg" height="250"/>
  <figcaption> The good old days..</figcaption>
</figure><br>

You don't need to worry too much about this! You just need **a** shell -- almost all will understand the commands we're going to be using.

<br>
## Linux systems

If you use Linux, then chances are you already know your shell and how to use it. Basically, just open your preferred terminal program and off you go! An X-Window server (X11) may also be useful if you want to be able to use GUIs; again, if you're using Linux you probably have one, and if you don't have one, it's probably because you intentionally disabled it!

<br>
## OSX (Mac computers and laptops)

Mac operating systems come with a terminal program, called Terminal. Just look for it in your Applications folder, or hit Command-Space and type 'terminal'. You may find that other, 3rd party terminal programs are more user-friendly and powerful -- I use [Iterm2](https://www.iterm2.com/).

<figure>
  <img src="{{ page.root }}/fig/s_terminal_app.png" width="500">
  <figcaption> <b>Terminal</b> is OSX's native terminal emulator.</figcaption>
</figure><br>

We also recommend installing [XQuartz](https://www.xquartz.org/), which will replace OSX's native X-Window server. XQuartz has some extra features that may offer better performance when using GUI programs. You'll need to log out and back in again after installing XQuartz in order for it to activate.

<br>
## Windows

If you're using a Windows machine, don't panic! You might not have used 'CMD' since Windows 95 but, rest assured, Windows still has a couple of terminal programs and shells buried in the Programs menu.

However, those aren't going to work for us, as you'll need extra programs and utilities to connect to Artemis, such as an _SSH_ implementation. To use Artemis on Windows, you have a couple of options:

### A. X-Win32 (recommended)

[X-Win32](https://www.starnet.com/xwin32/) is full-featured X-server and terminal emulator for Windows. USyd [provides a license](http://staff.ask.sydney.edu.au/app/answers/detail/a_id/316) for it; however, the download link is restricted to staff so, students. Install, and follow the instructions on the USyd-ICT page to activate -- you'll need to be on the USyd network or [VPN](http://staff.ask.sydney.edu.au/app/answers/detail/a_id/519/kw/vpn) to do so.

> <h2 data-toc-skip> Windows Defender and internet permissions</h2>
>
> Windows Defender (Windows' firewall/antivirus) will probably tell you that certain access has been blocked for X-Win32 and its built-in sound server.
>
> You should allow all connection domains (check all three boxes) in both of these dialogue windows, so that X-Win32 can work on all types of internet connections.
>
> Or, choose the ones you want if you know what you're doing!
{: .callout}

Then setup as follows:

1. Create a new 'Manual' connection, and select 'SSH'   
   <img src="{{ page.root }}/fig/s_newxwin.png" style="margin:10px">   
   <img src="{{ page.root }}/fig/s_sshxwin.png" style="margin:10px">   

2. Fill in the connection details:
  - Name: **eg "Artemis"**
  - Host: **hpc.sydney.edu.au**
  - Login: **\<Unikey\>**, or Artemis training account **ict_hpctrainX**
  - Command: ```/usr/bin/xterm -ls -fa 'consolas' -fs 11```
  - Password: **\<password\>**   

   <img src="{{ page.root }}/fig/s_confxwin.png" style="margin:10px">
3. Click 'Save'

<br>
### B. PuTTY

PuTTY, an SSH and telnet client, is another good option if you can't or don't want to install X-Win32. However, note that PuTTY **does not** provide an X11 server, so you won't be able to use GUI programs on Artemis with _just_ PuTTY.

Head to [https://putty.org](https://putty.org) and download PuTTY. You can install it to your computer, or just download the 'binary' and run it directly. Create a new session for use with Artemis as follows:

1. Fill in the connection details:
  - Host Name: **hpc.sydney.edu.au**
  - Port: **22**
  - Connection type: **SSH**   

   <img src="{{ page.root }}/fig/s_putty.png" style="margin:10px;height:400px" >
2. Name this session **"Artemis"** and click 'Save'

<br>
# Off-campus access

If you're attempting this training by yourself, or following on **[Zoom](https://uni-sydney.zoom.us/)**, _off-campus_ then you'll need to connect to the USyd internet network _before_ you can connect to Artemis.

There are a couple ways to do this:

## 1. The USyd VPN

**VPN** (Virtual Private Network) is a protocol that allows you to tap into a local private network remotely. Follow USyd ICT's instructions [here](http://staff.ask.sydney.edu.au/app/answers/detail/a_id/316). Once you've connected to the VPN, the above connection methods will work, just as though you were on-campus.

## 2. Use the Artemis Jump server

Artemis provides a 'gateway' server, called **Jump**, that allows connections from outside the University network, and is itself on the network. From the Jump server, you can then connect to Artemis directly. If using the Jump server, you will need to edit the **host address** used in the instructions above:

* Instead of **hpc.sydney.edu.au** _use_ **jump.research.sydney.edu.au**

This will connect you to Jump, rather than Artemis itself. You can then connect to Artemis directly via **SSH**. See [Episode 1 of the _Introduction to Artemis HPC_ course]({{ site.sih_pages }}/training.artemis.introhpc/01-intro).


# Graphical login nodes

There is one final way to access Artemis, and that is using our _graphical login nodes_. These special are graphics-enabled login servers which host 'NoMachine', a kind of remote desktop service.

To use the graphical login nodes:
* Download and install the [NoMachine Enterprise Client](https://pages.github.sydney.edu.au/rc/Artemis-HPC-glogin/) for your operating system. Please don't download the client from NoMachine directly, as their current version may not work with Artemis.
* If you're using MacOS, you need to install [XQuartz](https://www.xquartz.org/).
* Download the [glogin.nxs](https://pages.github.sydney.edu.au/rc/Artemis-HPC-glogin/glogin.nxs) session file to your computer.

To connect:
* Double-click or run the **glogin.nxs** shortcut you downloaded (it will open NoMachine Client).
* A NoMachine window should start, asking for a username and password.
* Replace “YOUR_UNIKEY” with your UniKey, then enter your UniKey password in the password field, then click ‘OK’.
* A short while later, an Artemis terminal window should open.


<br>

{% include links.md %}
