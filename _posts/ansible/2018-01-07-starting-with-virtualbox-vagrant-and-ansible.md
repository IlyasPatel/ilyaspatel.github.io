---
layout: post
title: Starting with VirtualBox, Vagrant and Ansible
comments: true
---

<style>
    table {
        border-collapse: collapse;
        border-spacing: 0;
        border:2px solid #EEEEEE;
    }
    td {
        border:1px solid #DDDDDD;
        padding: 5px
    }
</style>

As mentioned in [my first post][my-first-blog-post], I’d like to try and build a Linux machine with Jenkins, GIT, Maven, Java and Docker.
The end goal is to run Selenium WebDriver tests in dockerised containers. I have a lot to learn but I feel this is an area I need to grow my skill set.
Also, as I work in test, I am also learning to use <a href="https://kitchen.ci" target="_blank">test-kitchen</a> to execute infrastructure code.

The book I am learning from is <a href="https://www.apress.com/gb/book/9781484216606" target="_blank">Ansible From Beginner to Pro</a>.
I did encounter a few problems whilst following the book as versions of libraries have changed but I’ve managed to get past these errors so far.

This tutorial and the next few to follow have been completed on a Mac with OSX El Capitan. The steps on a Windows machine are likely to be different.



## Goals

In this tutorial, we will:

* Install VirtualBox
* Install Vagrant
* Install Ansible
* Check it all works


## Introduction

#### VirtualBox

Virtualisation allows multiple operating systems to simultaneously share your processor resources in a safe and efficient manner.
VirtualBox is a **provider** for virtualisation and it allows your operating system to run in a special **guest** environment on top of your existing host operating system.
In my case, the **host** operating system is OSX El Capitan.

#### Vagrant

Vagrant is an open source software for creating a portable development environment within minutes. It is useful for a
development team to mimic a server’s configuration locally. It can also be useful for
testing a version of Internet Explorer on a Mac for example.

#### Ansible

Ansible is a provisioning framework used to deploy software in a consistent manner in one or more machines.
It ensures each machine is working with an identical environment.

---

#### Note:
I have used Homebrew to install the above mentioned software.
Homebrew is a package managment system that
simplifies the installation of software on the Mac operating system.

To install <a href="https://brew.sh" target="_blank">Homebrew</a>, just follow the simple instructions on their website.

---

#### Step 1 - Install VirtualBox

In terminal, type in the following command

`brew cask install VirtualBox`

---

#### Step 2 - Install Vagrant

In terminal, type in the following command

`brew cask install vagrant`

---

#### Step 3 - Install Ansible

In terminal, type in the following command

`brew install ansible`

---

#### Step 4 - Check Installations were Successful

Type in the following commands and if the version is printed, then the installation was successful.

`VBoxManage --version`

`vagrant --version`

`ansible --version`

My versions respectively are:

`5.2.6r120293`

`Vagrant 2.0.1`

`ansible 2.4.2.0`

---

#### Step 5 - Install an Ubuntu Server with Vagrant

Before we start using Ansible, we'll need an environment which we can use to develop our new infrastructure. This is where we use Vagrant.

**a.** Create a new directory which will contain all your virtual machine installations. I normally put all my projects under a directory called *development*.
Create a directory called *virtualmachines* and inside this create a directory called *ansible-playbook*:

`mkdir virtualmachines && cd virtualmachines`

`mkdir ansible-playbook && cd ansible-playbook`

On my machine, the full path looks like this:

`/Users/pateli03/development/virtualmachines/ansible-playbook`

**b.** To create the virtual machine on which you're going to install packages and configure with Ansible, run this command:

`vagrant init ubuntu/trusty64`

This will create a file called **Vagrantfile**. We will be editing this file in Step 6 but this is what the contents of the file should look like (but with a lot more comments):

{% highlight ruby %}
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
end
{% endhighlight ruby %}

**c.** Finally, get Ubuntu Trusty up and running by executing:

`vagrant up`

When you execute this command the first time, a few things happen. First, it will check if a box with the name ubuntu/trusty64
exists on your machine. If it doesn't, it will download it from Atlas, a platform to host and discover Vagrant boxes.
It is maintained by <a href="https://www.hashicorp.com" target="_blank">HashiCorp</a>, the team behind Vagrant.

This could take a few minutes as this is downloading an entire operating system.

![alt text](/vagrant_up_completed.png "vagrant up completion")

* **Once downloaded, Vagrant uses VirtualBox to create a virtual machine and boot it.**

You can check the status of this virtual machine by running the command:

`vagrant status`

You can also log in to the machine using:

`vagrant ssh`

This logs in to the machine using a known SSH key that was generated when the virtual machine was created.

To logout of this machine you can use the command `exit`

---

#### Step 6 - Hello Ansible

We need to let Vagrant know that we want to run Ansible on this virtual machine by adding some instructions
to the Vagrantfile. Go ahead and open the Vagrantfile in your favourite text editor but I recommend using `Vim` which is
a text editor on Unix-like operating system. Vim has recently become important in my day-to-day life so it is another tool
I would like to be more fluent in.

Open your Vagrantfile using this command:

`vim Vagrantfile`

Press `i` on your keyboard to *insert* text. You will see the bottom of the terminal window change to `-- INSERT --`

before the last end statement, add the following configuration:

{% highlight ruby %}
config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provisioning/playbook.yml"
end
{% endhighlight ruby %}

It should look similar to this:

![alt text](/vagrantfile_adding_ansible.png "Vagrantfile adding Ansible")


To save the changes:

press `ESC` on the keyboard to come back to command mode

then `Shift` + `:`

then `w` + `q`

then `Enter` to finish saving your changes.

(**Note**: Don't enter the + symbol)

---

#### Step 7 - Creating the playbook

An Ansible playbook is a YAML file which contains a set of instructions to manage the configuration of remote machines.
We will create a playbook now as using the same path specified in our Vagrantfile above -- in a directory called provisioning.

`mkdir provisioning && cd provisioning`

`touch playbook.yml`

`vim playbook.yml`

press `i` on the keyboard to *insert* text

Copy and paste the below configuration code which will ping you virtual machine to confirm you can connect to it.

I'll go into more detail on the configuration format in subsequent posts but be aware **YAML syntax is whitespace sensitive**.
Tabs and whitespaces mean different things. YAML uses spaces and the suggested indentation is 2 spaces.

{% highlight ruby %}
---
- hosts: all
  tasks:
    - ping:
{% endhighlight ruby %}

To save the changes:

press `ESC` on the keyboard to come back to command mode

then `Shift` + `:`

then `w` + `q`

then `Enter` to finish saving your changes.


#### Step 8 - Running the playbook

You should now be able to run Ansible on your machine. Change directory one level up to the location of your Vagrantfile:

`cd ..`

`vagrant provision`

You should see something quite similar to the image below. Notice the ping task was successfully executed.

![alt text](/vagrant_provision_ping.png "Vagrant provision")

Boom! You've just provisioned your first virtual machine using Ansible and Vagrant.

---

#### Step 9 - Let's look at VirtualBox

As mentioned earlier, VirtualBox provides us with virtualisation and although it seems like we haven't done anything with it,
let's have a look. Open the VirtualBox application (You can easily do this with Spotlight - `cmd` + `space` and type VirtualBox)


You should have something similar to this:


![alt text](/virtualbox_success.png "VirtualBox")

This is letting us know that Ubuntu Trusty is running properly. Happy days.

## Summary

VirtualBox is the software which runs the Operating System and Vagrant acts as a wrapper around VirtualBox to manage it.
Together, they can be used to create a local environment that matches your production environment or an environment to
execute Selenium Tests.

With our foundations in place - the [next post][tdd-with-test-kitchen] will focus on using test-kitchen, a test harness used for testing infrastructure code
on isolated platforms. This is quite exciting, a TDD method to design infrastructure.

<br />
#### Commands used in this post

<table>
    <colgroup>
        <col width="25%" />
        <col />
    </colgroup>
    <tr>
        <td>Command</td>
        <td>Description</td>
    </tr>
    <tr>
        <td>brew cask install VirtualBox</td>
        <td>Installs VirtualBox using Homebrew</td>
    </tr>
    <tr>
        <td>brew cask install vagrant</td>
        <td>Installs Vagrant using Homebrew</td>
    </tr>
    <tr>
        <td>brew install ansible</td>
        <td>Installs Ansible using Homebrew</td>
    </tr>
    <tr>
        <td>VBoxManage --version</td>
        <td>Version of VirtualBox installed</td>
    </tr>
    <tr>
        <td>vagrant --version</td>
        <td>Version of Vagrant installed</td>
    </tr>
    <tr>
        <td>ansible --version</td>
        <td>Version of Ansible installed</td>
    </tr>
    <tr>
        <td>vagrant init ubuntu/trusty64</td>
        <td>Initialise a virtual machine with Ubuntu Trusty</td>
    </tr>
    <tr>
        <td>vagrant up</td>
        <td>Start a virtual machine</td>
    </tr>
    <tr>
        <td>vagrant status</td>
        <td>Status of virtual machine</td>
    </tr>
    <tr>
        <td>vagrant ssh</td>
        <td>SSH into the virtual machine</td>
    </tr>
    <tr>
        <td>vagrant provision</td>
        <td>Run a playbook to provision the virtual machine</td>
    </tr>
    <tr>
        <td>vim Vagrantfile</td>
        <td>Open a file called Vagrantfile with Vim text editor</td>
    </tr>
    <tr>
        <td>touch playbook.yml</td>
        <td>Create a new file called playbook.yml</td>
    </tr>

</table>

[my-first-blog-post]: /2018/01/01/my-first-blog-post
[tdd-with-test-kitchen]: /2018/01/14/tdd-your-infrastructure-code-with-kitchen

<div id="disqus_thread"></div>
<script>
/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://ilyaspatel-github-io.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>

<script id="dsq-count-scr" src="//ilyaspatel-github-io.disqus.com/count.js" async></script>