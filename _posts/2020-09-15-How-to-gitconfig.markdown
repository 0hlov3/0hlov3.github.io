---
title:  "How to Gitconfig"
date:   2020-09-15 19:00:00 +0200
categories: [0hlov3]
tags: [git]
---
In the last days i have been working on my gitconfig extensively and now i want to tell you, what i've done to get my work done.

## Setup your initial ~/.gitconfig
First of all i created a clean new ~/.gitconfig so i can start from zero.

```bash
git config --global user.name "$USERNAME"
git config --global user.email "$USEREMAIL"
```

after i did it, i get a .gitconfig looking like:

```
[user]
	name = $USERNAME
	email = $USEREMAIL
```
so i got a plain .gitconfig and it's nice, but when i am working for my company, i need to commit and push with my Name and E-Mail i use in the Company, so we will take a look to our next Step.

## Setup Users
So we need more then one git user, to get our work done, cause we have for example a private github-account and a company account.
In our test scenario, i will make a folder called git in my homedirectory. And we will configure our Git Users.

First of all, we have to tell git, wich userconfig we want to use for wich directory.

```
[user]
    name = $USERNAME
    email = $USEREMAIL
[includeIf "gitdir:~/git/COMPANY1/"]
    path = ~/.gitconfig_company1
[includeIf "gitdir:~/git/COMPANY2/"]
    path = ~/.gitconfig_company2
```

so we are telling git now, that we want generally use the user $USERNAME but if we are under the directory ~/git/COMPANY1/ we want to use the user from the gitconfig ~/.gitconfig_company1 and for the dorectory ~/git/COMPANY2/ we want to use the user from ~/.gitconfig_company2, so we have to generate our userconfigs now.

```
cat <<EOF > ~/.gitconfig_company1
[user]
	name = $COMPANY1_USERNAME
	email = $COMPANY1_USEREMAIL
EOF
```
and the second userconfig
```
cat <<EOF > ~/.gitconfig_company2
[user]
	name = $COMPANY2_USERNAME
	email = $COMPANY2_USEREMAIL
EOF
```

So we are done for now. We added different Users to different folders in our environment, so if we commit and push something in ~/git/COMPANY1/ we will commit with our user from the ~/.gitconfig_company1 

## Global .gitignore

But to make our work easier with git, we can add a global .gitignore to our gitconfig, so if we want to generate a new one, we can do it with:
```bash
git config --global core.excludesfile ~/.gitignore_global
```
now our .gitconfig has changed, there is one more entry now.
```
[user]
    name = $USERNAME
    email = $USEREMAIL
[includeIf "gitdir:~/git/COMPANY1/"]
    path = ~/.gitconfig_company1
[includeIf "gitdir:~/git/COMPANY2/"]
    path = ~/.gitconfig_company2
[core]
	excludesfile = ~/.gitignore_global
```
so for me the ~/.gitignore_global looks like that:
```
# Node
npm-debug.log
# Mac
.DS_Store
# Windows
Thumbs.db
# WebStorm
.idea/
# vi
*~
# General
log/
*.log
.*.swp
```

So for the most one, that will be enough. But i wanted to sign my work in general, so I looked for how I could realize this.

## Auto Sign with different git Users

first of all we need to get a GPG key, that matches our verified e-mail address.
```bash
gpg --full-generate-key
```
i recommend the 4096 bits long generation of a key and a expiration date for the key. But remember, if the date is reached, you have to update your key.
After we got the key, we have to check fo the keyid.

```bash
> gpg --list-secret-keys --keyid-format LONG
sec   rsa4096/590E5507213AC509 2020-09-15 [SC]
      E163E5A1D0E6CAD4852F4FCA590E5507213AC509
uid                 [ultimate] testing (Key for Testing) <testing@git.com>
ssb   rsa4096/7AB123B711CBE904 2020-09-15 [E]
```
Now we can add the ID 590E5507213AC509 to our gitconfig, for our global user, we can do it with our git command.

```bash
git config --global user.signingkey 590E5507213AC509
```

If we want to force gpgsign, we need a key for all of our three generated user. So we are going to add the key, to the other ~/.gitconfig_company$ also.
```
[user]
    name = $COMPANY1_USERNAME
	email = $COMPANY1_USEREMAIL
    signingkey = 590E5507213AC509 
```
For productive purposes i recommend to generate a gpg-key for each e-mail-adress.

So we have added our signingkeys to all of our Git Users, now we have to edit the ~/.gitconfig again, so because I am on the mac, I have to make more entries like the credential or the gpg block.

```
[user]
    name = $USERNAME
    email = $USEREMAIL
[includeIf "gitdir:~/git/COMPANY1/"]
    path = ~/.gitconfig_company1
[includeIf "gitdir:~/git/COMPANY2/"]
    path = ~/.gitconfig_company2
[gpg]
	program = /usr/local/bin/gpg
[commit]
	gpsign = true
	gpgsign = true
[credential]
	helper = osxkeychain
[core]
	excludesfile = ~/.gitignore_global
```