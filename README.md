# CUPS-docker

Run a CUPS print server on a remote machine to share USB printers over WiFi. Built primarily to use with Raspberry Pis as a headless server, but there is no reason this wouldn't work on `amd64` machines. Tested and confirmed working on a Raspberry Pi 3B+ (`arm/v7`) and Raspberry Pi 4 (`arm64/v8`).

Container packages are available from Docker Hub and Github Container Registry (ghcr.io)
  - Docker Hub Image: `infra7/cups`
  - GHCR Image: `ghcr.io/infra7ti/cups`

## Usage
Quick start with default parameters
```bash
docker run -d \
  --name cups \
  --ulimit nofile=65535:65535 \
  -p 631:631 \
  infra7/cups
```

Customizing your container
```bash
docker run -d \
  --name cups \
  --restart unless-stopped \
  --ulimit nofile=65535:65535 \
  --device /dev/bus/usb \
  -e TZ="America/Sao_Paulo" \
  -e CUPSADMIN=joe \
  -e CUPSPASSWORD=JoEpaS$w0rD \
  -v ./config:/etc/cups \
  -p 631:631 \
  infra7/cups
```
> Note: Using docker secrets (see ENV variables below) and changing the default username and password is highly recommended.

### Parameters and defaults
- `--port` -> default cups network port `631:631`. Change not recommended unless you know what you're doing
- `--ulimit` -> specify the user limits for cups process: for example, to set nofile pass `nofile=65535:65535`.

#### Optional parameters
- `--name` -> whatever you want to call your docker image. using `cups` in the example above.
- `--device` -> used to give docker access to USB printer. Default passes the whole USB bus `/dev/bus/usb`, in case you change the USB port on your device later. change to specific USB port if it will always be fixed, for eg. `/dev/bus/usb/001/005`.
- `-v|--volume` -> adds a persistent volume for CUPS config files if you need to migrate or start a new container with the same settings

Environment variables that can be changed to suit your needs, use the `-e` tag
| # | Parameter        | Default                    | Type   | Description                       |
| - | ---------------- | -------------------------- | ------ | --------------------------------- |
| 1 | TZ               | "Etc/UTC"                  | string | Time zone of your server          |
| 2 | CUPSADMIN        | admin                      | string | Name of the admin user for server |
| 3 | CUPSPASSWORD     | \_\_cUPsPassw0rd\_\_       | string | Password for server admin         |
| 4 | CUPSADMINFILE    | /run/secrets/cups_admin    | string | Filename storing admin username on container |
| 5 | CUPSPASSWORDFILE | /run/secrets/cups_password | string | Filename storing admin password on container |
| 6 | CUPSERRORLOG     | /dev/stderr                | string | Where to write error_log content  |


### docker-compose
```yaml
name: printing

services:
  cupsd:
    environment:
      CUPSADMINFILE: /run/secrets/cups_admin
      CUPSPASSWORDFILE: /run/secrets/cups_password
    healthcheck:
      test: wget -nv -t1 --spider http://localhost:631/printers/ || exit 1
      interval: 10s
      retries: 5
      start_period: 5s
      timeout: 5s
    image: infra7/cups:latest
    ports:
      - 631:631
    restart: unless-stopped
    secrets:
      - cups_admin
      - cups_password
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    volumes:
      - ${PWD}/config:/etc/cups

networks:
  default:
    name: printing

secrets:
  cups_admin:
    file: ${PWD}/secrets/cups_admin
  cups_password:
    file: ${PWD}/secrets/cups_password
```

## Server Administration
You should now be able to access CUPS admin server using the IP address of your headless computer/server http://192.168.xxx.xxx:631, or whatever. 
If your server has avahi-daemon/mdns running you can use the hostname, http://printer.local:631. (IP and hostname will vary, these are just examples)

If you are running this on your PC, i.e. not on a headless server, you should be able to log in on http://localhost:631

## Thanks
Based on the work done by **RagingTiger**: [https://github.com/RagingTiger/cups-airprint](https://github.com/RagingTiger/cups-airprint)
