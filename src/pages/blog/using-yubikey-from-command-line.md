---
layout: ../../layouts/PostLayout.astro
title: "Using a YubiKey from the command line"
pubDate: 2025-08-10
description: "A simple bash script to to generate OTPs from the command line using a YubiKey."
author: "Shane Pelletier"
tags: ["miscellany"]
---
I recently bought a YubiKey mostly to solve one problem: I don't want to have my phone near me when I'm working. Unfortunately my work often requires me to enter a two-factor authentication code to sign in to our various internal webpages. While this is great for security, it does mean that I always need to keep my phone with its authenticator app nearby. The webpages I sign into only support basic TOTP authentication, so I currently only use the TOTP support provided by the YubiKey. I could just use a program on my computer (or functionality built in to the password manager I use) to generate these OTP codes, but that feels a bit conterproductive toward the goal of requiring two separate factors to sign in. I plan to eventually migrate my accounts on other services to the newer protocols supported by the YubiKey which helps justify its cost a bit over another TOTP solution.

I'm trying to use more command line tools in my daily life as I'm quite comfortable on the CLI and there's something oddly satisfying about using text to talk to the computer. There do exist a couple solutions, mostly outdated, to use the YubiKey's TOTP support from the command line, but none of them really supported the features I wanted: ease of use, the ability to fuzzy-search through stored TOTP accounts, and automatic copying of the generated code to the clipboard. I eventually ended up with the following script which can be placed on the PATH and called from anywhere (note that this only works on the X Window System due to the use of `xclip`):

```bash
#!/bin/bash

OTP=$(ykman oath accounts code "$(ykman oath accounts list | fzf)" -s | tail -n 1 | tr -d '[:space:]')

if [[ -z "$OTP" ]]; then
    echo "No OTP found."
    return 1
fi

printf %s "$OTP" | tee >(xclip -sel clipboard)
```

This uses two CLI tools that need installation: [`ykman`](https://developers.yubico.com/yubikey-manager/) and [`fzf`](https://github.com/junegunn/fzf). Once those two tools are installed, there is a little bit of work needed to setup the YubiKey. This can be done using the [Yubico Authenticator](https://www.yubico.com/products/yubico-authenticator/) application (which is what I did before I realized how to do it with the CLI) or use `ykman`.

With this setup, I'm able to type `otp` into my terminal, press enter, see a list of accounts that I can search with using the fuzzy searching provided by `fzf`, select the account I wish to generate a TOTP for, and press enter. The code is printed in the terminal and automatically copied to the clipboard for easy pasting into whichever program needed it.
