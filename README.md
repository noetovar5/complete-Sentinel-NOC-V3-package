# complete-Sentinel-NOC-V3-package
complete Sentinel NOC V3 package
Yes. I built the **complete Sentinel NOC V3 package** so you can download it, prepare an Ubuntu container, transfer the folder to Linux, run the installer, and start monitoring.

The package includes the futuristic dashboard, PHP APIs, SQLite database support, HTTP/HTTPS checks, HTTP status codes, response-time monitoring, DNS tests, TCP-port monitoring, SSL expiration monitoring, incident/recovery history, 24-check health history, bandwidth upload/download/latency/jitter testing, and a **systemd timer that performs monitoring every 60 seconds even when your browser is closed**.

### Download the complete package

[Download Tovar Tech Sentinel NOC V3 Complete Package](sandbox:/mnt/data/TovarTech-Sentinel-NOC-V3-Complete.zip)

I also included the full Ubuntu installation guide inside the ZIP:

[Download Ubuntu Deployment Guide](sandbox:/mnt/data/sentinel_noc_v3/UBUNTU-DEPLOYMENT-GUIDE.txt)

The ZIP extracts into this structure:

```text
sentinel/
│
├── index.html
├── install.sh
├── README.md
├── UBUNTU-DEPLOYMENT-GUIDE.txt
│
├── api/
│   ├── bandwidth.php
│   ├── db.php
│   └── status.php
│
├── config/
│   └── endpoints.json
│
├── data/
│   └── sentinel.db     ← created during installation
│
└── scripts/
    ├── init_db.php
    ├── monitor.php
    ├── sentinel-monitor.service
    └── sentinel-monitor.timer
```

The installer uses Apache, PHP, PHP cURL, PHP SQLite, SQLite, OpenSSL, and UFW. Ubuntu's official server documentation supports installing PHP with Apache through Ubuntu's package system, and Ubuntu provides the `php-sqlite3` package for PHP SQLite support. ([Ubuntu][1])

## Step 1 — Create your Ubuntu container

For a Proxmox container, I'd start modestly:

```text
Ubuntu Server 24.04 LTS
CPU:     1 core
RAM:     1 GB
Disk:    8–16 GB
Network: Bridged LAN connection
```

A static IP or DHCP reservation is preferable because you don't want your monitoring server's address changing.

For example:

```text
Hostname: sentinel
IP:       192.168.1.190
```

You can use whatever available address fits your network.

Once Ubuntu starts, log into the console.

Run:

```bash
hostname -I
```

You should get something similar to:

```text
192.168.1.190
```

That is the address we'll use to connect to Sentinel.

---

# Step 2 — Update Ubuntu

Run:

```bash
sudo apt update
```

Then:

```bash
sudo apt upgrade -y
```

Ubuntu's normal package-management approach is appropriate here, and we'll install Apache/PHP from the Ubuntu repositories rather than adding unnecessary third-party repositories. ([Ubuntu][1])

---

# Step 3 — Install SSH

Since you mentioned FTP, I strongly recommend **SFTP instead of traditional FTP**.

SFTP runs over SSH, encrypting your credentials and transferred files.

Install OpenSSH:

```bash
sudo apt install openssh-server -y
```

Start it:

```bash
sudo systemctl enable --now ssh
```

Verify:

```bash
sudo systemctl status ssh
```

You want to see:

```text
Active: active (running)
```

Press:

```text
q
```

to exit the status screen.

---

# Step 4 — Transfer the ZIP from Windows

Since you're on Windows, **WinSCP** is a very convenient option.

Extract:

```text
TovarTech-Sentinel-NOC-V3-Complete.zip
```

on your Windows computer first.

You'll have:

```text
sentinel
```

folder.

Open WinSCP and use:

```text
File protocol: SFTP
Host name:     192.168.1.190
Port:          22
Username:      your Ubuntu username
Password:      your Ubuntu password
```

Obviously substitute your actual server IP.

Connect.

On the Linux side, navigate to:

```text
/home/yourusername/
```

Copy the entire:

```text
sentinel
```

folder there.

You should end up with:

```text
/home/yourusername/sentinel/
```

---

# Step 5 — Verify the files

SSH into Ubuntu.

You can use Windows Terminal or PowerShell:

```bash
ssh yourusername@192.168.1.190
```

Then run:

```bash
ls ~/sentinel
```

You should see:

```text
README.md
UBUNTU-DEPLOYMENT-GUIDE.txt
api
config
data
index.html
install.sh
scripts
```

---

# Step 6 — Create the Apache destination

Run:

```bash
sudo mkdir -p /var/www/html/sentinel
```

Now copy everything:

```bash
sudo cp -R ~/sentinel/* /var/www/html/sentinel/
```

Check:

```bash
ls -la /var/www/html/sentinel
```

You should see the Sentinel files.

---

# Step 7 — Run my automatic installer

This is the part I tried to make especially easy for you.

Move into the Sentinel directory:

```bash
cd /var/www/html/sentinel
```

Make the installer executable:

```bash
sudo chmod +x install.sh
```

Now run it:

```bash
sudo ./install.sh
```

The installer automatically installs:

```text
Apache
PHP
PHP cURL
PHP SQLite
SQLite
OpenSSL
CA certificates
UFW
```

The PHP/Apache components follow Ubuntu's documented package-based installation approach. ([Ubuntu][1])

It also:

```text
Creates the SQLite database
Configures directory permissions
Creates the monitoring systemd service
Creates the monitoring systemd timer
Starts Apache
Starts Sentinel monitoring
```

---

# Step 8 — Open Sentinel

From your Windows computer, open Chrome or Edge.

Enter:

```text
http://192.168.1.190/sentinel/
```

Replace that with your Ubuntu container IP.

You should now see:

```text
SENTINEL NOC // V3
TOVAR TECH INFRASTRUCTURE OBSERVATION PLATFORM
```

and the full futuristic dashboard.

---

# Step 9 — Run your first real monitor scan

Back on Ubuntu run:

```bash
sudo -u www-data php /var/www/html/sentinel/scripts/monitor.php
```

You should start seeing output similar to:

```text
2026-08-13 22:40:01 Tovar Tech => ONLINE HTTP=200 143ms DNS=OK PORT=OPEN SSL=74
2026-08-13 22:40:02 Tovar Books => ONLINE HTTP=200 181ms DNS=OK PORT=OPEN SSL=63
2026-08-13 22:40:02 Noe Tovar => ONLINE HTTP=200 125ms DNS=OK PORT=OPEN SSL=81
```

The exact numbers will obviously vary.

This is a significant improvement over our original HTML-only monitor because the **Ubuntu server itself** performs the tests.

---

# Step 10 — Verify automatic monitoring

Sentinel is configured to run a monitor check approximately every 60 seconds.

Check the timer:

```bash
systemctl status sentinel-monitor.timer
```

You should see:

```text
Active: active (waiting)
```

You can also see scheduled timers with:

```bash
systemctl list-timers | grep sentinel
```

---

# Step 11 — Watch Sentinel working live

This is a great Linux-learning command.

Run:

```bash
journalctl -u sentinel-monitor.service -f
```

You'll see monitoring activity as it occurs.

Exit with:

```text
CTRL+C
```

To see the most recent 50 entries:

```bash
journalctl -u sentinel-monitor.service -n 50 --no-pager
```

---

# Step 12 — Add your ten websites

The configuration is intentionally simple.

Edit:

```bash
sudo nano /var/www/html/sentinel/config/endpoints.json
```

It currently contains starter entries.

You can change it to something like:

```json
[
  {
    "name": "Tovar Tech",
    "url": "https://tovartech.org",
    "port": 443,
    "expected_text": ""
  },
  {
    "name": "Tovar Books",
    "url": "https://tovarbooks.com",
    "port": 443,
    "expected_text": ""
  },
  {
    "name": "Noe Tovar",
    "url": "https://noetovar.com",
    "port": 443,
    "expected_text": ""
  },
  {
    "name": "Web Server 04",
    "url": "https://server4.example.com",
    "port": 443,
    "expected_text": ""
  }
]
```

Continue until you have your ten systems.

Save with:

```text
CTRL+O
ENTER
CTRL+X
```

Then force a new scan:

```bash
sudo -u www-data php /var/www/html/sentinel/scripts/monitor.php
```

Refresh the browser.

---

# Step 13 — Use application validation

This is one of my favorite features in V3.

Instead of merely asking whether a website responds, Sentinel can check for text that should exist on the application.

For example:

```json
{
  "name": "Tovar Books",
  "url": "https://tovarbooks.com",
  "port": 443,
  "expected_text": "Tovar Books"
}
```

Sentinel now effectively asks:

```text
Did DNS resolve?
        ↓
Did TCP 443 open?
        ↓
Did TLS/SSL work?
        ↓
Did HTTP respond?
        ↓
Did we receive HTTP 200–399?
        ↓
Did the page contain "Tovar Books"?
```

That gives you a much better indication of **application health** than a ping alone.

---

# Step 14 — View SSL certificate life

Each HTTPS endpoint now includes something like:

```text
SSL DAYS
74
```

So at a glance you can see that the certificate has approximately:

```text
74 days remaining
```

This sets us up nicely for V3.1 where I can make Sentinel automatically classify certificates:

```text
90 days     GREEN
45 days     GREEN
29 days     AMBER
14 days     AMBER
9 days      RED
0 days      CRITICAL
```

---

# Step 15 — View HTTP status

You'll now see values such as:

```text
HTTP
200
```

or potentially:

```text
301
403
404
500
503
```

This is far more useful than simply saying:

```text
ONLINE
```

For example:

```text
DNS        OK
PORT       OPEN
SSL        OK
HTTP       503
APPLICATION OFFLINE
```

That tells you the network path works but the web application is failing.

---

# Step 16 — View response time

Sentinel also records:

```text
Response
143 ms
```

So you can eventually distinguish:

```text
ONLINE + FAST

from

ONLINE + SLOW
```

For example:

```text
65 ms       Excellent
140 ms      Healthy
450 ms      Slow
1,200 ms    Warning
4,000 ms    Critical
```

We haven't yet turned those ranges into alert rules, but the underlying measurements are now there.

---

# Step 17 — Incident and recovery history

If a site changes from:

```text
ONLINE
```

to:

```text
OFFLINE
```

Sentinel records an incident.

When it recovers:

```text
OFFLINE → ONLINE
```

Sentinel records a recovery.

You'll see these in:

```text
INCIDENT / RECOVERY TIMELINE
```

That starts turning Sentinel into a real NOC console.

---

# Step 18 — Inspect the database

Your database lives here:

```text
/var/www/html/sentinel/data/sentinel.db
```

Ubuntu can install SQLite directly through its package manager, and the PHP SQLite module is also available through Ubuntu packages. ([DigitalOcean][2])

Open it:

```bash
sqlite3 /var/www/html/sentinel/data/sentinel.db
```

Then:

```sql
.tables
```

You should see:

```text
checks
events
```

View recent checks:

```sql
SELECT * FROM checks ORDER BY id DESC LIMIT 10;
```

View incidents:

```sql
SELECT * FROM events ORDER BY id DESC LIMIT 10;
```

Exit:

```text
.quit
```

You're now actually using SQL to inspect your monitoring data.

---

# Step 19 — Run the bandwidth test

From Sentinel click:

**RUN BANDWIDTH SCAN**

You'll see:

```text
DOWNLOAD
486.4 Mbps

UPLOAD
41.7 Mbps

LATENCY
17 ms

JITTER
2.2 ms
```

Remember what those numbers mean.

If Sentinel runs on a Proxmox Ubuntu container **inside your house**, you're mostly testing:

```text
Windows PC
    ↓
Local Ethernet/Wi-Fi
    ↓
Proxmox
    ↓
Ubuntu Container
```

So the result primarily represents local network performance.

If Sentinel is eventually hosted on a remote cloud server:

```text
Windows PC
    ↓
Your ISP
    ↓
Internet
    ↓
Cloud server
```

then it becomes a much closer approximation of your Internet upload/download performance.

---

# Step 20 — Check PHP

Run:

```bash
php -v
```

Then:

```bash
php -m | grep -E 'curl|sqlite'
```

You should see items such as:

```text
curl
pdo_sqlite
sqlite3
```

If SQLite isn't present:

```bash
sudo apt install php-sqlite3 -y
```

Then:

```bash
sudo systemctl restart apache2
```

The generic `php-sqlite3` package is intended to provide SQLite support for the system's default PHP version. ([Ubuntu Packages][3])

---

# Step 21 — Check Apache

Run:

```bash
sudo apache2ctl configtest
```

You want:

```text
Syntax OK
```

Then:

```bash
sudo systemctl status apache2
```

You want:

```text
Active: active (running)
```

If necessary:

```bash
sudo systemctl restart apache2
```

---

# Step 22 — Firewall

Check:

```bash
sudo ufw status
```

If it is inactive, first make sure SSH is allowed:

```bash
sudo ufw allow OpenSSH
```

Then Apache:

```bash
sudo ufw allow 'Apache Full'
```

Then:

```bash
sudo ufw enable
```

And:

```bash
sudo ufw status
```

You should see Apache and OpenSSH allowed.

Be particularly careful not to enable a firewall remotely before permitting SSH, because that could lock you out of the container.

---

# Step 23 — Important security consideration

Initially I would keep this dashboard **inside your home network**.

Something like:

```text
http://192.168.1.190/sentinel/
```

is fine for the lab.

I would **not expose Sentinel directly to the public Internet yet** because it reveals operational information about your infrastructure.

Before we make it Internet-accessible, I'd add:

```text
HTTPS
Authentication
Login screen
Session security
Rate limiting
Optional IP restrictions
```

Or we could put it behind a private VPN or zero-trust access layer.

---

## What you've actually built now

We're no longer dealing with merely:

```text
HTML page
      ↓
Check website
```

We've moved to:

```text
                    SENTINEL NOC V3
                          │
             ┌────────────┴────────────┐
             │                         │
      Browser Dashboard        Ubuntu Monitoring Engine
             │                         │
        Visualization                  │
                                       ▼
                              Every 60 seconds
                                       │
                     ┌─────────────────┼────────────────┐
                     ▼                 ▼                ▼
                    DNS              TCP              SSL
                     │                 │                │
                     └─────────────────┼────────────────┘
                                       ▼
                                    HTTP
                                       │
                                       ▼
                               Application Content
                                       │
                                       ▼
                                Response Time
                                       │
                                       ▼
                                  SQLite DB
                                       │
                         ┌─────────────┴─────────────┐
                         ▼                           ▼
                    Health History             Incidents
                         │                           │
                         └─────────────┬─────────────┘
                                       ▼
                              SENTINEL DASHBOARD
```

That is a much more realistic enterprise-monitoring architecture.

The next version I would build after you get this running is **Sentinel V3.1 Server Telemetry**. That's where we add CPU %, RAM %, disk %, Linux uptime, service state (`apache2`, `sshd`, databases), SSL warning thresholds, 7-day/30-day uptime percentages, and email alerts. After that, we can tackle Windows Server and SQL Server monitoring.

[1]: https://ubuntu.com/server/docs/how-to/web-services/install-php/?utm_source=chatgpt.com "How to install and configure PHP"
[2]: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-sqlite-on-ubuntu-20-04?utm_source=chatgpt.com "How To Install and Use SQLite on Ubuntu"
[3]: https://packages.ubuntu.com/da/questing/php-sqlite3?utm_source=chatgpt.com "Details of package php-sqlite3 in questing"
