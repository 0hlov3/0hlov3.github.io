---
title: "Streamline Your SSH Workflow"
author: "0hlov3"
date: 2024-12-29T22:50:17.322Z
lastmod: 2025-01-24T23:27:36Z

description: ""
subtitle: "With KDE Plasma’s Wallet and ksshaskpass"

image: "1.jpeg" 
images:
 - "1.jpeg"
 - "2.png"
 - "3.png"
 
tags: ["Linux","Kde Plasma","Openssh","Ssh Keys","Ssh Agent"]

---
{{< figure src="1.jpeg" caption="Edited Photo — The Original Photo is by {{< newtablink \"https://unsplash.com/@hidd3n?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\" >}}Kevin Horvat{{< /newtablink >}} on {{< newtablink \"https://unsplash.com/photos/flat-screen-monitor-turned-on-Pyjp2zmxuLk?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\" >}}Unsplash{{< /newtablink >}}" alt="flat screen monitor turned-on" >}}

In the world of secure communication, {{< newtablink \"https://www.openssh.com/\" >}}SSH (Secure Shell){{< /newtablink >}} 
is a cornerstone technology, enabling encrypted connections to remote servers. For 
{{< newtablink \"https://kde.org/plasma-desktop/\" >}}KDE Plasma{{< /newtablink >}} users, managing
{{< newtablink \"https://wiki.archlinux.org/title/SSH_keys\" >}}SSH keys{{< /newtablink >}} and 
{{< newtablink \"https://www.ssh.com/academy/ssh/agent\" >}}agents{{< /newtablink >}} can be both secure and seamless, 
thanks to powerful tools like {{< newtablink \"https://invent.kde.org/plasma/ksshaskpass\" >}}ksshaskpass{{< /newtablink >}},
{{< newtablink \"https://wiki.archlinux.org/title/KDE_Wallet\" >}}KDE Wallet{{< /newtablink >}}, and the built-in SSH agent.

This article dives into the essentials of SSH management on KDE Plasma. Whether you’re new to SSH or looking to optimize your workflow, you’ll learn how to create SSH keys, understand the role of KDE Wallet, and leverage ksshaskpass to simplify passphrase handling. By the end of this guide, you’ll have a streamlined and secure SSH setup tailored for KDE Plasma.

### Prerequisites

Before diving into the details, make sure you have SSH installed on your system. On most Linux distributions, OpenSSH is the default SSH implementation and can be installed via your package manager. For example

**On Arch Linux/EndeavourOS**

 ```bash
sudo pacman -S openssh
```

**On Fedora**

 ```bash
sudo dnf install openssh
```

**On Debian/Ubuntu**

 ```bash
sudo apt install openssh-client
```

To verify the installation, run the following command

 ```bash
ssh -V
```

You should see the version of OpenSSH displayed.

### Generating a New SSH Key

An SSH key pair consists of a public and private key used for secure authentication. You can generate a new SSH key on your local machine and use it to authenticate with services like GitHub, Gtlab or Codeberg for operations over SSH, or just use it for Authenticate against your server.

#### Step 1: Open a Terminal

To begin, open your terminal application.

#### Step 2: Generate the Key

Run the following command to create a new SSH key. Replace the email address in the command with your email, preferably the one linked to your GitHub account or the service you’ll authenticate with:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

If you’re using an older system that doesn’t support the Ed25519 algorithm (like Azure Devops), you can use RSA instead:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

**What Do These Options Mean?**

- `-t ed25519` **or** `-t rsa`: Specifies the algorithm for the key pair.  
  - **Ed25519**: A newer, more secure algorithm. It is faster and generates smaller keys, making it ideal for modern systems.  
  - **RSA**: An older algorithm, still widely supported but requires longer keys (e.g., 4096 bits) to achieve comparable security to Ed25519.
- `-b 4096` (for RSA): Specifies the key size in bits. Larger keys provide stronger security but require more computational power.
- `-C "your_email@example.com"`: Adds a label to the key. This label helps identify the key later, especially useful if you manage multiple keys. While an email address is commonly used, the label can be any string that makes sense to you (e.g., "Work SSH Key" or "Username").

#### Step 3: Choose a File Location

After running the command, you’ll see the following prompt:

```bash
Enter a file in which to save the key (/home/YOU/.ssh/id_ed25519):
```

Press **Enter** to save the key to the default location (`~/.ssh/id_ed25519` for Ed25519 or `~/.ssh/id_rsa` for RSA). If you already have keys at the default location, you can create a new file name by typing a custom path, such as:

```bash
/home/YOU/.ssh/my_custom_key
```

#### Step 4: Secure Your Key with a Passphrase

Next, you’ll be prompted to set a passphrase for the private key:

```bash
Enter passphrase (empty for no passphrase):
```

For security, it’s highly recommended to set a passphrase. This ensures that even if your private key is stolen, the attacker would need to know your passphrase to use it.

After entering your passphrase, you’ll confirm it:

```bash
Enter same passphrase again:
```

#### Key Differences: Ed25519 vs. RSA

- **Algorithm Age**:  
  - RSA is a well-established algorithm introduced in 1977.  
  - Ed25519 is newer and leverages more modern cryptographic techniques, offering similar security with much smaller keys.
- **Performance**:  
  - Ed25519 is significantly faster for both signing and verification.
- **Key Size**:  
  - RSA requires larger keys (e.g., 2048–4096 bits) for strong security.  
  - Ed25519 uses fixed 256-bit keys, which are computationally lightweight and secure.
- **Compatibility**:  
  - RSA is universally supported, making it suitable for legacy systems.  
  - Ed25519 is supported on most modern systems and applications.

#### Why Use a Label?

The label (`-C` option) helps you identify the purpose of a key. This is particularly useful when you manage multiple SSH keys. For example:

- **GitHub Work Key**: `-C "work@example.com"`
- **Personal Key**: `-C "personal@example.com"`
- **Custom Label**: `-C "Project X SSH Key"`

You can view labels later to quickly recognize the key’s purpose.

### Introduction to SSH Agents

An SSH agent is a program designed to manage your SSH private keys securely and conveniently. Its primary purpose is to store decrypted private keys in memory, allowing you to authenticate multiple SSH sessions without repeatedly entering your passphrase.

#### Why Use an SSH Agent?

Private keys used for SSH authentication are typically encrypted with a passphrase for security. Each time you want to connect to a remote server or service, you’d need to decrypt the key by entering the passphrase. While this enhances security, it can become cumbersome during frequent or automated operations.

An SSH agent solves this problem by:

1. **Secure Key Storage**: It decrypts your private key once and keeps it securely in memory for the duration of your session.
2. **Convenience**: With the key loaded into the agent, you can initiate SSH connections, perform Git operations, or manage remote servers without re-entering your passphrase.

#### How It Works

1. **Key Loading**: After you start the SSH agent, you load your private key into it using the `ssh-add` command. During this step, you'll be prompted to enter your passphrase to decrypt the key.
2. **Authentication Requests**: When you connect to a server or perform an SSH operation, the agent provides the necessary credentials (your private key) automatically.
3. **Session Persistence**: The agent keeps your keys in memory until you close the agent or log out, at which point the keys are removed.

#### Example Workflow

Start the SSH agent

```javascript
eval $(ssh-agent)
```

Add your private key to the agent

```javascript
ssh-add ~/.ssh/id_ed25519
```

Connect to a remote server

```sql
ssh user@remote-server
```

You won’t need to enter your passphrase again since the agent handles the authentication.

By leveraging an SSH agent, you strike a balance between security and usability, making it an indispensable tool for anyone frequently working with SSH keys.

### What is ksshaskpass?

`ksshaskpass` is a KDE-specific SSH agent helper designed to integrate seamlessly with the KDE Plasma desktop environment. Its primary function is to provide a graphical interface for entering SSH key passphrases, enhancing the user experience for those who prefer or rely on graphical workflows.

#### Integration with KDE Plasma

Unlike the standard command-line `ssh-agent` workflow, which requires typing passphrases in a terminal, `ksshaskpass` integrates directly into KDE Plasma’s graphical environment. When an SSH key’s passphrase is needed, `ksshaskpass` triggers a pop-up dialog, allowing you to enter the passphrase in a convenient GUI prompt.

This integration offers:

- **System Notifications**: KDE Plasma can display system-wide notifications when passphrases are required.
- **Session Management**: With KDE Wallet enabled, `ksshaskpass` can securely store and manage passphrases, reducing the need for repeated entry.

#### How It Works

1. When an SSH operation requests a passphrase (e.g., connecting to a server or performing a Git operation), `ksshaskpass` intercepts the request.
2. A graphical dialog appears, prompting you to enter the passphrase.
3. Once entered, the passphrase is forwarded to the SSH agent, allowing the operation to proceed.

#### Why Use ksshaskpass?

- **Ease of Use**: For users who prefer graphical interfaces, `ksshaskpass` eliminates the need to interact with the terminal for passphrase entry.
- **Enhanced Usability**: Combined with KDE Wallet, `ksshaskpass` can securely save passphrases and automatically retrieve them during future sessions, streamlining SSH workflows.
- **Tailored for KDE**: Its tight integration with KDE Plasma makes it the ideal choice for KDE users, offering a consistent look and feel aligned with the desktop environment.

#### Benefits for KDE Users

- **User-Friendly Experience**: No need to switch to a terminal or remember `ssh-agent` commands, passphrase prompts are handled graphically.
- **Streamlined Workflow**: When paired with KDE Wallet, `ksshaskpass` reduces friction by managing passphrases automatically.
- **Secure and Reliable**: By leveraging KDE Plasma’s ecosystem, it ensures secure and robust handling of sensitive information.

For KDE Plasma users, `ksshaskpass` represents a significant improvement in usability and convenience over traditional SSH agent workflows. It is especially helpful for those new to SSH or those managing multiple keys who seek a more intuitive experience.

### Normal SSH-Agent vs. ksshaskpass

When managing SSH keys, you have two primary options: the standard SSH agent included in OpenSSH or the KDE-specific `ksshaskpass`. Both serve the same core purpose of securely managing private keys, but they differ significantly in their approach and user experience.

#### Normal SSH-Agent

The standard SSH agent is a command-line utility that comes bundled with OpenSSH. It is widely used across different environments and provides a basic, no-frills approach to key management.

- **Key Features**:  
  - Manages private keys securely by storing them in memory.  
  - Requires manual interaction for loading keys using the `ssh-add` command.  
  - Works entirely in the terminal, which can be intimidating for less experienced users.
- **Typical Workflow**:  
  - Start the agent: `eval $(ssh-agent)`   
  - Add a private key to the agent: `ssh-add ~/.ssh/id_ed25519`- Authenticate with the loaded key for SSH connections.
- **Advantages**:  
  - Universally supported across platforms.  
  - Lightweight and efficient.  
  - Excellent for power users comfortable with terminal commands.
- **Disadvantages**:  
  - No graphical interface; everything is handled via the command line.  
  - Requires manual intervention for each session unless additional automation is configured.

#### ksshaskpass

`ksshaskpass` is a KDE-specific solution that integrates deeply with the KDE Plasma desktop environment, offering a more user-friendly and graphical approach to SSH key management.

- **Key Features**:  
  - Provides a graphical interface for entering SSH key passphrases.  
  - Integrates with KDE Plasma’s notification and session management systems.  
  - Can work seamlessly with KDE Wallet to store and retrieve passphrases securely.
- **Typical Workflow**:  
  1. When a passphrase is required, `ksshaskpass` triggers a pop-up dialog.  
  2. The user enters the passphrase in the graphical prompt.  
  3. The key is loaded into the SSH agent automatically, and the session proceeds.
- **Advantages**:  
  - User-friendly GUI for passphrase management.  
  - Simplifies the process for users unfamiliar with terminal commands.  
  - Enhances workflow by automating key unlocking with KDE Wallet.
- **Disadvantages**:  
  - KDE-specific; not available outside the KDE Plasma environment.  
  - May not appeal to users who prefer command-line workflows.

#### Comparison at a Glance

| Feature     | Normal SSH-Agent                  | ksshaskpass                 |
|-------------|-----------------------------------|-----------------------------|
| Interface   | Command-line                      | Graphical (KDE Plasma)      |
| Ease of Use | Requires terminal knowledge       | Intuitive GUI for all users |
| Automation  | Manual setup required             | Automated with KDE Wallet   |
| Integration | Platform-independent              | KDE Plasma-specific         |
| User Base   | Power users, terminal enthusiasts | KDE users, GUI enthusiasts  |

For KDE Plasma users, `ksshaskpass` offers a more seamless and integrated experience, eliminating the need to interact with the terminal for SSH key management. On the other hand, the normal SSH agent is a versatile, platform-agnostic tool favored by users who prefer the command line.

### Why Use KDE Wallet?

KDE Wallet is a secure credential management system integrated into the KDE Plasma desktop environment. It provides a centralized, encrypted storage solution for sensitive information such as passwords, keys, and passphrases. When used alongside `ksshaskpass`, KDE Wallet significantly enhances the convenience and security of managing SSH key passphrases.

#### The Role of KDE Wallet

At its core, KDE Wallet is designed to securely store credentials and make them accessible only to authorized applications. In the context of SSH key management, it stores your SSH key passphrases in an encrypted format. When a passphrase is needed, KDE Wallet retrieves it and provides it to `ksshaskpass`, eliminating the need for manual entry.

#### How It Works with ksshaskpass

The integration between KDE Wallet and `ksshaskpass` is seamless:

1. **Initial Setup**: During the first SSH operation requiring a passphrase, `ksshaskpass` prompts you to enter the passphrase via a graphical dialog.
2. **Storing the Passphrase**: If KDE Wallet integration is enabled, the passphrase is securely stored in the wallet after the first use.
3. **Automatic Retrieval**: On subsequent SSH operations, `ksshaskpass` automatically fetches the passphrase from KDE Wallet without requiring user input.
4. **Session Management**: If configured correctly, KDE Wallet can unlock automatically when you log in, ensuring a smooth workflow.

#### Benefits of Using KDE Wallet

- **Encrypted Storage**:  
  Passphrases are encrypted using a robust algorithm, ensuring they are protected even if someone gains access to your system files.
- **Convenience**:  
  With KDE Wallet, you no longer need to remember or repeatedly enter passphrases. Once stored, they are retrieved automatically.
- **Automatic Unlocking**:  
  KDE Wallet can be configured to unlock automatically upon login, streamlining your workflow while maintaining security.
- **Improved Productivity**:  
  By reducing the need for repeated manual input, KDE Wallet lets you focus on your tasks without interruptions.

#### Why KDE Wallet Stands Out

KDE Wallet’s integration with KDE Plasma and its ability to work with `ksshaskpass` makes it a powerful tool for managing SSH passphrases. Compared to manually managing passphrases or relying solely on `ssh-agent`, KDE Wallet offers a superior balance of security and convenience, particularly for users in the KDE ecosystem.

For KDE Plasma users seeking a streamlined and secure SSH experience, leveraging KDE Wallet alongside `ksshaskpass` is a no-brainer. It simplifies key management, minimizes interruptions, and ensures your credentials remain safe.

### Comparison of Workflows

Managing SSH keys with KDE Wallet and `ksshaskpass` significantly simplifies the workflow compared to the traditional manual setup using `ssh-agent`. Let’s explore how these two approaches differ in terms of setup, usability, and day-to-day operations.

#### Traditional ssh-agent Workflow

The traditional `ssh-agent` approach relies on manual key management via the terminal.

1. **Starting the Agent**: First, you need to start the `ssh-agent`:
   ```bash
   eval $(ssh-agent)
   ```
2. **Adding Keys**: Add your private key to the agent:
   ```bash
   ssh-add ~/.ssh/id_ed25519
   ```
   During this step, you’ll be prompted to enter the passphrase for the key.
3. **Authentication**: Once the key is added, you can connect to remote servers or perform SSH operations:
   ```bash
   ssh user@remote-server
   ```
4. **Repetition Required**: Every time you restart your machine or the agent, you’ll need to repeat these steps unless additional automation is set up (e.g., via a `.bashrc` or `.zshrc` script).

#### KDE Wallet + ksshaskpass Workflow

When using KDE Wallet with `ksshaskpass`, the process is streamlined and integrated into KDE Plasma’s graphical environment.

1. **Initial Key Loading**: During the first SSH operation requiring your key, `ksshaskpass` triggers a graphical prompt for your passphrase.
2. **Passphrase Storage**: If KDE Wallet integration is enabled, the passphrase is securely stored in the wallet after the initial entry.
3. **Automatic Key Retrieval**: On subsequent SSH operations, `ksshaskpass` fetches the passphrase from KDE Wallet, eliminating the need for manual entry.
4. **Session Management**: KDE Wallet can unlock automatically upon login, ensuring that SSH operations are seamless right from the start of your session.

#### Configuration Examples

**Manual** `ssh-agent` **Setup**:

1. Add the following to your shell’s configuration file (e.g., `.bashrc` or `.zshrc`) to start `ssh-agent` automatically:

```bash
eval $(ssh-agent)
ssh-add ~/.ssh/id_ed25519
```

2\. Restart your shell or source the configuration file:

```bash
source ~/.bashrc
```

**KDE Wallet + ksshaskpass Setup:**

1. Install Required Tools:

Ensure `kwallet`, `kwalletmanager`, and `ksshaskpass` are installed:

```bash
sudo pacman -S --needed kwallet kwalletmanager ksshaskpass   # For Arch-based systems
sudo apt install kwallet ksshaskpass                       # For Debian-based systems
```

2\. Configure `ssh_askpass.conf`

Set the `SSH_ASKPASS` environment variable to `ksshaskpass` and enable its use:

```bash
mkdir -p ~/.config/environment.d
vim ~/.config/environment.d/ssh_askpass.conf
```

Add the following lines:

```ini
SSH_ASKPASS=/usr/bin/ksshaskpass
SSH_ASKPASS_REQUIRE=prefer
```

Restart your session (e.g., log out and log back in) for these changes to take effect.

Optional: Integration with zsh, oh-my-zsh, and Powerlevel10k

If you’ve followed the guide to set up `zsh`, `oh-my-zsh`, and `Powerlevel10k`, you can further enhance your workflow by configuring the `ssh` and `ssh-agent` plugins instead of the ~/.config/environment.d/ssh\_askpass.conf.

```bash
vim ~/.zshrc
``````ruby
plugins=(
  ssh
  ssh-agent
)

zstyle :omz:plugins:ssh-agent helper ksshaskpass
zstyle :omz:plugins:ssh-agent quiet yes
zstyle :omz:plugins:ssh-agent lazy yes

source $ZSH/oh-my-zsh.sh
```

3\. **First Time Use**

Perform an SSH operation. You will see a graphical prompt for the passphrase. Check the **“Remember password”** checkbox to store it in KDE Wallet. Future passphrase requests will be handled automatically.

3\. **Optional: Configure Git Credential Helper**

Use `ksshaskpass` to securely store Git credentials in KDE Wallet. Add this to your environment configuration

```bash
vim ~/.config/environment.d/git_askpass.conf
```

Add:

```ini
GIT_ASKPASS=/usr/bin/ksshaskpass
```

**Tip**: If `SSH_ASKPASS` is already set to `ksshaskpass`, setting `GIT_ASKPASS` explicitly is not required.

5\. **Test Your Setup**

Perform an SSH operation, and you should see a graphical prompt for the passphrase only the first time. Subsequent operations will retrieve the passphrase from KDE Wallet automatically.

#### Comparison at a Glance

| Feature                  | Traditional ssh-agent          | KDE Wallet + ksshaskpass        |
|--------------------------|--------------------------------|---------------------------------|
| Ease of Use              | Command-line, manual setup     | Graphical, automated workflow   |
| Passphrase Entry         | Repeated manual input          | One-time input, stored securely |
| Integration with Desktop | None                           | Full KDE Plasma integration     |
| Session Persistence      | Requires manual setup          | Automatic via KDE Wallet        |
| Best For                 | Terminal users, minimal setups | KDE Plasma users, GUI workflows |


#### Which Workflow is Right for You?

- Choose Traditional ssh-agent if:  
  - You prefer terminal-based workflows.  
  - You need a cross-platform solution.
- Choose KDE Wallet + ksshaskpass if:  
  - You’re using KDE Plasma.  
  - You want a streamlined, GUI-driven experience.

### Conclusion

For KDE Plasma users, managing SSH keys doesn’t have to be a tedious or repetitive task. By leveraging KDE-specific tools like KDE Wallet and `ksshaskpass`, you can enjoy a streamlined, secure, and user-friendly experience that integrates seamlessly with your desktop environment.

These tools offer significant benefits:

- **Simplified Workflow**: Forget about manual key management and terminal commands. KDE Wallet and `ksshaskpass` handle passphrases effortlessly.
- **Enhanced Security**: With encrypted storage and automatic unlocking, your credentials remain safe while being readily available when needed.
- **Desktop Integration**: Full KDE Plasma integration ensures a consistent, graphical interface that minimizes disruptions and enhances productivity.

Whether you’re new to SSH or a seasoned user, KDE Wallet and `ksshaskpass` provide a superior alternative to traditional `ssh-agent` workflows. By reducing the need for repetitive passphrase entry and offering a modern GUI approach, these tools let you focus on your work without compromising security.

Take the leap and try KDE Wallet with `ksshaskpass` today. It’s a game-changer for anyone looking to streamline their SSH workflow on KDE Plasma. Your future self will thank you!

### Sources

- **Official OpenSSH Documentation**  
  - {{< newtablink \"https://www.openssh.com/manual.html\" >}}https://www.openssh.com/manual.html{{< /newtablink >}}
  - For information about `ssh-agent`, `ssh-add`, and SSH key management.
- **KDE Wallet Documentation**  
  - {{< newtablink \"https://docs.kde.org/\" >}}https://docs.kde.org/  {{< /newtablink >}}
  - For details on configuring and using KDE Wallet.
- **ksshaskpass GitHub Repository**  
  - {{< newtablink \"https://github.com/KDE/ksshaskpass\" >}}https://github.com/KDE/ksshaskpass{{< /newtablink >}}
  - For information about `ksshaskpass` and its integration with KDE Plasma.
- **Arch Wiki — SSH Keys**
  - {{< newtablink \"https://wiki.archlinux.org/title/SSH\_keys\" >}}https://wiki.archlinux.org/title/SSH\_keys{{< /newtablink >}}
  - For guidance on generating and managing SSH keys.
- **Arch Wiki — KDE Wallet**
  - {{< newtablink \"https://wiki.archlinux.org/title/KDE\_Wallet\" >}}https://wiki.archlinux.org/title/KDE\_Wallet{{< /newtablink >}}
  - For steps to configure KDE Wallet for SSH key management.
- **GitHub Docs — Connecting with SSH**
  - {{< newtablink \"https://docs.github.com/en/authentication/connecting-to-github-with-ssh\" >}}https://docs.github.com/en/authentication/connecting-to-github-with-ssh{{< /newtablink >}}
  - For instructions on generating SSH keys and adding them to GitHub.
- **Man Pages**  
  `man ssh-agent`, `man ssh-add`, `man ssh-keygen`— For detailed command usage and options.

## Don’t Trust Me — Seriously

The author takes no responsibility for any mishaps, broken servers, or existential crises caused by following this information.

If you spot a mistake, have a better way of doing things, or just want to chat about tech, feel free to reach out.

Also, this isn’t an ad — unless my enthusiasm and advocacy for cool stuff count as advertising.