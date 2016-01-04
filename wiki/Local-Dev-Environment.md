---
title: Local Dev Environment
---

[Tools](tools) for a developer to write and run code, usually on their local computer. For most web applications, this involves at least having access to the programming language and a database. More frequently in recent years, build tools and additional infrastructure such as message queues are needed.

There are a number of different options for running this software in a local dev environment:

* [**Native**](Native-Dev-Environment): running the software packages directly on the OS that runs the development machine. This provides the best performance, but there are likely to be significant differences between this and production servers. And many tools don't have good support for running multiple versions on the same machine. Also, it's not typical to script setup, meaning that each team member needs to install these by hand each time they get a new machine.
* [**Virtualization**](Virtualized-Dev-Environment): running the software packages on one or more virtual machines on top of the dev machine's OS. Creation and running these VMs can be scripted to ensure all developers' environments are identical to production, but it also adds complexity, and usually requires developers to have some familiarity with the virtual OS.
* **Containers**: running the software packages in containers, which are similar to virtual machines but have less overhead and can be provisioned faster. Containers can also be deployed to production, but in the absence of this it may involve a lot of learning for developers for concepts that aren't paying off for them.
* **Cloud dev environments**: running the software packages in a cloud environment targeted for development, such as [Cloud9](https://c9.io/). This can remove the need for any local setup, but it also limits the native OS tools a dev can use. It also requires providing a third party with access to your code, which may not be an option for some organizations.
