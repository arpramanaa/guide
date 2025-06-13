# 🚫 Fail2ban Setup & Configuration

Fail2ban is an intrusion prevention software framework that protects computer servers from brute-force attacks. This guide explains how to install, configure, and customize Fail2ban—especially for securing SSH and custom web login endpoints.

---

## 📦 Installation

### Debian/Ubuntu
```bash
sudo apt update
sudo apt install fail2ban
````

### CentOS/RHEL

```bash
sudo yum install epel-release
sudo yum install fail2ban
```

---

## ⚙️ Basic Configuration

Do **not** edit `jail.conf` directly. Instead, create or modify `/etc/fail2ban/jail.local`:

```ini
[DEFAULT]
bantime  = 3600
findtime = 600
maxretry = 5
destemail = admin@example.com
sendername = Fail2Ban Alert
mta = sendmail
action = %(action_mwl)s
```

Available actions:

| Action          | Description                                 |
| --------------- | ------------------------------------------- |
| `action_mw`     | Block IP & send WHOIS email alert           |
| `action_mwl`    | Block IP, send WHOIS & include log in email |
| `action_cf_mwl` | Custom Cloudflare integration               |

---

## 🔐 SSH Jail Configuration

```ini
[sshd]
enabled = true
port    = ssh
filter  = sshd
logpath = /var/log/auth.log
maxretry = 5
```

---

## 🌐 Web Login Protection Example

Protect a custom `/login` endpoint (e.g. in PHP, Laravel, WordPress).

### 1. Create Filter

Create `/etc/fail2ban/filter.d/web-login.conf`:

```ini
[Definition]
failregex = <HOST> -.*POST /login.* 401
ignoreregex =
```

Adjust the regex to match your login failures in the log.

---

### 2. Enable Jail

Add to `/etc/fail2ban/jail.local`:

```ini
[web-login]
enabled = true
filter  = web-login
action  = %(action_mwl)s
logpath = /var/log/nginx/access.log
maxretry = 5
findtime = 600
bantime = 3600
```

---

### 3. Example Log Line

Ensure your logs include a recognizable failure pattern like:

```
192.0.2.1 - - [13/Jun/2025:15:23:11 +0700] "POST /login HTTP/1.1" 401 198 "-" "Mozilla/5.0"
```

If your app uses a different format (e.g., `login failed` in a custom log), point `logpath` to that file and adjust `failregex` accordingly.

---

## ✅ Enable & Test

Start the service:

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Check jail status:

```bash
sudo fail2ban-client status
sudo fail2ban-client status web-login
```

Unban IP (if needed):

```bash
sudo fail2ban-client set web-login unbanip 192.0.2.1
```

---

## 📁 Directory Overview

| Path                                    | Description         |
| --------------------------------------- | ------------------- |
| `/etc/fail2ban/jail.local`              | Main configuration  |
| `/etc/fail2ban/filter.d/web-login.conf` | Custom login filter |
| `/var/log/nginx/access.log`             | Example log path    |

---
## 📜 License

MIT License

