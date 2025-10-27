
_“Vagrant is a versatile tool used by devops engineers, developers, system administrators, and tech enthusiasts to create development and testing environments. It enables building and managing virtual machines within a single streamlined workflow. By focusing on automation, it reduces environment setup time, and makes the *works on my machine* issue a thing of the past.”_

If you’re already comfortable with Vagrant basics, the official documentation serves as a thorough reference for all features.

Vagrant provides easy to configure, reproducible, and portable environments built on industry-standard technology and controlled by one consistent workflow. This helps to maximize the productivity and flexibility of you and your team.

    https://developer.hashicorp.com/vagrant/intro

This guide explains how to get Vagrant running on fedora based system. I started with minimal Fedora install to minimize the host’s memory footprint, but the steps should work on any Fedora setup, whether Server or Workstation.

1. Verify the machine supports wirtualisation:
```
$ sudo lscpu | grep Virtualization
Virtualization:                  VT-x
Virtualization type:             full
```

2. Install qemu-kvm:
```
$ sudo dnf install qemu-kvm libvirt libguestfs-tools virt-install rsync
```

3. Start and enable libvirt daemon:
```
$ sudo systemctl enable --now libvirtd
```

4. Install Vagrant:
```
$ sudo dnf install vagrant
```

5. Install libvirtd’s plugin:
```
$ sudo vagrant plugin install vagrant-libvirt
```

6. Add the box:
```
$ vagrant box add fedora/41-cloud-base --provider=libvirt
```

7. Set up environment:
```
$ mkdir vagrant-demo
$ cd vagrant-demo
$ vi Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "fedora/41-cloud-base"
end
```

8. Check the file:
```
$ vagrant status
Current machine states:

default not created (libvirt)

The Libvirt domain is not created. Run 'vagrant up' to create it.
```

9. Start the box:
```
$ vagrant up
```

10. Connect to your machine:
```
$ vagrant ssh
```

Yeahhh, that’s it, you’ve now Vagrant working on you machine.

“To stop the machine”, use vagrant halt, it simply halts the host, leaving the vm and its storage disk in place.

**Tips:**
There’s no need to download boxes ahead of time, you can define the box provider in the Vagrantfile, then Vagrant will download the box when you run vagrant up if it’s missing. The following example includes Memory and CPU configuration:
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "fedora/41-cloud-base"
  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 1
    libvirt.memory = 1024
  end
end
```

For more details on using Vagrant, creating own machines, and working with different boxes, refer to the official documentation at https://www.vagrantup.com/docs
