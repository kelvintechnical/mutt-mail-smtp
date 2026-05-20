# Command-Line Email Testing (RHEL 9)
### RHCSA EX200 Lab | Part of [linux-ops-mastery](https://github.com/kelvintechnical/linux-ops-mastery)
> Use mutt and mail to test local SMTP configurations and verify messages are routed to the correct local /var/mail user spools.

![RHCSA](https://img.shields.io/badge/RHCSA-EX200-EE0000?style=flat&logo=redhat&logoColor=white)
![Topic](https://img.shields.io/badge/Topic-Remote_Administration-blue)

---

## 📋 Scenario

On **Node1**, use command-line email clients `mutt` and `mail` to test local SMTP configurations. Send messages between local users and verify they are delivered to the correct `/var/mail` spool files.

---

## 🎯 Requirements

1. Install `mutt` and `mailx` (provides the `mail` command).
2. Ensure the local MTA (`postfix`) is running.
3. Send a test email to a local user using `mail`.
4. Verify the message landed in the correct `/var/mail` spool.
5. Read the message using `mutt`.

---

## ✅ Tasks

- Install `mutt` and `mailx` via `dnf`
- Start and enable `postfix`
- Send mail to a local user with `mail`
- Verify delivery in `/var/mail/`
- Read mail interactively with `mutt`

---

## 📚 Command Decision Map

| Lab Phrase | Question Being Asked | Tool |
|------------|---------------------|------|
| "Test SMTP" | What MTA handles local mail on RHEL? | `postfix` |
| "Send from command line" | How do I send email without a GUI? | `mail -s "subject" user` |
| "Verify mail spool" | Where does local mail land? | `/var/mail/username` |
| "Read mail interactively" | What CLI mail reader has a full UI? | `mutt` |
| "Is postfix running?" | How do I check a service? | `systemctl status postfix` |

---

## 🧠 Big Concept — Local Mail on Linux

Linux systems have a built-in mail system for **local delivery** — messages between users on the same machine. This is separate from internet email. It's used by:

- **Cron jobs** — failures and output get mailed to root
- **System services** — alerts and reports go to local admin accounts
- **SMTP testing** — verifying your MTA config before pointing it at the internet

**The flow:**
```
mail command → postfix (MTA) → /var/mail/username (spool file)
```

`/var/mail/username` is a plain text file. Every message is appended to it. `mutt` and `mail` read from this file.

---

## Step 1 — Install mutt and mailx

```bash
sudo dnf install mutt mailx -y
```

> `mailx` provides the `mail` command. `mutt` is a full-featured terminal mail reader.

---

## Step 2 — Start and enable postfix

```bash
sudo systemctl enable --now postfix
```

**Verify:**
```bash
systemctl status postfix | grep Active
# Expected: Active: active (running)
```

---

## Step 3 — Send a test email to a local user

```bash
echo "This is a test message body" | mail -s "Test Subject" tom
```

> **Breaking it down:**
> - `echo "..."` — the message body piped into mail
> - `mail -s "subject"` — `-s` sets the subject line
> - `tom` — the local username to deliver to

**Send to root:**
```bash
echo "Root alert test" | mail -s "Root Test" root
```

---

## Step 4 — Verify delivery in the mail spool

```bash
sudo cat /var/mail/tom
```

**Expected output (mbox format):**
```
From root@ip-172-31-10-183 Wed May 20 12:00:00 2026
Return-Path: <root@ip-172-31-10-183>
Subject: Test Subject

This is a test message body
```

> `/var/mail/` is root-owned — use `sudo` to read another user's spool.

**Check if the file exists and has content:**
```bash
sudo ls -lh /var/mail/tom
# Non-zero file size = mail was delivered
```

---

## Step 5 — Read mail interactively with mutt

**Read tom's mail as root:**
```bash
sudo mutt -f /var/mail/tom
```

**mutt keyboard shortcuts:**

| Key | Action |
|-----|--------|
| `Enter` | Open selected message |
| `q` | Quit current view |
| `d` | Delete message |
| `r` | Reply |
| `Q` | Quit mutt entirely |

---

## Step 6 — Send multi-line mail from a file

```bash
cat > /tmp/mailbody.txt << EOF
Line 1: System check complete.
Line 2: All services nominal.
Line 3: No action required.
EOF

mail -s "System Report" root < /tmp/mailbody.txt
```

> Redirecting a file into `mail` is how cron jobs and scripts send formatted reports.

---

## 🧠 Key Concepts

| Concept | What it means |
|---------|--------------|
| `postfix` | The default MTA (Mail Transfer Agent) on RHEL 9 |
| MTA | Mail Transfer Agent — routes and delivers email messages |
| `/var/mail/username` | Local mail spool — plain text file, one per user |
| `mbox format` | Standard format for mail spools — messages appended sequentially |
| `mail -s` | Sets subject line; body comes from stdin or file redirect |
| `mutt -f /path` | Opens a specific mailbox file directly |
| Cron + mail | Failed cron jobs email output to root's local spool automatically |

---
  
## ⚠️ Pitfalls

- **Postfix not running** → mail command appears to succeed but nothing is delivered; always check `systemctl status postfix` first
- **`/var/mail/tom` doesn't exist** → file is created on first delivery; if it's missing, mail wasn't delivered
- **Reading without sudo** → `/var/mail/` is restricted; use `sudo cat` or `sudo mutt -f`
- **`mail` vs `mailx`** → on RHEL 9 the `mail` command comes from the `mailx` package — install `mailx`, not `mail`
- **EC2 outbound SMTP blocked** → port 25 is blocked by AWS; this lab only works for local delivery between users on the same instance, not external email

---

## ✅ Lab Checklist

- `mutt` and `mailx` installed ✓
- `postfix` enabled and running ✓
- `mail -s "subject" tom` sends message ✓
- `/var/mail/tom` contains the delivered message ✓
- `sudo mutt -f /var/mail/tom` reads the message ✓
- Multi-line mail sent from file with `< redirect` ✓

---

## 🔗 Related Labs

- [Command-Line Web and FTP Testing](https://github.com/kelvintechnical/elinks-lftp)
- [Network Troubleshooting with telnet and nmap](https://github.com/kelvintechnical/telnet-nmap)
- [Full RHCSA/RHCE Study Guide →](https://github.com/kelvintechnical/linux-ops-mastery)

---

## 👤 Author

**Kelvin R. Tobias** — [kelvinintech.com](https://kelvinintech.com) | [GitHub](https://github.com/kelvintechnical) | [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
