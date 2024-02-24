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
Run Gunicorn: Run Gunicorn to serve the CTFd application.

```bash
gunicorn 'CTFd:create_app()' -b 127.0.0.1:8000
```
Configure Apache2: Install Apache2 and enable the required modules.
```bash
sudo apt update
sudo apt install apache2
sudo a2enmod proxy
sudo a2enmod proxy_http
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
Now, Apache2 should be proxying requests to Gunicorn, which is serving the CTFd application.
