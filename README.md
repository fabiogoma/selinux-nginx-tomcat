# Why disable SELinux when you can get a NSA level security?
Security Enhanced Linux (SELinux) is a Linux kernel security module that provides a mechanism for supporting access control security policies, including mandatory access controls (MAC).  

It was first developed and then open sourced by the National Security Agency (NSA) in 2000. You can find the current list of contributors [here](https://www.nsa.gov/what-we-do/research/selinux/contributors.shtml)  
 
Back in the early days, SELinux was just so complex that the first thing the great majority of sysadmins did, was to set it to disabled mode. Now a days you should not be doing that, even if you don't have time or availability to study it, you should at least set it to permissive, which basicaly permits everything and generate audit alerts if necessary.  

To check the current status of your SELinux
```bash
$ getenforce 
Enforcing
$
```

To set it to permissive
```bash
$ sudo setenforce 0
```

To make it permanent
```bash
$ sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g'  /etc/selinux/config
```

Now, if you have time, is open to learn and to get the benefits of what's considered a best practice in security systems, please keep reading.

# Tools used for this demo
I'm assuming you run GNU/Linux on your desktop, have installed and configured [nss-mdns](https://github.com/lathiat/nss-mdns), also have installed [vagrant](https://www.vagrantup.com/), [ansible](https://www.ansible.com/) and [virtualbox](https://www.virtualbox.org/). The other tools will be downloaded automatically by my automation scripts.  

<p align="center">
  <img src="images/logos.png">
</p>

# Topology
This is a extremely simple environment with two hosts only.  

<p align="center">
  <img src="images/topology.png">
</p>

# Bringing up your own environment
After clone my repo, navigate to the project folder and use vagrant to instantiate your own environment.  

```bash
$ git clone https://github.com/fabiogoma/selinux-nginx-tomcat.git
$ cd selinux-nginx-tomcat
$ vagrant up
```
Hopefully, after a few minutes your environment will be up, depending on your internet connection.  
After the execution, both machines will be up and with [nginx](https://www.nginx.com/) and [tomcat](http://tomcat.apache.org/) running as a systemd service. From your host you can test them both, by just making an http request using curl

```bash
$ curl tomcat.local:8080
$ curl nginx.local
```
During this demo you will need to ssh the two hosts, I recommend you to use vagrant to do it, otherwise you need to extract the vagrant ssh key generated for each host.

```bash
$ vagrant ssh nginx
```
or
```bash
$ vagrant ssh tomcat
```

This is how the ansible playbooks are organized:
```bash
$ tree provisioning/
provisioning/
├── nginx-playbook.yml
├── roles
│   ├── general
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
│   │   │   ├── tomcat.conf
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
│       │   └── tomcat.service
│       ├── handlers
│       │   ├── main.yml
│       │   └── restart-tomcat.yml
│       └── tasks
│           ├── create-npa.yml
│           ├── install-tomcat.yml
│           ├── main.yml
│           └── manage-service.yml
├── tomcat-playbook.retry
└── tomcat-playbook.yml

13 directories, 30 files
```
In the general role, a few packages are installed. Among others, **setroubleshoot** and **setroubleshoot-server**. This two packages provides a serie of tools that helps on the troubleshooting regarding SELinux issues.

# Time to get your hands dirty
SSH into the nginx server, switch to root user and move the pages from users to NGINX default html folder
```bash
$ vagrant ssh nginx
[vagrant@nginx ~]$ sudo su -
[root@nginx ~]# mv /home/daphne/* /usr/share/nginx/html/
[root@nginx ~]# mv /home/fred/* /usr/share/nginx/html/
[root@nginx ~]# mv /home/shaggy/* /usr/share/nginx/html/
[root@nginx ~]# mv /home/velma/* /usr/share/nginx/html/
```
Now with the files in place, open your browser and try to access one of the pages
http://nginx.local/velma.html  

If you followed all the steps correctly, you should be able to see a page similar to this  
<p align="center">
  <img src="images/forbidden.png">
</p>
:astonished: What? Why? Ok, as an experienced engineer, let's check the permissions  

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

Now Let's check again  
<p align="center">
  <img src="images/forbidden.png">
</p>
:confounded: What can be possibly happening?  
Ok, no reason for panic, here we are going to execute the first command that will use SELinux features to give us a hint  

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

The parameter **Z** shows us the SELinux context of all files, this paramenter can be used not only with **ls** command, but with a myriad of commands that manipulate files and sockets (i.e: netstat, ps, id etc.). Let's now use our swiss army knives provided by the packages **setroubleshoot** and **setroubleshoot-server** installed during the provision step.

Because our SELinux is by default set to be enforced, every possible risk issue will generate alerts on **/var/log/audit/audit.log**, but this file is too ugly to be seen on a naked eye, let's use the **sealert** tool instead.

```bash
[root@nginx ~]# sealert -a /var/log/audit/audit.log
```
<p align="center">
  <img src="images/sealert.png">
</p>

Here is a good time to talk about SELinux booleans.  

Booleans are on/off switches that allow you to enable or disable access based on the context, you can check all possible booleans available by using the command **getsebool -a** 

Back to our problem, the issue seams to be preety clear on the log reported by sealert, it even gives you a suggestion about how to fix it.

```bash
SELinux is preventing /usr/sbin/nginx from read access on the file velma.html.

*****  Plugin catchall_boolean (89.3 confidence) suggests   ******************

If you want to allow httpd to read user content
Then you must tell SELinux about this by enabling the 'httpd_read_user_content' boolean.

Do
setsebool -P httpd_read_user_content 1

```

What the command **setsebool -P httpd_read_user_content 1** will do is basically set the the boolean to on, this can fix the problem, but also means that nginx process will be able to read all files generated by users, which might not be what we wan't for now.

:orange_book: **With SELinux disabled, there's no boolean controls, which means nginx can read even /etc/shadow if your server has any vulnerability that can be exploited. How good is that, hum?**

Instead of setting this boolean ON, let's change the context of the files, there are two ways to do it, the easy way and the easiest way  

The easy way, make sure you are in the correct folder, check the context of one existent file like index.html for example and use the same datails for your files. We need to change the context of files and for that the command **chcon** will be used:
```bash
[root@nginx html]# cd /usr/share/nginx/html
[root@nginx html]# ls -lZ index.html 
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 index.html
[root@nginx html]# chcon -u system_u -r object_r -t httpd_sys_content_t velma.*
[root@nginx html]# ls -lZ velma*
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 velma.html
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 velma.png
```
Then try to access on your browser http://nginx.local/velma.html
<p align="center">
  <img src="images/velma-page.png">
</p>

Now the easiest way, if you know index.html already works, why not just use it as a reference for your files?  
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

Now, choose one of these two options and do the same for the rest of the files and then try to access them  

This one http://nginx.local/daphne.html  
<p align="center">
  <img src="images/daphne-page.png">
</p>

And last but not least http://nginx.local/shaggy.html  
<p align="center">
  <img src="images/shaggy-page.png">
</p>