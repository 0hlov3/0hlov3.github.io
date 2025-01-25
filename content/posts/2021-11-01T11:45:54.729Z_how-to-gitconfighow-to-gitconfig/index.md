---
title: "How to Gitconfig"
author: "0hlov3"
date: 2021-11-01T11:45:54.729Z
lastmod: 2025-01-24T23:27:13Z

description: ""
subtitle: "A practical Guide to managing multiple git Users"

image: "1.jpeg" 
images:
 - "1.jpeg"
---
{{< figure src="1.jpeg" caption="Photo by {{< newtablink \"https://unsplash.com/@synkevych?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\" >}}Roman Synkevych{{< /newtablink >}} on {{< newtablink \"https://unsplash.com/photos/black-and-white-penguin-toy-wX2L8L-fGeA?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\" >}}Unsplash{{< /newtablink >}}" alt="A figurine of an oktokat in the center, in the background a laptop with the main page of the GitHub open." >}}

When working with Git, having an organized and efficient configuration can make a world of difference. Over the past few 
days, I’ve been diving deep into my .gitconfig, tweaking it to suit both my personal and professional needs. In this article, 
I’ll walk you through the steps I took to set up a clean and flexible Git configuration, including managing multiple users, 
enabling GPG signing, and using a global .gitignore for a smoother development experience.

Whether you're just starting with Git or looking to refine your setup, these tips will help you get the most out of your workflow.

## Setting up your initial ~/.gitconfig

The first step to organizing your Git setup is creating a clean, fresh ~/.gitconfig file. Starting from scratch ensures 
that you’re working with a simple and minimal configuration before adding any customizations.

To do this, I used the following commands to set my global username and email address:

```bash
git config --global user.name "$USERNAME"
git config --global user.email "$USEREMAIL"
```

After running these commands, my initial `~/.gitconfig` looked like this:

```plaintext
[user]
	name = $USERNAME
	email = $USEREMAIL
```

This basic configuration is sufficient for general Git usage, but it has limitations. For example, if you’re working on 
projects for both personal and professional purposes, you might need to use different usernames and email addresses 
depending on the context. A single global configuration won’t cover this scenario, so we’ll explore how to manage multiple 
Git user profiles in the next section.

## Configuring Multiple Git Users

If you use Git for both personal and professional projects, you’ll likely need multiple user profiles, one for your personal 
GitHub account and another for your company account. Thankfully, Git allows us to manage multiple users by configuring 
specific profiles for different directories. Let’s walk through how to set this up.

### Step 1: Create a Base Configuration
First, start with a global user configuration. This will act as the default user profile when no other specific configuration applies:
```plaintext
[user]
    name = $USERNAME
    email = $USEREMAIL
```

### Step 2: Set Up Directory-Specific Configurations

Next, we’ll tell Git to use specific user configurations based on the directory you’re working in. For example, 
if you’re working on projects under `~/git/COMPANY1/`, Git should use the user profile defined in a separate configuration 
file, `~/.gitconfig_company1`. Similarly, for `~/git/COMPANY2/`, another specific profile will be used.

Here’s how you can set this up in your `~/.gitconfig` file

```plaintext
[user]
    name = $USERNAME
    email = $USEREMAIL
[includeIf "gitdir:~/git/COMPANY1/"]
    path = ~/.gitconfig_company1
[includeIf "gitdir:~/git/COMPANY2/"]
    path = ~/.gitconfig_company2
```

- `includeIf "gitdir:~/git/COMPANY1/"`: Tells Git to use the configuration file `~/.gitconfig_company1` for any repository 
  located in the `~/git/COMPANY1/` directory.
- `includeIf "gitdir:~/git/COMPANY2/"`: Specifies that Git should use `~/.gitconfig_company2` for repositories under `~/git/COMPANY2/`.

### Step 3: Create User-Specific Configuration Files

Now, let’s create the configuration files for the company-specific users. For the first user (COMPANY1), run the following 
command to generate `~/.gitconfig_company1`:

```plaintext
cat <<EOF > ~/.gitconfig_company1
[user]
    name = $COMPANY1_USERNAME
    email = $COMPANY1_USEREMAIL
EOF
```

Repeat the process for the second user (COMPANY2) by creating `~/.gitconfig_company2`:

```plaintext
cat <<EOF > ~/.gitconfig_company2
[user]
	name = $COMPANY2_USERNAME
	email = $COMPANY2_USEREMAIL
EOF
```

### Step 4: Verify the Configuration

At this point, you’ve successfully configured Git to use different user profiles based on the project directory. For example:
- If you work in `~/git/COMPANY1/`, Git will use the credentials from `~/.gitconfig_company1`.
- If you work in `~/git/COMPANY2/`, Git will use the credentials from `~/.gitconfig_company2`.

This setup ensures that commits and pushes in each directory are associated with the correct username and email address.

## Setting Up a Global .gitignore

To simplify your Git workflow, it’s a good idea to set up a global `.gitignore`. This file allows you to define patterns 
for files or directories that Git should ignore across all your projects, saving you from having to create project-specific 
`.gitignore` files every time.

### Step 1: Configure a Global .gitignore

You can create and configure a global `.gitignore` by running the following command:

```plaintext
git config --global core.excludesfile ~/.gitignore_global
```

This tells Git to reference the `~/.gitignore_global` file whenever it checks for files to ignore. Once configured, 
your `.gitconfig` will include the following entry:

```plaintext
[core]
    excludesfile = ~/.gitignore_global
```

### Step 2: Populate Your Global .gitignore

Next, add the patterns of files and directories you want Git to ignore to the `~/.gitignore_global` file. 
Here’s an example of a typical `.gitignore_global`:

```plaintext
# Node.js
npm-debug.log

# macOS
.DS_Store

# Windows
Thumbs.db

# WebStorm
.idea/

# vi
*~

# General logs
log/
*.log

# Swap files
.*.swp
```

This configuration helps you avoid committing unnecessary files like editor-specific directories, operating system artifacts, and temporary files.

### Step 3: Use Tools to Generate .gitignore Files
if you’re unsure of what to include in your `.gitignore`, tools like 
{{< newtablink "https://www.toptal.com/developers/gitignore/" >}}Toptal’s Gitignore Generator{{< /newtablink >}} can help you quickly create 
tailored .gitignore files for different programming languages and frameworks. Simply select the technologies you’re using, 
and it will generate a comprehensive .gitignore for you, which you can then adapt for global or project-specific use.

By setting up a global .gitignore, you’ll streamline your workflow and ensure that unnecessary files don’t clutter your repositories. 
With tools like Toptal’s Gitignore Generator, creating efficient .gitignore files becomes even easier.

## Automatically Signing Commits with Different Git Users

If you want to ensure the integrity of your commits, using GPG signing is a great practice. 
Here’s how to set up automatic commit signing for multiple Git users, step by step.

### Step 1: Generate a GPG Key

To sign commits, you’ll first need a GPG key associated with your verified email address. Generate one using the following command:

```bash
gpg --full-generate-key
```

When prompted, choose a key length of 4096 bits for added security, and set an expiration date if you prefer. Remember, 
once the key expires, you’ll need to extend its validity or generate a new one.

### Step 2: Find Your GPG Key ID

After generating your key, you can find its key ID by running:
```bash
gpg --list-secret-keys --keyid-format LONG
```
The output will look something like this:
```plaintext
sec   rsa4096/590E5507213AC509 2020-09-15 [SC]
      E163E5A1D0E6CAD4852F4FCA590E5507213AC509
uid                 [ultimate] testing (Key for Testing) <testing@git.com>
ssb   rsa4096/7AB123B711CBE904 2020-09-15 [E]
```

The key ID in this case is 590E5507213AC509. We’ll use this ID to configure Git.

### Step 3: Add the Signing Key to Your Global Git Configuration

To enable GPG signing for your global Git user, run:

```plaintext
git config --global user.signingkey 590E5507213AC509
```

This sets the signing key for the global user profile.

### Step 4: Configure Signing Keys for Multiple Users

If you have multiple Git user profiles (e.g., for work and personal projects), you’ll need to configure a signing key for each. 
For example, in the configuration file for your first company (~/.gitconfig_company1), you’d include:

```plaintext
[user]
    name = $COMPANY1_USERNAME
    email = $COMPANY1_USEREMAIL
    signingkey = 590E5507213AC509
```

Repeat this process for other user profiles (`~/.gitconfig_company2`, etc.), ensuring that each user has their own GPG key.

> Tip: For maximum security, generate a unique GPG key for each email address instead of reusing the same key across multiple profiles.


### Step 5: Finalize Your .gitconfig
After setting up your signing keys, you’ll need to update your global ~/.gitconfig to include the necessary GPG and 
credential settings. On macOS, for example, your configuration might look like this:

```plaintext
[user]
    signingkey = $KEY
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

- [gpg] program: Specifies the path to the GPG program (in this case, /usr/local/bin/gpg on macOS).
- [commit] gpgsign: Ensures that all commits are signed by default.
- [credential] helper: Configures the credential helper for storing credentials securely.

With this setup, all your commits will be signed, ensuring their authenticity. Additionally, you’ve tailored the 
configuration to handle multiple Git users, with signing keys specific to each profile. This is particularly useful for 
managing both personal and professional repositories securely and efficiently.

## Wrapping It Up

In this guide, we’ve built a robust Git configuration to streamline and enhance your workflow. Here’s what we’ve accomplished:

- Multi-User Configuration: Set up a flexible .gitconfig to seamlessly manage multiple Git user profiles for personal and professional projects.
- Global .gitignore: Created a global .gitignore file to exclude unnecessary files from being committed across all repositories.
- GPG-Signed Commits: Enabled global commit signing to ensure the authenticity and integrity of your work.

With this setup, you’re now equipped to handle diverse Git scenarios efficiently, whether you're contributing to a personal project or committing to a company repository.

You’re ready to work smarter with Git, happy coding!

## Don‘t trust me

The author is not responsible for any errors or damages resulting from the use of this information.

If you have any questions or suggestions for improvement, please feel free to reach out.
