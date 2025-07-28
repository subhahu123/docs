# Updating and Installing Self-Hosted UniFi Network Servers (Linux)

```
sudo apt install docker-compose
```

https://gist.github.com/linucksrox/8d994f8b53070978a0c4842ac4964f07

`docker-compose.yml`

```yaml
version: '3.7'

services:

  unifi-db:
    image: mongo:4.4.29
    container_name: unifi-db
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: GETYOUROWNPASSWORD
      MONGO_USER: unifi
      MONGO_PASS: GETYOUROWNPASSWORD
      MONGO_DBNAME: unifi
      MONGO_AUTHSOURCE: admin
    volumes:
      - /data/mongo:/data/db
      - ./init-mongo.sh:/docker-entrypoint-initdb.d/init-mongo.sh:ro
    restart: unless-stopped

  unifi-network-application:
    image: lscr.io/linuxserver/unifi-network-application:latest
    container_name: unifi-network-application
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - MONGO_USER=unifi
      - MONGO_PASS=GETYOUROWNPASSWORD
      - MONGO_HOST=unifi-db
      - MONGO_PORT=27017
      - MONGO_DBNAME=unifi
      - MONGO_AUTHSOURCE=admin
      - MEM_LIMIT=1024 #optional
      - MEM_STARTUP=1024 #optional
      - MONGO_TLS= #optional
    volumes:
      - /data/unifi:/config
    ports:
      - 8443:8443
      - 3478:3478/udp
      - 10001:10001/udp
      - 8080:8080
      - 1900:1900/udp #optional
      - 8843:8843 #optional
      - 8880:8880 #optional
      - 6789:6789 #optional
      - 5514:5514/udp #optional
    restart: unless-stopped
```

`init-mongo.sh`

```
#!/bin/bash

mongo <<EOF
use ${MONGO_AUTHSOURCE}
db.auth("${MONGO_INITDB_ROOT_USERNAME}", "${MONGO_INITDB_ROOT_PASSWORD}")
db.createUser({
  user: "${MONGO_USER}",
  pwd: "${MONGO_PASS}",
  roles: [
    { db: "${MONGO_DBNAME}", role: "dbOwner" },
    { db: "${MONGO_DBNAME}_stat", role: "dbOwner" },
    { db: "${MONGO_DBNAME}_audit", role: "dbOwner" }
  ]
})
EOF
```

```
chmod +x init-mongo.sh
sudo mkdir /data
sudo docker-compose up -d
```
