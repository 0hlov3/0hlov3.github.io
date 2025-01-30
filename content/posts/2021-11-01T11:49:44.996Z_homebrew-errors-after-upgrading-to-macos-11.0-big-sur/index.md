---
title: "Homebrew errors after upgrading to macOS 11.0 Big Sur"
author: "0hlov3"
date: 2021-11-01T11:49:44.996Z
lastmod: 2025-01-24T23:27:12Z

description: ""
subtitle: ""

image: "1.jpeg" 
images:
 - "1.jpeg"

tags: ['macOSBigSur','Homebrew','CommandLineTools','Debugging','macOSTroubleshooting']
---
{{< figure src="1.jpeg" caption="Photo by {{< newtablink \"https://unsplash.com/@cgower?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit\" >}}Christopher Gower{{< /newtablink >}} on {{< newtablink \"https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit\" >}}Unsplash{{< /newtablink >}}" alt="" >}}

Hi there,

After upgrading to macOS Big Sur, I encountered some errors while trying to use Homebrew. It turned out that I needed to 
reinstall the Command Line Tools (CLT) to get Homebrew working again. Here’s a quick walkthrough of the issue and how I resolved it.

## The Problem

When running a `brew upgrade` command for `ruby-build`, I encountered the following warning and error:

```bash
❯ brew upgrade ruby-build --fetch-HEAD
Updating Homebrew...
==> Auto-updated Homebrew!
Updated 1 tap (homebrew/core).
==> Updated Formulae
Updated 3 formulae.
```
However, I received this warning:
```plaintext
Warning: You are using macOS 11.0.
We do not provide support for this released but not yet supported version.
You will encounter build failures with some formulae.
Please create pull requests instead of asking for help on Homebrew's GitHub,
Twitter or any other official channels. You are responsible for resolving
any issues you experience while you are running this
released but not yet supported version.
```
When attempting to upgrade ruby-build, the process failed with this error:
```plaintext
==> Upgrading 1 outdated package:
ruby-build 20201005 -> 20201118
==> Upgrading ruby-build 20201005 -> 20201118
==> Downloading https://github.com/rbenv/ruby-build/archive/v20201118.tar.gz
Already downloaded: /Users/ohlove/Library/Caches/Homebrew/downloads/e8d1029ed579c311197b62dde571d68d19df7793a0119904b11b8584b62d8bfe--ruby-build-20201118.tar.gz
Error: Your CLT does not support macOS 11.0.
It is either outdated or was modified.
Please update your CLT or delete it if no updates are available.
Error: An exception occurred within a child process:
  SystemExit: exit
```

## The Solution

To resolve this issue, I reinstalled the Command Line Tools (CLT) by following these steps:

1. Remove the existing CLT installation:
```bash
sudo rm -rf /Library/Developer/CommandLineTools
```

2. Reinstall the CLT:

```bash
sudo xcode-select --install
```

## Don’t Trust Me — Seriously

The author takes no responsibility for any mishaps, broken servers, or existential crises caused by following this information.

If you spot a mistake, have a better way of doing things, or just want to chat about tech, feel free to reach out.

Also, this isn’t an ad — unless my enthusiasm and advocacy for cool stuff count as advertising.