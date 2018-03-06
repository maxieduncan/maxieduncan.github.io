---
layout: post
category : solarnetwork
title: "SolarNetwork Development Environment"
tags : [solarnetwork]
---
{% include JB/setup %}

This post improves on an earlier one for [SolarDRAS development on OSX](http://blog.maxieduncan.co.nz/solarnetwork/2017/06/22/dras-developer-setup) and is based on improvements made to the SolarNetwork development environment over the last few months. In addition to improvements made to the vagrant developer VM, a native development environment can now be easily setup on OSX using scripts to automate the majority of the setup.

These steps are already covered quite well by the [solarnetwork-dev README](https://github.com/SolarNetwork/solarnetwork-dev) and the [Developer VM page](https://github.com/SolarNetwork/solarnetwork/wiki/Developer-VM) on the wiki; additionally detailed instructions for manually setting up an [eclipse development environment](https://github.com/SolarNetwork/solarnetwork/wiki/Eclipse-Setup-Guide) are provided. This post covers the steps I personally follow to get a development environment up and running.

## OSX
The following steps allow you to quickly get a development environment up and running natively on OSX.

### Requirements
You'll need to have the following software installed:
* [Eclipse](http://www.eclipse.org/downloads/)
* [PostgreSQL](https://www.postgresql.org/download/macosx/)
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

### Installation steps
* checkout the solarnetwork-dev project:
`git co https://github.com/SolarNetwork/solarnetwork-dev.git`
* go into the bin directory:
`cd solarnetwork-dev/bin`
* and run the ./setup.sh script to configure the eclipse workspace, initialise the DB and checkout the required projects:
`./setup.sh ~/solarnet-workspace`, feel free to replace ~/solarnet-workspace with your own custom location

You can now run Eclipse and point it to use the workspace you provided to the setup script, this is `~/solarnet-workspace` in the example provided. Once Eclipse is running jump down to the Final Steps section to begin development.

## Developer VM
The Developer VM takes longer to install but will work on Windows, Linux or OSX operating systems.

### Requirements:
You'll need to have the following software installed:
* [Vagrant](https://www.vagrantup.com/downloads.html)
* [Vagrant disksize plugin](https://github.com/sprotheroe/vagrant-disksize) - install via `vagrant plugin install vagrant-disksize`
* [Virtual Box](https://www.virtualbox.org/wiki/Downloads)
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) - optional, you can just download the project from the GitHub website

### Installation steps
* checkout the solarnetwork-dev project:
`git co https://github.com/SolarNetwork/solarnetwork-dev.git`
* go into the vagrant configuration folder for development:
`cd ~/solarnet-dev/vagrant/solarnet-dev`
* (optional) personally at this stage I like to customise the Vagrant configuration by creating a file in this folder named:  `Vagrantfile.local`
```
cat > Vagrantfile.local <<EOF
desktop_packages='virtualbox-guest-dkms virtualbox-guest-additions-iso lubuntu-desktop chromium-browser'
memory_size=4048
no_of_cpus=2
EOF
```
this snippet uses a more user friendly desktop and increases the resources assigned to the VM from the defaults provided; you can change these settings and others as documented in the [README](https://github.com/SolarNetwork/solarnetwork-dev).
* lastly initialise the Virtual Machine:
`vagrant up`

The initialisation of the VM will take quite a while and you need to wait for the entire process to complete before you try logging into the VM. Once it's finished the login details are provided in a message at the end of the terminal output.

Once you log in you can complete the Final Steps below to begin development. It's likely you'll need to restart to the VM at this point if you want to get the screen resolution to resize automatically, which can be done using: `vagrant reload`.

## Final Steps
For both the native OSX installation and the Developer VM you'll want to run through the following steps so that you can begin development:
* (Optional) install the Spring Tools plugin in Eclipse
* [Eclipse Setup](https://github.com/SolarNetwork/solarnetwork/wiki/Developer-VM#eclipse-setup) - import the Project Set File to set up the eclipse projects and OSGi target
It's important to make sure you've followed the step to launch then stop the SolarNetwork platform at this stage, this is to generate the necessary X.509 certificates and you will encounter issues trying to access the applications if you haven't done this.
* [Create a development SolarNet account](https://github.com/SolarNetwork/solarnetwork/wiki/Developer-VM#create-a-development-solarnet-account)  - create a local account for testing
* [Set up development SolarNode](https://github.com/SolarNetwork/solarnetwork/wiki/Developer-VM#set-up-development-solarnode) - create a local node for testing
