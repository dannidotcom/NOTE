
## Installation Guide for Odoo 18 on Ubuntu 24.04

## Step 1: Log in to the Ubuntu server via SSH as the root user
Connect to your server via SSH using the following command:
```bash
ssh username@IP_Address -p Port_number
# Example:
ssh root@127.0.0.1 -p 22
```

---

## Step 2: Update Packages
Update and upgrade the system packages:
```bash
sudo apt-get update
sudo apt-get upgrade -y
```

---

## Step 3: Create an Odoo user
Create a new system user `odoo18` with a home directory at `/opt/odoo18`:
```bash
sudo useradd -m -d /opt/odoo18 -U -r -s /bin/bash odoo18
```

---

## Step 4: Install Dependencies
Install Python dependencies and other required packages:
```bash
sudo apt install -y git python3-pip python-dev python3-dev libxml2-dev libxslt1-dev zlib1g-dev libsasl2-dev libldap2-dev build-essential libssl-dev libffi-dev libmysqlclient-dev libjpeg-dev libpq-dev libjpeg8-dev liblcms2-dev libblas-dev libatlas-base-dev
sudo apt install -y npm
sudo ln -s /usr/bin/nodejs /usr/bin/node
sudo npm install -g less less-plugin-clean-css
sudo apt-get install -y node-less
```

---

## Step 5: Install and Configure PostgreSQL
Install PostgreSQL and create a user `odoo18`:
```bash
sudo apt install postgresql -y
sudo su - postgres
createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo18
psql
ALTER USER odoo18 WITH SUPERUSER;
\q
exit
```

---

## Step 6: Install Wkhtmltopdf
Install the required version of Wkhtmltopdf:
```bash
sudo wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.bionic_amd64.deb
sudo dpkg -i wkhtmltox_0.12.5-1.bionic_amd64.deb
sudo apt install -f
```

---

## Step 7: Install Odoo
Switch to the `odoo18` user and clone Odoo:
```bash
sudo su - odoo18
git clone https://www.github.com/odoo/odoo --depth 1 --branch 18.0 odoo18
python3 -m venv odoo18-venv
source odoo18-venv/bin/activate
pip3 install wheel
pip3 install -r odoo18/requirements.txt
deactivate
```

---

## Step 8: Create a Directory for Third-party Addons
Create and configure the directory for custom addons:
```bash
mkdir /opt/odoo18/odoo18/custom-addons
exit
```

---

## Step 9: Create a Configuration File
Create the Odoo configuration file:
```bash
sudo nano /etc/odoo18.conf
```
Add the following content:
```ini
[options]
admin_passwd = admin_passwd
db_host = False
db_port = False
db_user = odoo18
db_password = False
addons_path = /opt/odoo18/odoo18/addons,/opt/odoo18/odoo18/custom-addons
xmlrpc_port = 8069
```

---

## Step 10: Create a Systemd Service File
Create and configure the systemd service for Odoo:
```bash
sudo nano /etc/systemd/system/odoo18.service
```
Add the following content:
```ini
[Unit]
Description=Odoo18
Requires=postgresql.service
After=network.target postgresql.service

[Service]
Type=simple
SyslogIdentifier=odoo18
PermissionsStartOnly=true
User=odoo18
Group=odoo18
ExecStart=/opt/odoo18/odoo18-venv/bin/python3 /opt/odoo18/odoo18/odoo-bin -c /etc/odoo18.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
```
Reload the systemd daemon and enable the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now odoo18
sudo systemctl status odoo18
```

---

## Step 11: Test the Odoo Installation
Open your browser and navigate to:
```
http://<your_domain_or_IP_address>:8069
```
