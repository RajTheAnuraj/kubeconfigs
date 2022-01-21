# Kong

Kong is an open source API gateway
So that we can redirect our stuff with readable address

* First create a file called `kong.yml` at a place which you can mount to the container. I created at a place on the docker host `/mnt/nfsvolume/kong/kong.yml`. The path will be mounted into the container using volume mount

```
_format_version: "2.1"

services:
- name: my-link-app
  url: http://192.168.86.250:30033
  routes:
  - name: dashboard
    paths:
    - /
```

* Now create another file as kong.conf in this repo. That is the default config of kong only modified property is `declarative_config`. This points to the folder where our kong.yaml is mounted.

* once both the files are in you can just run the below docker command

```
docker run -d --rm --name kong \
    -e "KONG_DATABASE=off" \
    -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
    -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
    -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
    -e “KONG_DECLARATIVE_CONFIG=/home/kong/kong.yml” \
    -p 80:8000 \
    -p 8443:8443 \
    -p 8001:8001 \
    -p 8444:8444 \
    --volume /mnt/nfsvolume/kong:/home/kong \
    kong kong start -c /home/kong/kong.conf 
```

* Note that database is off, so we are loading config from kong.yml
* Also note that we are exposing the port 80 of host
* Note we volume mount nfsvolume where we have the kong files to /home/kong 
* also note that the start command is passing the configuration file path

