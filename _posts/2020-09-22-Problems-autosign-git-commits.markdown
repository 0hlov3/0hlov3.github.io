---
title:  "Problems with git commit autosigning on mac"
date:   2020-09-15 18:30:00 +0200
categories: [0hlov3]
tags: [git]
---
i have the Problem, that i get a error messahe sometimes, since i've enabled git gpgsign on mac.

## Error message
sometime the gpg-agent stucks, so i get the error message:

```bash
❯ git commit -m 'Test'
error: gpg failed to sign the data
fatal: failed to write commit object
```
## Quik fix
to quik fix this issue you can kill the gpg-agent.
```bash
pkill -9 gpg-agent && export GPG_TTY=$(tty)
```

## General fix
add the fallowing lines to your bashrc.

```bash
GPG_TTY=$(tty)
export GPG_TTY
```
