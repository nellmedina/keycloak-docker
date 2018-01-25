
### Install Keycloak on Nginx proxy server with Https

1. Create the volumes

```bash
docker volume create nginx_volume
docker volume create keycloak_volume
docker volume create keycloak_postgresql_volume
```

2. Run `docker-compose up`

3. Inside the keycloak container, add a new socket-binding element to the socket-binding-group element of Keycloak `standalone.xml`

```xml
<socket-binding-group name="standard-sockets" default-interface="public"
    port-offset="${jboss.socket.binding.port-offset:0}">
    ...
    <socket-binding name="proxy-https" port="443"/>
    ...
</socket-binding-group>

```

4. Inside the postgres container, execute this SQL to allow https.

```bash
psql -d mydb -U myuser

update REALM set ssl_required='NONE' where id = 'master';
```

5. The `blacklabelops/nginx` creates the nginx configuration automatically which will need editing to make proxy work for keycloak.

```bash
# go to /etc/nginx/conf.d/server1.conf and add an upstream just above the server block
upstream keycloak.nellmedina.com {
     server keycloak:8080;
}

# go to /etc/nginx/conf.d/server1/reverseProxy.conf
location / {
          proxy_pass http://keycloak.nellmedina.com;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $remote_addr;
          proxy_set_header X-Forwarded-Proto https;
          proxy_set_header X-Forwarded-Port 443;
        }
``` 

6. Restart the nginx and keycloak server then access keycloak at https://keycloak.nellmedina.com.