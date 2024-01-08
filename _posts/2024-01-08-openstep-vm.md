---
title: Running OPENSTEP 4.2 in a Virtual Machine
---

An article on Adafruit.com describes [how to run OPENSTEP 4.2 in a virtual machine](https://learn.adafruit.com/build-your-own-next-with-a-virtual-machine/overview). I went through the steps, and it went smoothly. The only hiccup was a few settings in VirtualBox that are phrased slightly differently in 7.0 for macOS.

Here are screenshots and info about the VirtualBox 7.0 settings to use when following along with the Adafruit article. This article isn't a complete tutorial for setting up OPENSTEP 4.2 in VirtualBox, as there are a lot more steps beyond this; follow along with both articles side-by-side for the best experience.

![OPENSTEP 4.2 desktop with a File Viewer displaying the Demos folder, and BoinkOut.app selected](/img/posts/openstep-vm/00-openstep-desktop.png)

When first creating the virtual machine, the "Type" dropdown is set to "Other", and the "Version" dropdown is set to "Other/Unknown". Note that you don't specify an "ISO Image" at this point; leave it at the default "&lt;not selected&gt;".

![The VirtualBox "Create Virtual Machine" dialog, with ISO Image not selected, Type set to "Other", and Version set to "Other/Unknown"](/img/posts/openstep-vm/01-vm-name.jpg)

In the Hardware step, the "Base Memory" value is what you set to 128 MB.

![The VirtualBox "Hardware" dialog, with Base Memory set to 128 megabytes](/img/posts/openstep-vm/02-hardware.jpg)

In the Virtual Hard disk step, make sure the disk size is set to 2 GB. Note that you will only be prompted for the disk size; there is not a need to specify VDI or Dynamically allocated.

![The VirtualBox "Virtual Hard disk" dialog, with Disk Size set to 2 gigabytes](/img/posts/openstep-vm/03-virtual-hard-disk.jpg)

When you open Settings, go to Storage, and choose the Optical Drive, the setting you should change it to is now called "IDE Primary Device 1".

![The VirtualBox "Storage" settings window, with the Optical drive set to IDE Primary Device 1](/img/posts/openstep-vm/04-storage-optical.jpg)

To add a floppy disk controller, click the "Add New Controller" icon, then choose "I82078 (Floppy)".

![The VirtualBox "Storage" settings window, with a Floppy Controller added and a floppy disk image named 4.2_Install_Disk.img](/img/posts/openstep-vm/05-storage-floppy.jpg)

To deactivate USB, click the Ports icon then the USB tab, then uncheck "Enable USB Controller".

![The VirtualBox "Ports" settings window showing the USB tab, with "Enable USB Controller" unchecked](/img/posts/openstep-vm/06-ports.jpg)

This covers all the VirtualBox settings you'll need to do. If you continue along in the Adafruit tutorial, you should be set to run OPENSTEP 4.2 in a virtual machine!
