# Load Balancer Solution With Nginx and SSL/TLS

## Configure Nginx as a Load Balancer

Create an EC2 VM based on Ubuntu Server 20.04 LTS and named it Nginx LB. Opened TCP port 80 for HTTP connections, also opened TCP port 443 – this port is used for secured HTTPS connections.

![inbound](./images/05_add_https_inbound_rule.png)

Update /etc/hosts file for local DNS with Web Servers’ names and their local IP addresses:

`sudo vi /etc/hosts`

![hosts](./images/02_etc_host.png)

Installed and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers:

`sudo apt update`
`sudo apt install nginx`
`sudo systemctl enable nginx`
`sudo systemctl start nginx`
`sudo systemctl status nginx`

![apt update](./images/03_apt_update.png)
![install nginx](./images/04_install_nginx.png)

Configured Nginx LB using Web Servers names defined in /etc/hosts:

`sudo vi /etc/nginx/sites-available/load_balancer.conf`

Insert the following:
```
upstream web { 
    server 172.31.24.238;
    server 172.31.23.240;
  }   

server {
    		listen 80; 
    		server_name <www.domain.com>;
    location / {
			proxy_set_header x-Forwarded-For $proxy_add_x_forwarded_for;
      			proxy_pass http://web; 
    }
  }
```

![configure nginx](./images/16_update_nginx_conf.png)

Restart Nginx and make sure it's up and running:

`sudo systemctl restart nginx`
`sudo systemctl status nginx`

![nginx status](./images/07_nginx_status.png)

## Register a new domain name and configure a secured connection using ssl/tls certificates

Registered a new domain www.tooling-amar.com on godaddy.com

Assign an Elastic IP to the Nginx LB server and associate the domain name with this Elastic IP address:

![allocate elastic ip](./images/08_allocate_elastic_ip.png)
![](./images/09.png)
![](./images/10.png)

Associate the domain name to the Elastic IP by creating two A records and hosted zone on route53:

![](./images/13_public_hosted_zone.png)
![](./images/14_create_a_record2.png)
![](./images/14_create_a_record3.png)

Run the command below to remove the default site so the reverse proxy will be directing to our new config file:

`sudo rm -f /etc/nginx/sites-enabled/default`

Check if nginx is successfully configured:

`sudo nginx -t`

![](./images/systanx_ok.png)

Link the load balancer config file to site enabled file so that the nginx can access the configuration file:

`cd /etc/nginx/sites-enabled/`

`sudo ln -s ../sites-available/load_balancer.conf`

![](./images/sites_enabled.png)

Restart nginx:

`sudo systemctl restart nginx`

Check the Web Server can be reached from browser using the new domain name using HTTP protocol - http://<your-domain-name.com>.

![](./images/17_check_domain_works.png)

Confirm the status of snapd service if it's up and running:

`sudo systemctl status snapd`

![](./images/18_snapd_status.png)

Install certbot and request for an SSL/TLS certificate:

`sudo apt install certbot -y`
`sudo apt install python3-certbot-nginx -y`

![](./images/19_certbot_install.png)
![](./images/19_certbot_install1.png)

Request Certificate:

`sudo certbot --nginx -d tooling-amar.com -d www.tooling-amar.com`

![](./images/20_certbot_cert.png)
![](./images/20_certbot_cert1.png)

Access the website by using HTTPS protocol:

![](./images/21_confirm_website_secure.png)

Set up periodical renewal for the SSL/TLS certificate:

`sudo certbot renew --dry-run`

![](./images/22_dry_run.png)

Edit crontab file to configure the command to run twice a day:

`crontab -e`

Add the following line:

`* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`

![](./images/23_cronttab.png)

Done



