
# Load Balancer Solution With Nginx and SSL/TLS
### Task
Our task in this project is to configure Nginx as a Load Balancer and register a new domain configured for SSL/TSL connection.
## Part 1 - Configure Nginx As A Load Balancer
- Created an EC2 VM based on Ubuntu Server 20.04 LTS and named it  `Nginx LB`
- Opened port 443 for inbound connections for HTTPS
- Updated the  `/etc/hosts` file for local DNS with the Web Servers
- Installed and configured Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

```
sudo apt update
sudo apt install nginx
```
```
sudo vi /etc/nginx/nginx.conf

#inserted following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;
```
- Restarted Ngnix and made sure the service was up and running
```
sudo systemctl restart nginx
sudo systemctl status nginx
```
![nginx status](https://user-images.githubusercontent.com/20668013/123494639-78d95400-d618-11eb-835d-a52328682f5b.JPG)
## Part 2 - Register a new domain name and configure secured connection using SSL/TLS certificates

- Registered a new domain with NoIP.com `dotdev.ddns.net`  
- Updated the A record in domain registrar to pint to Nginx LB
- Checked that webserver can be reached from browser using domain name `http://dotdev.ddns.net`  
![dotdev ddns](https://user-images.githubusercontent.com/20668013/123495053-ffdafc00-d619-11eb-9fa8-a341a1c1eaa9.JPG)
- Configured Nginx to recognize new domain name by updating `nginx.conf` with domain name.
-  Installed certbot and requested for an SSL/TSL certificate
-  Made sure `snapd` service is up and running
```
sudo systemctl status snapd
```
```
sudo snap install --classic certbot
```
- Requested for ssl certificate 
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```
![cert](https://user-images.githubusercontent.com/20668013/123495532-023e5580-d61c-11eb-990e-25bae7b0aef2.JPG)
- Tried accessing the site using https  
![ssl-working](https://user-images.githubusercontent.com/20668013/123495587-46315a80-d61c-11eb-95ef-fdaf84aec350.JPG)
- Set up periodic renewal of SSL/TLS certificate
```
sudo certbot renew --dry-run
```
![ssl-renewal](https://user-images.githubusercontent.com/20668013/123495671-90b2d700-d61c-11eb-9e4e-84583ecdc65d.JPG)

- Created a cronjob to run renew command  
```
crontab -e
```
```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```
