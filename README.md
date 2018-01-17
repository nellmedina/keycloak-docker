
#### Install Keycloak on Nginx proxy server secured by Https

1. Create the volumes

```bash
docker volume create keycloak_volume
docker volume create keycloak_postgresql_volume
```

2. Run `docker-compose up`

3. Then add a new socket-binding element to the socket-binding-group element of Keycloak `standalone.xml`

```xml
<socket-binding-group name="standard-sockets" default-interface="public"
    port-offset="${jboss.socket.binding.port-offset:0}">
    ...
    <socket-binding name="proxy-https" port="443"/>
    ...
</socket-binding-group>
```

4. Restart the keycloak server by `docker restart keycloak_server`