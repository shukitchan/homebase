# Project WatchTower

Self Hosting useful applications for your home
* Standard Notes: Note Taking application
* Bitwarden: Password Management application
* Wallabag: Bookmark application

## Pre-req

A working Mac machine with the following installed
* Tailscale
* Docker Desktop
* Amphetamine App

## Generating SSL certificate for your machine through Tailscale
* Generate SSL cert for the domain name. Domain name is the full domain that can be retrieved from tailscale web UI
```
/Applications/Tailscale.app/Contents/MacOS/Tailscale cert <domain name>
```

## Amphetamine App
* Create Indefinitely New Session
* Yes - "Allow display sleep"
* No - "Allow system sleep when display is closed"

## Setting containers for the applications

* Apache Traffic Server (ATS) for exposing the applications under one domain
* Standard Notes
* Bitwarden
* Wallabag
 
### ATS
* Use ats-alpine container - https://github.com/shukitchan/ats-alpine
* expose container port 8443 to port 443
* setup /work directory to point to ~/Library/Containers/io.tailsacle.ipn/macos/Data
* records.config - export port 8443 as SSL server port
* allow DELETE in ip_allow.yaml
* remap.config
```
map https://<domain name>/headers http://httpbin.org/headers
map https://<domain name>/sn/ http://host.docker.internal:3000/
map https://<domain name>/bw/ http://host.docker.internal:30080/
map wss://<domain name>/bw/ ws://host.docker.internal:30080/
map https://<domain name> http://host/docker.internal:20080/
```
* ssl_multicert.config
```
dest_ip=* ssl_cert_name=/work/<domain name>.crt ssl_key_name=/work/<domain name>.key
```

## Standard Notes
* https://docs.standardnotes.com/self-hosting/docker
* Self host repo - https://github.com/shukitchan/standalone
* Decrypt repo for data restore - https://github.com/shukitchan/decrypt

## Bitwarden
* https://bitwarden.com/help/install-on-premise-linux
  * provide a domain name
  * not using let's encrypt, not having a SSL cert to use, do not generate self-signed SSL cert
  * need installation key and id
  * Change config.yml in bwdata to use 30080/30443 for ports
* Self host repo - https://github.com/shukitchan/self-host
* Can consider using backup for data restore - https://bitwarden.com/help/backup-on-premise
* Upgrade instructions - https://bitwarden.com/help/updating-on-premise/
* Recommend to export vault before the upgrade

## Wallabag
* Use redis as the backend storage
  * `docker run -p 6379:6379 --name redis redis:alpine` 
* Run wallabag on port 20080 and provide the domain name. We need to also link it with the redis
  * `docker run -p 20080:80 -e "SYMFONY__ENV_DOMAIN_NAME=https://<domain name>" --link redis:redis wallabag/wallabag`
