---
title: Migrating a Mastodon Account
---

The Mastodon instance I've been using, mastodon.technology, is shutting down in a few months, so I've migrated my Mastodon account to a new server. I wanted to share the steps I went through in case the details are helpful to anyone, especially others migrating off that instance--*especially* a warning about when you lose access to your old account. I don't know if these are the best steps to follow, but they worked for me.

(Note that unfortunately transferring to another server does not transfer your posts, media, favorites, mentions, etc. But you can at least back these up—see [IzzyOnDroid's "Mastodon Backups" tips](https://android.izzysoft.de/articles/named/fediverse-2#backups) for more detail.)

To migrate to a new server:

1. *Create a new Mastodon account on another instance.* You can go with one you've seen others on, search for one on [instances.social](https://instances.social/), or create your own instances. I ended up creating a small instance on [masto.host](https://masto.host/); it was extremely easy to set up.
1. *Set up your account and send a post from your new account explaining that you're moving.* This way, once you import followers, for people who require approval for follows they will see that it's a real account. As far as I can tell you need to copy over your bio, profile pic, and banner manually. Note that although you can download your old posts, I'm not aware of any way to migrate them to another account.
1. *Export the list of people you follow.* On your old account, go to Preferences > Import and export, then click the "CSV" link next to "Follows". You may want to download other lists from there too. Note that if you are following people who have configured their account to require approval of follower requests, they will need to re-approve this new request.
1. *Import the people you follow into your new account.* In your new account, go to Preferences > Import and export > Import, then import the list you downloaded. You may want to import othere lists too.
1. *Create an alias from your old to new account.* In your new account, go to Preferences > Account, then scroll down and under "Moving from a different account" click "create an account alias". Enter your old account handle and click "Create alias". Note that this doesn't change anything yet; it appears to just indicate to your old account that, once you configure it to redirect, it's allowed to redirect to this new account.
1. **STOP** and see if you have anything in your old account you will need access to, such as notifications and DMs. **Once you proceed to the next step you will no longer have access to your old account at all.**
1. *Redirect your old account to your new account.* In your old account, go to Preferences > Account, then scroll down to "Moving to a different account" and click "configure it here". Enter your new account handle and your **old** account password (because this is on the old account server) and click "Move followers". Note that if you see an error like "is not an alias of this account", make sure you completed step 4 above, then you may need to wait up to 24 hours and try again.

After this step, your old account will no longer be accessible to you, saying "Your account is inactive because it is currently redirecting to [new handle]." Your instance will begin notifying the people who follow you of your new handle, and they will be automatically switched to follow your new account. If you have enough followers, it may batch these requests up, so you may see new followers appear on your new account regularly over the next 24 hours.
