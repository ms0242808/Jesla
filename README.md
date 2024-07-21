# Jesla

Prerequisite 
---
- Raspberry Pi
- SD Card at least 64 GB with PI OS installed

Upgrade all existing packages on the Raspberry Pi
---
```bash
sudo apt update
sudo apt upgrade
```

Download and run the official Docker setup script
---
```bash
curl -sSL https://get.docker.com | sh
```

For another user to be able to interact with Docker, it needs to be added to the docker group
---
```bash
sudo usermod -aG docker $USER
```

Test if Docker is working
---
```bash
docker run hello-world
```

A message with the following text in it.
```bash
Hello from Docker!
This message shows that your installation appears to be working correctly.
```


Create a file called `docker-compose.yml`
---
```bash
services:
  teslamate:
    image: teslamate/teslamate:latest
    restart: always
    environment:
      - ENCRYPTION_KEY=secretkey #replace with a secure key to encrypt your Tesla API tokens
      - DATABASE_USER=teslamate
      - DATABASE_PASS=password #insert your secure database password!
      - DATABASE_NAME=teslamate
      - DATABASE_HOST=database
      - MQTT_HOST=mosquitto
    ports:
      - 4000:4000
    volumes:
      - ./import:/opt/app/import
    cap_drop:
      - all

  database:
    image: postgres:16
    restart: always
    environment:
      - POSTGRES_USER=teslamate
      - POSTGRES_PASSWORD=password #insert your secure database password!
      - POSTGRES_DB=teslamate
    volumes:
      - teslamate-db:/var/lib/postgresql/data

  grafana:
    image: teslamate/grafana:latest
    restart: always
    environment:
      - DATABASE_USER=teslamate
      - DATABASE_PASS=password #insert your secure database password!
      - DATABASE_NAME=teslamate
      - DATABASE_HOST=database
    ports:
      - 3000:3000
    volumes:
      - teslamate-grafana-data:/var/lib/grafana

  mosquitto:
    image: eclipse-mosquitto:2
    restart: always
    command: mosquitto -c /mosquitto-no-auth.conf
    # ports:
    #   - 1883:1883
    volumes:
      - mosquitto-conf:/mosquitto/config
      - mosquitto-data:/mosquitto/data

volumes:
  teslamate-db:
  teslamate-grafana-data:
  mosquitto-conf:
  mosquitto-data:
```


Start the docker containers with docker compose up
---
```bash
  docker compose up -d
```

Generate your own Tesla tokens
---
[Tesla Auth](https://github.com/adriankumpf/tesla_auth) (macOS, Linux, Windows)

Usage
---
Open the web interface http://your-ip-address:4000
Sign in with your Tesla Account
The Grafana dashboards are available at http://your-ip-address:3000. Log in with the default user admin (initial password admin) and enter a secure password.


Cloudflare tunnel
---
