# deploy_cftd_on_ubuntu

## Deploy CTFd with Gunicorn + Apache2 on Ubuntu. 

### Clone CTFd Repository: First, clone the CTFd repository from GitHub.
```bash
git clone https://github.com/CTFd/CTFd.git
```
Install Required Packages: Install required packages using pip.
```bash
cd CTFd
pip install -r requirements.txt
```
### Configure CTFd: (Optional)
Configure CTFd as per your requirements. This includes setting up the database and other settings in CTFd/config.py  

### Setup Gunicorn: Install Gunicorn using pip.

```bash
pip install gunicorn
```
or install directly:
```bash
sudo apt install gunicorn
```

Run Gunicorn: Run Gunicorn to serve the CTFd application.

```bash
gunicorn 'CTFd:create_app()' -b 127.0.0.1:8000
```

### Create a systemd service for Gunicorn:
```bash 
sudo nano /etc/systemd/system/ctfd.service
```
Paste the following, replacing <your_username> with your username:

```bash 
[Unit]
Description=CTFd web application
After=network.target

[Service]
User=<your_username>
Group=<your_username>
WorkingDirectory=/home/<your_username>/CTFd
Environment="PYTHONUNBUFFERED=1"
ExecStart=/usr/bin/gunicorn 'CTFd:create_app()' -b 127.0.0.1:8000
Restart=always

[Install]
WantedBy=multi-user.target
```

Configure Apache2: Install Apache2 and enable the required modules.
```bash 
sudo apt update
sudo apt install apache2
sudo a2enmod proxy proxy_http headers rewrite
```
Create Apache2 Config: Create a new Apache2 configuration file for CTFd.
```bash
sudo nano /etc/apache2/sites-available/ctfd.conf
```
Add the following configuration, replacing example.com with your domain and 8000 with the port you used for Gunicorn:
```bash
<VirtualHost *:80>
    ServerName example.com
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/
</VirtualHost>
```
Save and close the file.

Enable the Site: Enable the new site and restart Apache2.
```bash
sudo a2ensite ctfd.conf
sudo systemctl restart apache2
```
## Create a self-certificate:

```bash 
sudo apt install openssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ctfd.key -out ctfd.crt
```

Configure Apache2 for HTTPS:

```bash
sudo nano /etc/apache2/sites-available/ctfd.conf
```

Add below HTTP in the VirtualHost section with:
```
<VirtualHost *:443>
    ServerName <your_domain>
    SSLEngine on
    SSLCertificateFile /path/to/ctfd.crt
    SSLCertificateKeyFile /path/to/ctfd.key
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>
```

Restart Apache2 and access your CTFd instance using HTTPS:

```
sudo systemctl restart apache2
```

Now, Apache2 should be proxying requests to Gunicorn, which is serving the CTFd application.
