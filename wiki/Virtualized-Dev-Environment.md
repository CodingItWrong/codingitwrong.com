---
title: Virtualized Dev Environment
---

Running your [local dev environment](Local-Dev-Environment) on one or more virtual machines on top of the dev machine's OS.

There are at least two ways to run a virtual machine locally:

* **Directly**: Use a VM software package like [VirtualBox](https://www.virtualbox.org/) directly. This involves using a GUI to configure the VM settings, including folder mapping. For teams, this can involve a lot of manual work.
* [**Vagrant**](https://www.vagrantup.com/): This is the preferred choice for many developers. Vagrant allows you to specify VM settings in a config file commited to source control, including running provisioners.

To install software on a VM, there are a few options:

* **Hand-crafted**: Install the software packages by hand. This isn't very efficient if you have a team of developers, or if you want to keep local, staging, and production in sync.
* **Custom-scripted**: This refers to writing a custom shell script to install software. This is the easiest option when following along with instructions online for software installation. But when you have to make a small change, there's no guarantee your script can be re-run without erroring out.
* **Configuration management software**: These tools are made to allow you to control installation and configuration of software packages, and they make it easier to do things like rerun a script with minimal changes. [Ansible](http://www.ansible.com/), [Capistrano](http://capistranorb.com/), [Chef](https://www.chef.io/), and [Puppet](https://puppetlabs.com/) are common tools for this purpose.