# docker-nginx-proxy-template

Nginx Reverse Proxy Template in Docker With Self Signed Certificates

```
version: '3.3'
services:
  nginx:
    container_name: nginx
    image: nginx:latest
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./hosts:/etc/nginx/conf.d
      - ./certs:/etc/ssl/private
    networks:
      - nginx-public

networks:
  nginx-public:
    external: true
```

# Usage

```
docker network create nginx-public
docker-compose up -d
```

./hosts/app1.conf
```
server {
        listen 80;
        listen [::]:80;
        server_name app1.example.com;
        location / {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_pass http://<container_name>;
        }
}
```

./hosts/app2.conf
```
server {
        server_name app2.example.com;
        location / {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_pass http://<container_name>;
        }

        listen [::]:443 ssl ipv6only=on;
        listen 443 ssl;
        ssl_certificate /etc/ssl/private/cert.pem;
        ssl_certificate_key /etc/ssl/private/key.pem;
}

server {
        if ($host = app2.example.com) {
        return 301 https://$host$request_uri;
        }
        listen 80;
        listen [::]:80;
        server_name app2.example.com;
        return 404;
}
```

# Self-signed Wildcard Certificates
```
cd certs
cp /usr/lib/ssl/openssl.cnf .
```

Edit the following:
```
[ v3_req ]
 
# Extensions to add to a certificate request
 
basicConstraints = CA:TRUE # TRUE if you want to use it on Android as well, else FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
```
Create a new section
```
[alt_names]
DNS.1 = example.com
DNS.2 = *.example.com
```

```
openssl genrsa -out hostname.key 2048
openssl rsa -in hostname.key -out hostname-key.pem
openssl req -new -key hostname-key.pem -out hostname-request.csr
openssl x509 -req -extensions v3_req -days 14600 -in hostname-request.csr -signkey hostname-key.pem -out cert.pem -extfile openssl.cnf
```

```
docker restart nginx
```


