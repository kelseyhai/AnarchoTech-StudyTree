# Introduction to Securing (Virtualized) Secure Shell Servers

[Secure Shell (SSH)](https://simple.wikipedia.org/wiki/Secure_Shell) is an encrypted network protocol originally developed in 1995. Since the 90’s, SSH has been a cornerstone of remote administration. It’s one of the first and still one of the most important tools any system administrator must learn to use. The most widely known use of SSH is its application as a remote login tool. The SSH protocol provides the operator of a computer system a secure channel over an unsecured network (like the Internet) to use to access the command line of a remote system, although any network-capable service can be secured using the SSH protocol.

To learn SSH, you need at least two computers talking to each other: one playing client (the administrator's workstation or laptop), and one playing server (the remote system that the admin wants to log in to from afar). These days, multi-machine setups like this are easy to create using the magic of [Virtual Machine (VM)](https://simple.wikipedia.org/wiki/Virtual_machine) hypervisors, which can create many (virtual) machines in just a few clicks. Sometimes referred to as a [“Virtual Private Cloud” (VPC)](https://en.wikipedia.org/wiki/Virtual_private_cloud), these lab environments offer completely free and astonishingly powerful educational and operational opportunities.

This workshop presents a brief crash course in configuring and hardening SSH. Along the way, we’ll also touch on some basics of spinning up a simple VPC using the free and open source [VirtualBox](https://en.wikipedia.org/wiki/VirtualBox) type-2 [hypervisor](https://en.wikipedia.org/wiki/Hypervisor) and the [Vagrant](https://en.wikipedia.org/wiki/Vagrant_%28software%29) hypervisor automation utility. We’ll have to create both the machines themselves and their virtualized network environment, so we'll cover some basic network engineering concepts as well. Finally, we’ll dig into the gritty of hardening (securing) your SSH server and client configurations so you can let your comrades in and keep [the CIA](https://www.ssh.com/ssh/cia-bothanspy-gyrfalcon) out.

![Desired state of the virtualized network topology.](https://github.com/AnarchoTechNYC/meta/raw/master/train-the-trainers/practice-labs/introduction-to-securing-virtualized-secure-shell-servers/Virtualized%20Network%20Topology.svg?sanitize=true)

# Contents

1. [Objectives](#objectives)
1. [Bill of materials](#bill-of-materials)
1. [Prerequisites](#prerequisites)
1. [Set up](#set-up)
    1. [VirtualBox installation](#virutalbox-installation)
        1. [VirtualBox installation on Windows](#virtualbox-installation-on-windows)
        1. [VirtualBox installation on macOS](#virtualbox-installation-on-macos)
        1. [VirtualBox installation on GNU/Linux](#virtualbox-installation-on-gnulinux)
        1. [VirtualBox installation on FreeBSD](#virtualbox-installation-on-freebsd)
        1. [VirtualBox installation on Solaris](#virtualbox-installation-on-solaris)
    1. [Vagrant installation](#vagrant-installation)
        1. [Vagrant installation on Windows](#vagrant-installation-on-windows)
        1. [Vagrant installation on macOS](#vagrant-installation-on-macos)
        1. [Vagrant installation on GNU/Linux](#vagrant-installation-on-gnulinux)
        1. [Vagrant installation on FreeBSD](#vagrant-installation-on-freebsd)
        1. [Vagrant installation on Solaris](#vagrant-installation-on-solaris)
    1. [Vagrantfile creation](#vagrantfile-creation)
1. [Practice](#practice)
    1. [Introduction](#introduction)
1. [Discussion](#discussion)
    1. [Vagrant multi-machine](#vagrant-multi-machine)
1. [Additional references](#additional-references)

# Objectives

When you complete this lab, you will have acquired the following capabilities:

* The ability to use Vagrant to create VirtualBox-backed virtual machines.
* The ability to perform basic configuration and troubleshooting of VirtualBox-backed virtual networks.
* The ability to audit an SSH server and client configuration file to spot potential security weaknesses.
* The ability to log in to an SSH server using SSH's built-in public key-based ("passwordless") authentication mechanism.

# Bill of materials

This folder contains the following files and folders:

* `README.md` - This file.
* `Virtualized Network Topology.svg` - A Scalable Vector Graphics image file displaying the desired network topology for this lab.
* `Virtualized Network Topology.xml` - An editable [Draw.IO](https://draw.io/) diagram that can be exported as SVG to produce the `Virtualized Network Topology.svg` image file.
* `centos-7/` - Used for the CentOS 7 Vagrant VM.
    * `Vagrantfile` - The Vagrant configuration for the CentOS 7 virtual machine.
* `ubuntu-xenial64/` - Used for the Ubuntu Xenial 64  Vagrant VM.
    * `Vagrantfile` - The Vagrant configuration for the Ubuntu Xenial64 virtual machine.

# Prerequisites

To perform this lab, you must have:

* A computer running any modern version of:
    * Windows,
    * macOS,
    * FreeBSD,
    * Solaris, or
    * any flavor of the GNU/Linux operating system.
* An active Internet connection (for downloading the required tools in the [set up](#set-up) step, as well as Vagrant base boxes, and the required software packages into the virtual machines; you do not need an Internet connection once you have completed the set up portion of this lab).

> :beginner: :computer: This exercise requires the use of a command line, or "terminal." If you don't know what that means, or if you do but you feel intimidated by that, consider spending an hour at [Codecademy's interactive tutorial, "Learn the Command Line"](https://www.codecademy.com/learn/learn-the-command-line) (for macOS or GNU/Linux users) or reviewing [Computer Hope's article, "How to use the Windows command line (DOS)"](http://www.computerhope.com/issues/chusedos.htm) (for Windows users) if you want to get started quickly. You don't need to complete the whole tutorial or article to understand this exercise, but it will dramatically improve your comprehension of this exercise's mechanics. If you want a more thorough but equally gentle introduction to the same material, consider reading (and listening to) [Taming the Terminal](https://www.bartbusschots.ie/s/blog/taming-the-terminal/).

# Set up

In addition to your laptop or desktop computer, you will need to acquire the following tools.

* For managing the virtual machines: [VirtualBox](https://www.virtualbox.org/) version 5.0 or newer, sometimes written as *VBox*.
* For automatically configuring virtual machine settings: [Vagrant](https://vagrantup.com/), version 2.1 or newer.
    * [Ruby](https://www.ruby-lang.org/), which is requried by Vagrant.

There are pre-built versions of the VirtualBox hypervisor software for Windows, macOS, GNU/Linux, and Solaris available for download from the [VirtualBox downloads page](https://www.virtualbox.org/wiki/Downloads). Your operating system's package repositories may also include a copy of VirtualBox, but be certain to double-check the version available there before installing it to ensure you are using a new-enough version of the software. For [FreeBSD users, VirtualBox is provided as a package or a source port and can be installed by following the instructions in the FreeBSD Handbook, §21.6](https://www.freebsd.org/doc/handbook/virtualization-host-virtualbox.html).

Similarly, there are pre-built versions of Vagrant for Windows, macOS, and numerous different GNU/Linux flavors available from the [Vagrant downloads page](https://www.vagrantup.com/downloads.html). [Vagrant is also provided as a FreeBSD port](https://svnweb.freebsd.org/ports/head/sysutils/vagrant/). Solaris users can [install Vagrant by installing from source](https://www.vagrantup.com/docs/installation/source.html).

## VirtualBox installation

In most cases, VirtualBox can be easily installed by [downloading the appropriate pre-built package](https://www.virtualbox.org/wiki/Downloads) for your operating system and hardware architecture and following the standard installation procedure as you would any other software. This section provides additional guidance for installing VirtualBox, and offers advice to troubleshoot installation problems.

### VirtualBox installation on Windows

> :construction: TK-TODO

### VirtualBox installation on macOS

> :construction: TK-TODO

### VirtualBox installation on GNU/Linux

> :construction: TK-TODO

### VirtualBox installation on FreeBSD

> :construction: TK-TODO

### VirtualBox installation on Solaris

> :construction: TK-TODO

## Vagrant installation

In most cases, Vagrant can be easily installed by [downloading the appropriate pre-built package](https://vagrantup.com/downloads.html) for your operating system and hardware architecture and following the standard installation procedure as you would any other software. This section provides additional guidance for installing Vagrant, and offers advice to troubleshoot installation problems.

### Vagrant installation on Windows

> :construction: TK-TODO

### Vagrant installation on macOS

> :construction: TK-TODO

### Vagrant installation on GNU/Linux

> :construction: TK-TODO

### Vagrant installation on FreeBSD

> :construction: TK-TODO

### Vagrant installation on Solaris

> :construction: TK-TODO

## Vagrantfile creation

Once VirtualBox is installed and running, you can manually create a new virtual machine by using the VirtualBox Manager graphical user interface (GUI) application, described in [the VirtualBox Manual §1.7](https://www.virtualbox.org/manual/ch01.html#idm272). Setting up a new virtual machine through the graphical interface typically requires many clicks, and can take a considerable amount of time. Vagrant is used to automate this process so that a virtual machine with a given configuration is accessible to you in mere moments.

To accomplish this, Vagrant reads files that describe various aspects of a virtual machine. These aspects range from what hardware to virtualize (e.g., how many network adapters the virtual machine should have, or how much memory should be installed into it), to what commands should be run when the virtual machine boots up for the first time (e.g., which software packages to install to prepare it for a given use). All of this information is contained within [a file literally named `Vagrantfile`](https://www.vagrantup.com/docs/vagrantfile/).

> :beginner: The `Vagrantfile`s for both the server and client virtual machines have already been created for you. If you are not interested in learning some Vagrant basics right now, you can download these files, place them inside two sibling folders manually, and then continue to the [Practice](#practice) section of this lab. The two files you'll need are [`centos-7/Vagrantfile`](centos-7/Vagrantfile) and [`ubuntu-xenial64/Vagrantfile`](ubuntu-xenial64/Vagrantfile). Read this section to learn how we created these files and what their contents describe.

A single `Vagrantfile` is intended to describe a complete Vagrant *project*. When Vagrant finds a `Vagrantfile`, the folder in which the `Vagrantfile` is found is considered the Vagrant *project root*. For the purposes of this lab, we will be using separate Vagrant projects for each virtual machine. This means we will be creating two `Vagrantfile`s, one for the SSH server and the other for our SSH client.

> :bulb: A single `Vagrantfile` can actually describe the configuration of multiple virtual machines. See the [Vagrant multi-machine](#vagrant-multi-machine) discussion for more information on this Vagrant feature. If you are already comfortable with Vagrant, consider re-writing our multiple `Vagrantfile`s as a single multi-machine `Vagrantfile`, instead.

Since a Vagrant project must contain a `Vagrantfile`, we will need to make a folder to house that file. Inside that new folder, we must write a `Vagrantfile` that describes the configuration of the first of our two machines. Then we'll repeat the process to describe our desire for the second of our two machines. This lab is written to use GNU/Linux [CentOS](https://centos.org/) 7 for the server and GNU/Linux [Ubuntu](https://ubuntu.com/) 16 (codenamed "Xenial") as the client. In theory, you could use any operating systems you want, and we encourage you to try out other operating systems after you complete this lab.

Vagrant's main command line utility (`vagrant`) offers a number of convenience functions to help us write these `Vagrantfile`s. We'll use [the `vagrant init` command](https://www.vagrantup.com/docs/cli/init.html) for this purpose. Let's create our Vagrant virtual machine configurations now.

**Do this:**

1. Create a new folder named `centos-7`.
1. In the `centos-7` folder, create a new file named `Vagrantfile` that contains a Vagrant configuration for a CentOS 7 virtual machine:
    ```sh
    vagrant init --minimal --output centos-7/Vagrantfile centos/7
    ```
1. Alongside the first folder, create a second folder named `ubuntu-xenial64`.
1. In the `ubuntu-xenial64` folder, create a new file named `Vagrantfile` that contains a Vagrant configuration for an Ubuntu Xenial virtual machine:
    ```sh
    vagrant init --minimal --output ubuntu-xenial64/Vagrantfile ubuntu/xenial64
    ```

The final argument to both commands (`centos/7` in the first case and `ubuntu/xenial64` in the second) map to Web addresses of pre-packaged virtual machine settings and hard disk images containing pre-installed copies of the named operating system at the specified version. These pre-packaged virtual machine environments are called [*Vagrant boxes*](https://www.vagrantup.com/docs/boxes.html). A public catalog of Vagrant boxes is available at [VagrantCloud.com](https://vagrantcloud.com/), and both [the `centos/7` Vagrant box](https://app.vagrantup.com/centos/boxes/7) as well as [the `ubuntu/xenial64` Vagrant box](https://app.vagrantup.com/ubuntu/boxes/xenial64) are listed there.

In these commands, the `--minimal` flag is optional. It merely instructs the `vagrant init` command not to include instructional comments in the written `Vagrantfile`. These comments are useful for new projects but unnecesary for this lab.

The `--output` flag is how you can tell `vagrant init` to write the `Vagrantfile` at a particular fileystem location, rather than the default. The default is simply to place the written `Vagrantfile` in the current folder. Since we wanted to write the `Vagrantfile` into the folder we just created, we specified `--output` explicitly.

At this point it would behoove you to inspect the Vagrantfiles, so open each in a text editor. Find the line that begins with `config.vm.box`. This is a variable assignment. When Vagrant loads a `Vagrantfile`, it constructs a `config` object. The `config` object has a `vm` member variable, which is also an object. [In this `vm` object, Vagrant keeps the specific settings for the virtual machine](https://www.vagrantup.com/docs/vagrantfile/machine_settings.html). In this case, the `box` variable stores the name of the Vagrant box on which this Vagrant project is based.

> :beginner: This multi-level ("nested") object construction is typical of code written in the [Ruby](https://ruby-lang.org/) programming language. In fact, a `Vagrantfile` is just a Ruby script with numerous pre-defined variables that you are expected to set as you desire. Since a Vagrantfile is just a Ruby script, the more Ruby you learn, the more your Vagrantfiles can do. If Ruby is new (and interesting) to you, we encourage you to spend some time at [Codecademy's Learn Ruby tutorial](https://www.codecademy.com/learn/learn-ruby). If you have less time, you can also [visit TryRuby to get an interactive, whirlwind tour in your Web browser](https://ruby.github.io/TryRuby/). We also like [Ruby Monsters's Ruby for Beginners guide](https://ruby-for-beginners.rubymonstas.org/).

For instance, the CentOS 7 machine's `Vagrantfile` should have a line that looks like this:

```
config.vm.box = "centos/7"
```

Meanwhile, the Ubuntu Xenial machine's `Vagrantfile` should have a similar line, but the `config.vm.box` variable should be assigned a different value:

```
config.vm.box = "ubuntu/xenial64"
```

On the left side of the `=` sign is the full name of the variable (`config.vm.box`). The equals sign (`=`) is Ruby's assignment operator, which takes the value to the right of the equals sign and saves it to the variable named on the left. After Vagrant reads this line in your `Vagrantfile`, Vagrant will know which box you want to use in your project.

Every virtual machine that Vagrant configures always has at least one network adapter. This first network adapter and its configuration is hard-coded and cannot be changed. (Well, not without changing the source code for Vagrant itself, anyway). Vagrant always configures this adapter to use [VirtualBox's `NAT` networking mode](https://www.virtualbox.org/manual/ch06.html#network_nat). In this mode, the virtual machine is able to access the Internet through the physical (host) machine's own network connection, but no other machines can access it because it is placed behind a virtual [Network Address Translation (NAT)](https://simple.wikipedia.org/wiki/Network_address_translation) router of its own.

In order for our two virtual machines to be able to hear one another when they speak, they need to be connected to the same network. To accomplish this, we can instruct Vagrant to instruct VirtualBox to add a second virtualized [network interface card (NIC)](https://en.wikipedia.org/wiki/Network_interface_controller) to each machine and to connect both machines's second NIC to the same virtualized network. To have Vagrant add subsequent NICs to a virtual machine, we use [the `config.vm.network` method](https://www.vagrantup.com/docs/networking/basic_usage.html) call.

> :beginner: A *method*, in programmer-speak, is a function that an object can perform. In Ruby, everything is an object, so all functions are technically methods. In our case, the `config.vm` object is, as stated, an object, and `network` is the name of one of the `vm` object's functions. This is the method Vagrant uses to configure virtualized NIC hardware on the virtual machine. What a method or function actually does depends on the *arguments* given (or "passed") to it.

Each time we call the `config.vm.network` method, Vagrant tries adding another NIC to the virtual machine it's building for us. We'll want to ensure that our second NIC is not accessible by the outside world, so we'll pass [`private_network`](https://www.vagrantup.com/docs/networking/private_network.html) as the first [(positional) argument](https://ruby-doc.org/core-2.0.0/doc/syntax/calling_methods_rdoc.html#label-Positional+Arguments) to the method. Further, we want to have Vagrant configure the virtual machine's operating system to automatically configure an IP address for that network interface, so we'll also pass `type: "dhcp"` as a [keyword argument](https://ruby-doc.org/core-2.0.0/doc/syntax/calling_methods_rdoc.html#label-Keyword+Arguments). Finally, we don't just want to attach the network interface card to any random network, but a specific network, so we'll give this network a name, say `sshtestnet`, by passing `virtualbox__intnet: "sshtestnet"` as a another keyword argument. The full method call will look like this:

```
config.vm.network "private_network", type: "dhcp", virtualbox__intnet: "sshtestnet"
```

Both of our `Vagrantfile`s will need this same line, and by including this same line in both projects, both virtual machines will be attached to the same virtual network.

**Do this**:

1. Open the `Vagrantfile` for your CentOS 7 Vagrant project in a text editor.
1. Add the `config.vm.network` method call as described above inside the configuration block (i.e., immediately following the `config.vm.box` line), then save the file.
1. Repeat the first two steps for your Ubuntu Xenial Vagrant project, as well.

Your Vagrant projects are now configured. :) You're ready to begin the practice lab.

# Practice

We'll begin by ensuring you have successfully completed the [set up](#set-up) steps. This process will also introduce the fundamentals that you need to understand to complete the rest of the exercise.

## Introduction

### Lay of the land

> :construction: Keep developing this!

First, let's get a lay of the land by checking out our Vagrantfiles. The Vagrantfiles determine how our virtual machines (run on VirtualBox) are going to be configured and provisioned.

In the main directory of the lab (`introduction-to-securing-virtualized-secure-shell-servers`), you'll see two other directories already here for you, the names of which may be familiar to you. They are:

* `ubuntu-xenial64`
* `centos-7`

These are, as you may know, the names of operating systems. We can imagine that these are actually two actual computers we have. First, we're going to check out what our `ubuntu-xenial64` machine looks like.

So, `cd` into the directory `ubuntu-xenial64` and `ls` to take a look around.

You'll see there's one file here, called `Vagrantfile`. That's the [Vagrantfile](https://www.vagrantup.com/docs/vagrantfile/) for our computer running on Ubuntu Xenial (16.04). To confirm that, do a quick `less` on the `Vagrantfile`. You'll see a few lines that configure the OS (`config.vm.box = ubuntu-xenial64`).

Now you can do the same thing in your `centos-7` directory's Vagrantfile.

> :beginner: :bulb: It's really helpful to be able to see both screens at once. I recommend using a program like `screen` to enable multiple terminals at once, or else you can simply open two terminal windows and place them next to each other.

`cd` into the `centos-7` directory.

`cd` into the `ubuntu-xenial64` directory.

`ls` in both directories to confirm the existence of the Vagrantfile in both directories.

Great! You can check out the Vagrantfiles if you like, but we recommend not changing them for the purposes of this lab, since they've been configured to make learning hardening SSH.

### Assigning first roles

Now you're going to decide which machine will play the server and which will play the client. It's more typical to expect the CentOS 7 machine to play the server and the Ubuntu machine will be the client, due to the nature of the OSes, but it's certainly not unheard of to have things be the other way around. We're going to learn how to do this both ways, anyway, so it doesn't matter too much what you choose right now. ;)

## Confirming DHCP works

In order for our two computers to be able to talk to each other, they each need their own unique IP address. This comes from a virtual DHCP server that needs to be created before we start anything.

The configuration specified in the Vagrantfile using `config.vm.network` already included in this repository *should* do this for you automatically, however, if it *doesn't*, we need to set up a DHCP server manually and renew the leases on our machines.

Here's how to first check and see if you need to renew the leases, and how to do that if necessary. 

### Checking IP addresses

This process must be done twice; once for either machine.

1. `cd` into the directory of the machine.
1. Use the command `ip a` to show the network IP addresses for each device.
1. At the bottom of that output, typically on device 3 (`eth1` or `enp0s8`, for example), you should see output similar to the following:

```
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:e0:f3:4f brd ff:ff:ff:ff:ff:ff
    inet 172.16.10.2/24 brd 172.16.10.255 scope global noprefixroute dynamic eth1
       valid_lft 1092sec preferred_lft 1092sec
    inet6 fe80::a00:27ff:fee0:f34f/64 scope link
       valid_lft forever preferred_lft forever
```
1. If you *have* an IP address following `inet`, then your DHCP configuration most likely worked, and you should be okay to skip to [Assigning first roles](#assigning-first-roles).

If you *do not* have an IP address following `inet`, or no `inet` line at all, then do the following steps to [renew your DHCP leases](#renewing-dhcp-leases).

### Renewing DHCP leases

This process must be repeated for any/either machine that *does not* have an IP address.

1. First, shutdown your virtual machines by exiting to your host and using `vagrant halt`.

1. In the directory of the machine (eg. `ubuntu-xenial64`), use the VirtualBox commandline tool `vboxmanage` to list your DHCP servers:

`vboxmanage dhcpserver list`

1. The DHCP server we want to see in this list is called `sshtestnet`. This was configured by the authors of this lab so that users like you could have a DHCP server ready to go. If it's not working, this means it needs to be added to the list of DHCP servers. We can do this by using `vboxmanage` with the following options:

```
vboxmanage dhcpserver add --netname sshtestnet --ip 172.16.10.1 --netmask 255.255.255.0 --lowerip 172.16.10.2 --upperip 172.16.10.25 --enable
```
Here's a breakdown of this command and a tiny surface intro to some networking basics:

* `vboxmanage` - the VirtualBox command line configuration utility
* `dhcpserver` - tell `vboxmanage` we're going to be configuring DHCP servers
* `add --netname sshtestnet` - Tell `vboxmanage` that the `intnet` we already have called `sshtestnet` needs to be added to the list of DHCP servers
* `--ip 172.16.10.1` - For our DHCP server, we'd like an IP range using the non-routable IP address range `172.16.10`, and we'd like the DHCP server itself to be known as machine `1` in the context of this address.
* `--netmask 255.255.255.0` - This configures the netmask so that our IP address has a certain structure; namely, that the first three "octets" (so, the first three parts, as in: `###.###.###`) in an IP address are reserved for network addresses. The final `0` tells us that the last octet in our IP addresses will be to specify machines (and from the command prior, we told VirtualBox we want our DHCP server - the machine itself - to be machine number `1`).
* `--lower-ip 172.16.10.2` - This tells us that the lowest number in our IP range will be `2`. It can't be `1`, remember, since that's already been assigned to the DHCP server.
* `--upper-ip 172.16.10.25` - This tells us that the highest number in our IP range will be `25`, which also means that we have space for a total of 24 other machines on this network. (Note: the highest it can go up to is 254, because that's the maximum number of bits that can be expressed in an octet. The reason we have Ipv6 now is actually because we're running out of space just using the Ipv4 conventions.)
* `--enable` - Turn that sucker on. (Enable the DHCP server we just specified.)

1. Now that your DHCP server is configured (hopefully correctly), go ahead and spin up your virtual machines again by running `vagrant up` on either/both. *Make sure to check again by running `ip a` on both machines to ensure that both machines now have IP addresses, and that they are configured as expected!*

## Setting up the server

Once you decide which machine is going to be your server for the first go-around, `cd` into the directory of that machine.

The command to start a virtual machine using Vagrant is `vagrant up`. In either directory, go ahead and do this. This will start up the virtual machines respectively.

Vagrant automatically provisions the machines with an SSH server, so that you can go inside them. If this were not the case, we'd have to manually install `sshd` using the appropriate package manager (such as `yum` or `apt`).

`vagrant ssh` into whichever machine will play the role of the server for now.

We're going to need to be `sudo` in order to do business here, so I recommend just immediately doing `sudo su` right off the bat.

First, let's set up our server machine in a secure way.

1. Use a text editor (such as Vi) to edit the `/etc/ssh/sshd_config` file.
1. We're going to use [the Tech Autonomy guide on hardening SSH](https://we.riseup.net/tech-autonomy+infrastructure/ssh#harden-ssh-server) for this.
    1. Follow the instructions under "Harden SSH server" there, including the `moduli` file instructions using the bash commands, if applicable.
        * *Note*: The last line of the `sshd_config` suggested in the Tech Autonomy guide says to add `DebianBanner no` so that the client does not see what the OS of the server is upon login. For CentOS 7, this configuration line is already available as the commented line:

        ```
        # no default banner path
        # Banner none
        ```
        Simply uncommenting (removing the `#` from the beginning of `Banner none`) will do the same thing. 

    1. Remove unused keys as described by the guide; regenerate them with stronger crypto.
    1. Make the special sudo group `ssh-users`. This group will specify who can SSH into this machine in the first place.
    1. Right away, add the following users to this group, or else you won't be able to `vagrant ssh` into your machine anymore:

        * `vagrant`
        * `root`
        * Any other user accounts you may have optionally created

    Using the command:
    `usermod -a -G ssh-users $THE_USERNAME`

1. Now that we're prepared, we're going to pretend to be very businessy business people and create an account for our client.

   ![Business!](https://media1.tenor.com/images/e64055c1a218c7b602b29d85bde7ee38/tenor.gif)

    1. `adduser [CLIENT-NAME]`. In my case, for example, I made a client account called `special-client`. Best sysadmin ever.

    1. Add the client to the `ssh-users` group using the command from before:

        `usermod -a -G ssh-users $THE_USERNAME`

    1. Following the Tech Autonomy guide, be sure to [test and apply the new configuration](https://we.riseup.net/tech-autonomy+infrastructure/ssh#test-and-apply-the-new-configuration).

# Discussion

## Vagrant multi-machine

> :construction: Discuss the utility of [Vagrant's multi-machine features](https://www.vagrantup.com/docs/multi-machine/). This is a great "exercise left to the reader," since it is a relatively advanced Vagrant-specific construct and slightly tangential to SSH hardening.

# Additional references
