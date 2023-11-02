# nginx-certbot-issuer
Let's Encrypt SSL Certificate issuer with Docker, Certbot and Nginx.
This project is used to issue SSL Cert, which can be installed for securing the services communication with HTTPS

Note: 
_The certificate issued is free, but we have to renew it every 3 months_

## Prerequisite
* A managed domain name
* An Accessible host machine from the internet with Docker installed

## How to issue a certificate

Following the steps below to issue the SSL certificate
1. Create DNS A Record for SSL Domain Name
    * Manage Domain > DNS/NameServer > Create A Record:
        1. Type = A
        2. Name = ${Domain Name} (This could be sub-domain name ex: admin.{example.com})
        3. TTL = 3600
        4. Data = Server IP
2. Test if host accessible
    * Run Nginx server: ```docker compose -f docker-compose-nginx.yml up nginx ```
    * Test the access: ```curl {Domain Name}``` (we can check the log on the above docker compose as well)
    * Clean the test: ```docker compose -f docker-compose-nginx.yml down```
3. Issue SSL Certificate
    
    * Create config ```.env``` file with content : (```> touch .env```)
        ```
        EMAIL=your_email@
        DOMAIN_NAME=your_domain_name      #Ex: admin.example.com
        ```
        Note: _If the host is the home machine, we need to config the home router PORT_FORWARD. And update the local ```hosts`` file with the DOMAIN_NAME ```127.0.0.1 localhost $YOUR_DOMAIN_NAME```_
        
    * Run Nginx Server on host for Lets Encrypt challenges the Domain eligible
        
        ```docker compose up -d nginx```
    * Run Certbot to issue the certificate
    
        ```docker compose up certbot```
    * Certificate will be issued
    ``` 
    Attaching to certbot
    certbot  | Saving debug log to /var/log/letsencrypt/letsencrypt.log
    certbot  | Requesting a certificate for admin.{Domain Name}
    certbot  |
    certbot  | Successfully received certificate.
    certbot  | Certificate is saved at: /etc/letsencrypt/live/admin.{Domain Name}/fullchain.pem
    certbot  | Key is saved at:         /etc/letsencrypt/live/admin.{Domain Name}/privkey.pem
    certbot  | This certificate expires on xxxx-xx-xx.
    certbot  | These files will be updated when the certificate renews.
    certbot  | NEXT STEPS:
    certbot  | - The certificate will need to be renewed before it expires. Certbot can               automatically renew the certificate in the background, but you may need to take steps to          enable that functionality. See https://certbot.org/renewal-setup for instructions.
    certbot  |
    certbot  | - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    certbot  | If you like Certbot, please consider supporting our work by:
    certbot  |  * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
    certbot  |  * Donating to EFF:                    https://eff.org/donate-le
    certbot  | - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    certbot exited with code
    ```
    * The certificates are stored in ```./certbot/conf/live/admin.{Domain Name}/```
    
5. Clean up and done 
    
    ```docker compose down```


>> Update the DOMAIN_NAME in .env file and re-run step 3 to issue others certificates


## Usage example
1. Use for k8s https ingress 
```
# Store certificate in Secret it's typically used for TLS.

kubectl create secret tls your-tls-name --key privkey.pem --cert fullchain.pem
```
2. Use for NGINX https
```
# Create https section in nginx config file
http {
    ...
    server {
        listen 443 ssl http2;
        # use the certificates
        ssl_certificate     fullchain.pem;
        ssl_certificate_key privkey.pem;
        ...
    }
}
```

## License
MIT
