# deploy_cftd_on_ubuntu

## Install Package Requirement. 

```bash
sudo apt install git python3-pip gunicorn openssl -y
```

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

### Setup Gunicorn.

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
WorkingDirectory=/path/to/CTFd
Environment="PYTHONUNBUFFERED=1"
ExecStart=/usr/bin/gunicorn 'CTFd:create_app()' -b 127.0.0.1:8000
Restart=always

[Install]
WantedBy=multi-user.target
```

Save the file and run:

```
sudo systemctl daemon-reload
sudo systemctl enable ctfd
sudo systemctl start ctfd
```

Configure Apache2: Install Apache2 and enable the required modules.
```bash 
sudo apt install apache2
sudo a2enmod proxy proxy_http headers rewrite ssl
sudo systemctl restart apache2
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
mkdir openssl
cd openssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ctfd.key -out ctfd.crt
```

### Configure Apache2 for HTTPS:

```bash
sudo nano /etc/apache2/sites-available/ctfd.conf
```

Add below HTTP in the VirtualHost section with:
```
<VirtualHost _default_:443>
    ServerName <your_domain>
    SSLEngine on
    SSLCertificateFile /path/to/CTFd/openssl/ctfd.crt
    SSLCertificateKeyFile /path/to/CTFd/openssl/ctfd.key
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>
```
Apache2 RedirectPermanent to all Requests HTTP and Respond with HTTPS by Following this configuration

```bash
sudo nano /etc/apache2/sites-available/ctfd.conf
```
```
<VirtualHost *:80>
    ServerName <your_domain>
    Redirect permanent / https://<your_domain>/
</VirtualHost>


<VirtualHost _default_:443>
    ServerName <your_domain>
    SSLEngine On
    SSLCertificateFile /path/to/CTFd/openssl/ctfd.crt
    SSLCertificateKeyFile /path/to/CTFd/openssl/ctfd.key

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

### Testing Domain in Local

```
sudo nano /etc/hosts
```

```
127.0.0.1       localhost
127.0.1.1       taro-VirtualBox
#Add your configure here
<IP Address>  <your_domain>

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
