---
title:  "Homebrew errors after upgrading to macOS 11.0 Big Sur"
date:   2020-11-19 15:00:00 +0200
categories: [Mac]
tags: ["MacOS", "Apple"]
---
Hi there,

after upgrading to Big Sur i got some errors, so i got homebrew working again, after reinstalling the CommandLineTools.

```
❯ brew upgrade ruby-build --fetch-HEAD
Updating Homebrew...
==> Auto-updated Homebrew!
Updated 1 tap (homebrew/core).
==> Updated Formulae
Updated 3 formulae.

Warning: You are using macOS 11.0.
We do not provide support for this released but not yet supported version.
You will encounter build failures with some formulae.
Please create pull requests instead of asking for help on Homebrew's GitHub,
Twitter or any other official channels. You are responsible for resolving
any issues you experience while you are running this
released but not yet supported version.

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
```
sudo rm -rf /Library/Developer/CommandLineTools
sudo xcode-select --install
```
#### don't trust me...
I do not know what I am doing, so I cannot take responsibility for the accuracy of the information provided here