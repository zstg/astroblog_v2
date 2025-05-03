--- 
title : "Setting up Gmail with Notmuch"
published: 2025-05-03T14:39:39+05:30 
author: ZeStig 
tags : ["Email", "Emacs", "NixOS", "Notmuch"] 
category : "Emacs"
description : "How to configure Notmuch" 
---

A simple guide to setting up Emacs as an email client for Gmail.

# Setting up Gmail with Notmuch

If you're looking to use Gmail with Notmuch in a modern email workflow—especially within Emacs—this post guides you through setting it up using [lieer](https://github.com/gauteh/lieer), a simple tool that syncs Gmail labels with Notmuch tags. We'll use `nix` to install tools in an isolated environment, and integrate everything with Emacs.

---

## Installing Notmuch and Lieer

To get started quickly, I used a temporary `nix shell`. If you're using `home-manager`, you can permanently add `notmuch` and `lieer` there.

```nix
nix shell nixpkgs#{notmuch, lieer}
```

---

## Notmuch Initial Setup

Initialize Notmuch with:

```fish
notmuch setup
```

You'll be prompted for some initial information. Here's a sample configuration:

```text
Your full name [Stig]: ZeStig
Your primary email address [stig@host]: zestig@duck.com
Additional email address [Press 'Enter' if none]:
Top-level directory of your email archive [/home/stig/mail]: /home/stig/.mail
Tags to apply to all new messages (separated by spaces) [unread inbox]: new
Tags to exclude when searching messages (separated by spaces) [deleted spam]:
```

> **Important**: Edit `~/.notmuch-config` to include the following in the `[new]` section:

```ini
[new]
tags=new
ignore=/.*[.](json|lock|bak)$/
```

---

## Setting up Lieer (`gmi`)

Create the email directory and initialize the Gmail repo:

```fish
mkdir -p ~/.mail/zestig/
notmuch new
cd ~/.mail/zestig/
gmi init zestig@duck.com
```

Apply these minor config tweaks:

```bash
gmi set --replace-slash-with-dot
gmi set --ignore-tags-local new
```

Now, start the sync process. 

**It will open a browser to authenticate with Gmail. Be sure to sign in with the correct account!**

```bash
gmi sync
```
---

## Configuring Emacs for Notmuch

### Custom `sendmail` wrapper

Create a script called `gmi-sendmail` and place it in your `$PATH`:

```bash
#!/usr/bin/env bash
MAILBOX_PATH="$HOME/.mail/zestig/"

# Read email content from stdin and send via gmi
gmi send --quiet -C "$MAILBOX_PATH" -t
```

Make it executable with `chmod +x gmi-sendmail`.

---

### Emacs Configuration
__*Admittedly this part took me way more time than it should have.*__

Emacs supports [notmuch](https://notmuchmail.org/notmuch-emacs/) very well. It is, in my humble opinion, among the best Email clients out there (along with [mu4e](https://notmuchmail.org/notmuch-emacs/)) of course.
So here's how to configure `notmuch` in Emacs:

```emacs-lisp
(defun stig/sendmail-via-gmi ()
  "Send mail using the `gmi-sendmail' shell script as the `sendmail' program."
  (let ((sendmail-program "gmi-sendmail"))
    (message-send-mail-with-sendmail)))

(use-package notmuch
  :config
  (setq sendmail-program "gmi-sendmail"
        message-send-mail-function #'stig/sendmail-via-gmi
        notmuch-fcc-dirs nil
        notmuch-always-prompt-for-sender 'nil
        notmuch-search-oldest-first nil)
  (setq notmuch-saved-searches
        '((:name "inbox"
                 :query "tag:inbox"
                 :sort-order newest-first)
          (:name "unread"
                 :query "tag:unread"
                 :sort-order newest-first))))
```

This will let you compose, send, and manage emails all within Emacs using Notmuch and Gmail.

---
