---
title: Git Push-to-Deploy
---

[Git](Git) 2.4 and above have a simple mechanism that allows you to push to a server
to [deploy](Deployment) files just as you would push to a repository.

## Server Setup

This assumes your server is accessible via SSH.

* On your server, create the folder you want to deploy to, and initialize a Git repo in it:

```
mkdir my-project
cd my-project
git init
```

* Configure that Git repo to update the current branch upon deployment:

```
git config receive.denyCurrentBranch updateInstead
```

## Local Setup

* In your local project folder, add the server as a Git remote.

```
git remote add production ssh://myuser@mydomain.com/var/www/my-project
```

* To do a deployment:

```
git push deploy
```

For info on how to execute commands upon deployment, see [Pete Coop's post](https://petecoop.co.uk/blog/git-2-3-push-deploy).
