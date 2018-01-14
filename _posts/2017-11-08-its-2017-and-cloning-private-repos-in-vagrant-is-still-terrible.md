---
title: "It's 2017 and Cloning Private Repos in Vagrant is Still Terrible"
---

I've been working on a Vagrant box configured with Ansible, and I need to clone a private Bitbucket git repo inside it. This turned out to be pretty tricky! A few years ago when some coworkers of mine set up a Vagrant/Ansible box, it copied our private SSH keys into the box to accomplish this. Today, a Vagrant feature called "SSH agent forwarding" seems to be preferred. And any time I can avoid copying my private keys somewhere seems safest. Besides, I plan on using this same Ansible script to configure a cloud server, and it would be nice if that server didn't need to have my private key on it.

Here are the steps that ended up working for me. These steps should presumably work for private repos on GitHub or any other git host too. FYI, I'm using the following versions:

- macOS 10.13
- Vagrant 2.0.1
- Box `ubuntu/xenial64` -- Ubuntu 16.04.3
- Ansible 2.4.1.0
- VirtualBox 5.1.30 which probably shouldn't matter

Also, I'm using the default SSH key location `~/.ssh/id_rsa`; if you're using a different key location you may need additional steps.

## Access on Your Host

First, as a sanity check, it's a good idea to make sure you can connect to the repo on your host machine. This seems obvious, but in my case a macOS upgrade had just caused some issues with my SSH keys, and I was testing with a repo I don't work in day-to-day.

Make sure your SSH key is set up on your host. Follow the instructions provided by your git host ([GitHub instructions](https://confluence.atlassian.com/bitbucket/set-up-ssh-for-git-728138079.html), [Bitbucket instructions](https://help.github.com/articles/connecting-to-github-with-ssh/)).

To test that it's working, run `git clone` and make sure it succeeds:

```
git clone git@bitbucket.org:myuser/myrepo.git
```

## Access in Vagrant

Next, let's make sure we can access within Vagrant directly, before trying via Ansible.

There are a few steps to get this working on macOS. I'm not sure if they're all necessary, because I don't understand them well enough to be able to undo them to see if it still works! 😅 I'd recommend trying without these steps and then, if it doesn't work, add them in one at a time to see if they're needed.

Start the ssh-agent in the background:

```
eval "$(ssh-agent -s)"
```

Next, modify your `~/.ssh/config` file to automatically load keys into the ssh-agent and store passphrases in your keychain. If there are already `Host *` entries in there, you can just add another one.

```
Host *
 AddKeysToAgent yes
 UseKeychain yes
 IdentityFile ~/.ssh/id_rsa
```

Add your SSH key which is tied to your git account to the host's ssh-agent:

```
ssh-add ~/.ssh/id_rsa
```

Now, add your git server as to your known hosts. Even though we aren't _cloning_ via Ansible yet, it's a good idea to go ahead and set up this `known_hosts` entry via Ansible, so you won't forget to add it in there later:

```yml
# todo: add to the file instead of overwrite, but only if not already present
- name: set up bitbucket.org as a known host
  shell: 'ssh-keyscan -H bitbucket.org > /home/ubuntu/.ssh/known_hosts'
```

Now run `vagrant provision`. Then connect to your VM with `vagrant ssh`. The clone command should then succeed:

```
[vagrant@localhost ~]$ git clone git@bitbucket.org:myuser/myrepo.git
```

Cancel the operation and delete the cloned repo if it showed up. That way we can ensure Ansible itself can really clone it next.

## Access in Ansible

Now that you've confirmed you can clone your private repo in your Vagrant box, let's set up Ansible to be able to use the same connection.

First, in your `Vagrantfile`, configure your Ansible provisioner to use SSH agent forwarding:

```ruby
Vagrant.configure('2') do |config|
  # ...
  config.ssh.forward_agent = true
  # ...
end
```

Next, we need an Ansible task to create the directory for the repo to be checked out into. Usually the `git` command can do that, but we need to run the `git` command not as root, and so it may not have permission to create the directory.

```yml
- name: create dir for repo as sudo
  file:
    path: /path/to/myrepo
    state: directory
    owner: ubuntu
    group: ubuntu
```

Finally, add the task to clone the repo. We need to specify that the task should _not_ be run as root, so that SSH agent forwarding will work:

```yml
- name: clone repo
  become: no # to use ssh agent forwarding, must not be root
  git:
    repo: 'git@bitbucket.org:myuser/myrepo.git'
    dest: /path/to/myrepo
```

After this, run `vagrant provision` and your clone repo should succeed!

## What Didn’t Work

In case you’ve been googling around and seen some other suggestions, I want to specifically call out the things I *didn’t* need to do. I got most of these from [this SO question](https://stackoverflow.com/questions/20952689/ansible-ssh-forwarding-doesnt-seem-to-work-with-vagrant). No offense to those posters—I’m sure all these things helped at the time in their context!

I didn’t need to:

- Set `config.ssh.private_key_path = "~/.ssh/id_rsa"`
- Set `ansible.sudo = true` (it’s deprecated anyway)
- Set `ansible.host_key_checking = false`
- Set any `ansible.extra_vars`
- Set `ansible.raw_ssh_args = ['-o UserKnownHostsFile=/dev/null']`
- Create an `ansible.cfg` file

Resources that helped me get this working:

* <https://coderwall.com/p/p3bj2a/cloning-from-github-in-vagrant-using-ssh-agent-forwarding>
* <https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/>
* <https://stackoverflow.com/questions/23935138/cannot-clone-private-repo-from-vagrant-provision-file>
