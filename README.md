Think twice before you disable SELinux
======================================

Security Enhanced Linux (SELinux) is a Linux kernel security module that provides a mechanism for supporting access control security policies, including mandatory access controls (MAC). The policies provided by SELinux can be used to determine access not only, but mostly between: Users, Files, Directories, Memory, Process, Devices, Sockets, TCP/UDP Ports etc.

It was first developed and then open sourced by the National Security Agency (NSA) in 2000. You can find the current list of contributors [here](https://www.nsa.gov/what-we-do/research/selinux/contributors.shtml)

Security-Enhanced Linux (SELinux) can be a key component of a defense-in-depth architecture. It enables the enforcement of fine-grain (Mandatory Access Control) security policies. The SELinux road can sometimes be bumpy, which leads many engineers to switch it off. Nowadays you should not be doing that anymore, even if you're a not willing to study it, you should at least set it to permissive, which basicaly allows everything to pass through and generate audit alerts when necessary.

To check the current status of your SELinux

```bash
$ getenforce
Enforcing
```

To set it to permissive

```bash
$ sudo setenforce 0
$
```

To make it permanent

```bash
$ sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g'  /etc/selinux/config
$
```

Now, if you think you can get any benefit from it and has an open mind to learn new things, then please continue and enjoy the reading.

Tools used on this demo
-----------------------

I'm assuming you run GNU/Linux on your desktop (I'm running this **updated version of this** demo on Fedora 28), have installed and configured [nss-mdns](https://github.com/lathiat/nss-mdns), also have installed [vagrant](https://www.vagrantup.com/), [ansible](https://www.ansible.com/) and [libvirt](https://libvirt.org/). Other tools will be downloaded, installed and configured automatically by ansible playbooks.

<p align="center">
  <img src="images/logos.png">
</p>

>NOTICE: **By default vagrant only support virtualbox, here we use libvirt+KVM, so don't forget to install the vagrant plugin [vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt)**

Topology
--------

This is an extremely simple environment with two hosts only.

<p align="center">
  <img src="images/topology.png">
</p>

Bringing up your own environment
--------------------------------

After cloning my repo, navigate to the project folder and start the provisioning by using vagrant commands.

```bash
$ git clone https://github.com/fabiogoma/selinux-nginx-tomcat.git
Cloning into 'selinux-nginx-tomcat'...
remote: Counting objects: 354, done.
remote: Compressing objects: 100% (41/41), done.
remote: Total 354 (delta 13), reused 19 (delta 7), pack-reused 306
Receiving objects: 100% (354/354), 1.19 MiB | 2.53 MiB/s, done.
Resolving deltas: 100% (134/134), done.
$ cd selinux-nginx-tomcat
$
$ vagrant up tomcat
Bringing machine 'tomcat' up with 'libvirt' provider...
==> tomcat: Checking if box 'centos/7' is up to date...
==> tomcat: Creating image (snapshot of base box volume).
...
...
...
RUNNING HANDLER [general : Restart mDNS service] *******************************
changed: [tomcat]

RUNNING HANDLER [tomcat : Reload systemd and restart tomcat service] ***********
changed: [tomcat]

PLAY RECAP *********************************************************************
tomcat                     : ok=21   changed=20   unreachable=0    failed=0
```

The previous command will provision a new virtual machine and setup the tomcat application server inside it. This setup is using a CentOS image which is very similiar to redhat, I would say they are slightly different only on trade mark images and logos, in fact, CentOS is owned by [Redhat Inc](https://www.redhat.com).

The time spent on this provision depends entirely on the type of hardware you have and also your internet speed.

After the execution, let's now do the same for our NGINX host

```bash
$ vagrant up nginx
Bringing machine 'nginx' up with 'libvirt' provider...
==> nginx: Checking if box 'centos/7' is up to date...
==> nginx: Creating image (snapshot of base box volume).
...
...
...
RUNNING HANDLER [general : Restart mDNS service] *******************************
changed: [nginx]

RUNNING HANDLER [nginx : Reload systemd and restart NGINX service] *************
changed: [nginx]

PLAY RECAP *********************************************************************
nginx                      : ok=18   changed=17   unreachable=0    failed=0
```

Now, you have both machines up and running. One host containing [tomcat](http://tomcat.apache.org/) and the other [NGINX](https://www.nginx.com/), both of them running as a systemd service. Firewalld is up and configured to accept connections on the necessary ports. You will find SELinux already enforced as well.

You can check more details about the provisioning steps just executed by taking a deep look on my [Ansible](https://www.ansible.com/) playbooks and roles here on this [repo](https://github.com/fabiogoma/selinux-nginx-tomcat).

Although it's possible to access the servers directly by IP, I chose to setup multicast DNS on both of them, which gives us a capability of reach the machines by name. From the CLI of your host you can test it, by just making an http request using curl.

```bash
$ curl tomcat.local:8080
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
...
...
...
```

```bash
$ curl nginx.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
...
...
```

During this demo you will need to ssh the two hosts, I recommend you to use vagrant to do it, otherwise you need to extract the vagrant ssh key generated for each host.

```bash
$ vagrant ssh tomcat
Last login: Tue May 29 17:41:51 2018 from 192.168.121.1
[vagrant@tomcat ~]$
```

or

```bash
$ vagrant ssh nginx
Last login: Tue May 29 17:46:59 2018 from 192.168.121.1
[vagrant@nginx ~]$
```

This is how the ansible playbooks and roles are organized:

```bash
$ tree provisioning/
provisioning/
├── nginx-playbook.retry
├── nginx-playbook.yml
├── roles
│   ├── general
│   │   ├── files
│   │   │   └── tomcat.xml
│   │   ├── handlers
│   │   │   ├── main.yml
│   │   │   └── restart-services.yml
│   │   └── tasks
│   │       ├── main.yml
│   │       ├── packages.yml
│   │       └── security.yml
│   ├── nginx
│   │   ├── files
│   │   │   ├── daphne.png
│   │   │   ├── default.conf
│   │   │   ├── fred.png
│   │   │   ├── nginx.repo
│   │   │   ├── shaggy.png
│   │   │   └── velma.png
│   │   ├── handlers
│   │   │   ├── main.yml
│   │   │   └── restart-nginx.yml
│   │   ├── tasks
│   │   │   ├── create-configuration.yml
│   │   │   ├── create-home-pages.yml
│   │   │   ├── create-users.yml
│   │   │   ├── install-nginx.yml
│   │   │   └── main.yml
│   │   └── templates
│   │       └── page.html.j2
│   └── tomcat
│       ├── files
│       │   ├── deploy
│       │   │   ├── daphne.png
│       │   │   ├── fred.png
│       │   │   ├── index.html
│       │   │   ├── shaggy.png
│       │   │   └── velma.png
│       │   └── tomcat.service
│       ├── handlers
│       │   ├── main.yml
│       │   └── restart-tomcat.yml
│       └── tasks
│           ├── create-npa.yml
│           ├── deploy-static-content.yml
│           ├── install-tomcat.yml
│           ├── main.yml
│           └── manage-service.yml
├── tomcat-playbook.retry
└── tomcat-playbook.yml

15 directories, 37 files
```

The general role will install a few packages, among others, **setroubleshoot** and **setroubleshoot-server**. This two packages provides a series of tools that helps on the troubleshooting regarding SELinux issues.

Time to get your hands dirty
----------------------------

SSH into the NGINX server, switch to root user and move the pages from users to NGINX default html folder

```bash
$ vagrant ssh nginx
[vagrant@nginx ~]$ sudo su -
[root@nginx ~]# mv /home/daphne/* /usr/share/nginx/html/
[root@nginx ~]# mv /home/fred/* /usr/share/nginx/html/
[root@nginx ~]# mv /home/shaggy/* /usr/share/nginx/html/
[root@nginx ~]# mv /home/velma/* /usr/share/nginx/html/
```

Or in a short way

```bash
[root@nginx ~]# find /home -type f \( -name "*.html" -or -name "*.png" \) -exec mv {} /usr/share/nginx/html/ \;
```

Now with files in place, open your browser and try to access one of the pages. http://nginx.local/velma.html

If you followed all the steps correctly, you should be able to see a page similar to the page below.

<p align="center">
  <img src="images/forbidden.png">
</p>

:astonished: What? Why? Ok, as an experienced engineer, let's check the permissions.

```bash
[root@nginx ~]# cd /usr/share/nginx/html/
[root@nginx html]# ls -l
total 276
-rw-r--r--. 1 root   root     537 Jul 11 13:50 50x.html
-rw-r--r--. 1 daphne daphne   155 Oct  7 19:38 daphne.html
-rw-r--r--. 1 daphne daphne 60648 Oct  7 19:38 daphne.png
-rw-r--r--. 1 fred   fred     153 Oct  7 19:38 fred.html
-rw-r--r--. 1 fred   fred   51301 Oct  7 19:38 fred.png
-rw-r--r--. 1 root   root     612 Jul 11 13:50 index.html
-rw-r--r--. 1 shaggy shaggy   155 Oct  7 19:38 shaggy.html
-rw-r--r--. 1 shaggy shaggy 68574 Oct  7 19:38 shaggy.png
-rw-r--r--. 1 velma  velma    154 Oct  7 19:38 velma.html
-rw-r--r--. 1 velma  velma  72203 Oct  7 19:38 velma.png
[root@nginx html]#
```

:smiley: Ah, that might be the issue. Let's fix it

```bash
[root@nginx html]# chown -R root. *
```

Now let's try again

<p align="center">
  <img src="images/forbidden.png">
</p>

:confounded: What can be possibly wrong? Ok, no reason for panic, here we are going to execute the first command that will use SELinux features to gives us a hint.

```bash
[root@nginx html]# ls -lZ
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 50x.html
-rw-r--r--. root root unconfined_u:object_r:user_home_t:s0 daphne.html
-rw-r--r--. root root unconfined_u:object_r:user_home_t:s0 daphne.png
-rw-r--r--. root root unconfined_u:object_r:user_home_t:s0 fred.html
-rw-r--r--. root root unconfined_u:object_r:user_home_t:s0 fred.png
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 index.html
-rw-r--r--. root root unconfined_u:object_r:user_home_t:s0 shaggy.html
-rw-r--r--. root root unconfined_u:object_r:user_home_t:s0 shaggy.png
-rw-r--r--. root root unconfined_u:object_r:user_home_t:s0 velma.html
-rw-r--r--. root root unconfined_u:object_r:user_home_t:s0 velma.png
[root@nginx html]#
```

The parameter **Z** shows us the SELinux labels for all files, this paramenter can be used not only with **ls** command, but with a myriad of commands that manipulate files and sockets (i.e: netstat, ps, id etc.). Let's now use our swiss army knives provided by the packages **setroubleshoot** and **setroubleshoot-server** installed during the provisioning step.

Because our SELinux is by default set to be enforced, every possible risk issue will generate alerts on **/var/log/audit/audit.log**, but this file is too ugly to be seen on a naked eye, let's use the **sealert** tool instead.

```bash
[root@nginx ~]# sealert -a /var/log/audit/audit.log
```

<p align="center">
  <img src="images/sealert.png">
</p>

Here is a good time to talk about SELinux booleans.

Booleans are on/off switches that allow you to enable or disable access based on the context, you can check all possible booleans available by using the command **getsebool -a**.

Back to our problem, the issue seams to be pretty clear on the log reported by sealert, it even gives you a suggestion about how to fix it.

```bash
SELinux is preventing /usr/sbin/nginx from read access on the file velma.html.

*****  Plugin catchall_boolean (89.3 confidence) suggests   ******************

If you want to allow httpd to read user content
Then you must tell SELinux about this by enabling the 'httpd_read_user_content' boolean.

Do
setsebool -P httpd_read_user_content 1

```

What the command **setsebool -P httpd_read_user_content 1** will do is basically set the the boolean to on, which can fix the problem. The downside is that nginx process will be able to read all files generated by users, which might not be what we want for now.

>NOTICE: **With SELinux disabled, there's no boolean controls, which means nginx can read even /etc/shadow if your server has any vulnerability that can be exploited. How good is that, hum?**

We could change the boolean and set it to ON, but this is not what we want, instead let's change the context of the files and make it permanent. There are a few different ways to do this, let's understand the caveats and pick the best solution.

The easiest way is to check the context of a file already in place, the one that works as expected (i.e: index.html) and use the same context for all your other files. Because we need to change the context of files let's use the command **chcon** for now:

```bash
[root@nginx html]# cd /usr/share/nginx/html
[root@nginx html]# ls -lZ index.html 
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 index.html
[root@nginx html]# chcon -t httpd_sys_content_t velma.*
[root@nginx html]# ls -lZ velma*
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 velma.html
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 velma.png
```

Then try to access on your browser http://nginx.local/velma.html
<p align="center">
  <img src="images/velma-page.png">
</p>

You could also had copied the entire SELinux configuration from index.html, you can actually do it using index.html as a **--reference** for your own files?

```bash
[root@nginx html]# chcon --reference /usr/share/nginx/html/index.html fred.*
[root@nginx html]# ls -lZ fred.*
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 fred.html
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 fred.png
[root@nginx html]#
```

Then try to access on your browser http://nginx.local/fred.html
<p align="center">
  <img src="images/fred-page.png">
</p>

By using **chcon**, you know for sure it will work, as you could see in the examples, the problem is that in case of relabeling, you could loose your customized configuration.

The recommended way is to change your policy and then restore the state from your policy to your folder. You can do this using the **semanage** and then **restorecon** commands.

```bash
[root@nginx html]# semanage fcontext -a -t httpd_sys_content_t "/usr/share/nginx/html(/.*)?"
[root@nginx html]# restorecon -Rv /usr/share/nginx/html/
restorecon reset /usr/share/nginx/html/daphne.html context unconfined_u:object_r:user_home_t:s0->unconfined_u:object_r:httpd_sys_content_t:s0
restorecon reset /usr/share/nginx/html/daphne.png context unconfined_u:object_r:user_home_t:s0->unconfined_u:object_r:httpd_sys_content_t:s0
restorecon reset /usr/share/nginx/html/fred.html context unconfined_u:object_r:user_home_t:s0->unconfined_u:object_r:httpd_sys_content_t:s0
restorecon reset /usr/share/nginx/html/fred.png context unconfined_u:object_r:user_home_t:s0->unconfined_u:object_r:httpd_sys_content_t:s0
restorecon reset /usr/share/nginx/html/shaggy.html context unconfined_u:object_r:user_home_t:s0->unconfined_u:object_r:httpd_sys_content_t:s0
restorecon reset /usr/share/nginx/html/shaggy.png context unconfined_u:object_r:user_home_t:s0->unconfined_u:object_r:httpd_sys_content_t:s0
restorecon reset /usr/share/nginx/html/velma.html context unconfined_u:object_r:user_home_t:s0->unconfined_u:object_r:httpd_sys_content_t:s0
restorecon reset /usr/share/nginx/html/velma.png context unconfined_u:object_r:user_home_t:s0->unconfined_u:object_r:httpd_sys_content_t:s0
```

You may now be able to access all of your other pages hosted on your NGINX web server.

This one http://nginx.local/daphne.html  
<p align="center">
  <img src="images/daphne-page.png">
</p>

And last but not least http://nginx.local/shaggy.html  
<p align="center">
  <img src="images/shaggy-page.png">
</p>

Communication across the network
--------------------------------

Now that we are a little bit more familiar with SELinux, I suppose we can assume that it's not that scary, right? :dragon:

Let's now move on to the next step, where we are going to access a different endpoint in our NGINX server and that request will be redirected to tomcat on a remote server.

Using your browser, try to access http://nginx.local/characters
<p align="center">
  <img src="images/proxy-pass-error.png">
</p>

Once again we get and error message instead of what we are expecting. But now we know how to debug, let's use **sealert** again to see if this is an issue related to SELinux

```bash
[root@nginx conf.d]# sealert -a /var/log/audit/audit.log
```

Once again, the logs are very clear about what's happening. SELinux is preventing NGINX daemon to remotely access another server.

```bash
SELinux is preventing /usr/sbin/nginx from name_connect access on the tcp_socket port 8080.

*****  Plugin catchall_boolean (47.5 confidence) suggests   ******************

If you want to allow httpd to can network connect
Then you must tell SELinux about this by enabling the 'httpd_can_network_connect' boolean.

Do
setsebool -P httpd_can_network_connect 1

*****  Plugin catchall_boolean (47.5 confidence) suggests   ******************

If you want to allow httpd to can network relay
Then you must tell SELinux about this by enabling the 'httpd_can_network_relay' boolean.

Do
setsebool -P httpd_can_network_relay 1

*****  Plugin catchall (6.38 confidence) suggests   **************************
```

>NOTICE: **A similar problem would happen if you were trying to access a database, for example**

Let's set the booleans suggested by SELinux and see what happens next.

```bash
[root@nginx conf.d]# setsebool -P httpd_can_network_connect 1
[root@nginx conf.d]# setsebool -P httpd_can_network_relay 1
```

Using your browser, check again if you now can access http://nginx.local/characters

Hopefully you'll see something like that
<p align="center">
  <img src="images/proxy-pass.png">
</p>

Summary
-------

During this demo, we managed to configure SELinux to allow access to local files and also to a remote component. Those are common examples that actually happen on daily basis as a sysadmin or a DevOps engineer.

If you setup your environments using any configuration management tool like Ansible, Chef, Puppet etc., you can setup all of those context and booleans in an automated way, that was not done here on purpose, to teach you how use them.

Time to go
----------

I hope by now you no longer think about disable SELinux before you give it a try. The next step is to destroy the environment and free up space on your host. If you are still logged on the VM, exit and return to your host before execute the command below.

```bash
$ vagrant destroy -f
==> nginx: Removing domain...
==> tomcat: Removing domain...
$
```

Credits
-------

[Thomas Cameron](http://people.redhat.com/tcameron/) from [Redhat](http://www.redhat.com) gave a few talks about this same subject and that was the inspiration for this demo.  
His talk is called SELinux for mere mortals and you can find the presentation [here](http://people.redhat.com/tcameron/Summit2016/selinux/selinux_for_mere_mortals.pdf)  

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/cNoVgDqqJmM/0.jpg)](https://www.youtube.com/watch?v=cNoVgDqqJmM)

©Hanna-Barbera characters is a registered trademark of the Hanna-Barbera Productions, Inc.
