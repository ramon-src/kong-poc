# POC Kong Gateway

Videos

https://www.youtube.com/watch?v=DZZEKyH0MHQ
https://www.youtube.com/watch?v=OjF95vVldxY
https://docs.konghq.com/hub/kong-inc/jwt/


Run Kong in DB Less mode inside the folder kongdbless

```sh
cd kongdbless && \
rm -rf prefix_volume tmp_volume && \
docker run --read-only -d --name kong-dbless \
--network=kong-net \
-v "$(pwd)/declarative:/kong/declarative/" \
-v "$(pwd)/tmp_volume:/tmp" \
-v "$(pwd)/prefix_volume:/var/run/kong" \
-e "KONG_PREFIX=/var/run/kong" \
-e "KONG_DATABASE=off" \
-e "KONG_DECLARATIVE_CONFIG=/kong/declarative/kong.yml" \
-e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
-e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
-e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
-e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
-e KONG_LICENSE_DATA \
-p 8000:8000 \
-p 8443:8443 \
-p 8001:8001 \
-p 8444:8444 \
-p 8002:8002 \
-p 8445:8445 \
-p 8003:8003 \
-p 8004:8004 \
kong/kong-gateway:3.6.1.1
```

Refreshing container

```sh
docker restart kong-dbless
```

Check if kong is working

```sh
curl -i http://localhost:8001/services
```

Run docker containers to map in kong

```sh
docker run -d -p 8081:80 --name servicea --network kong-net servicea
-e "SERVER_PORT=8081" \
-e "HOSTNAME=service a" 

docker run -d -p 8082:80 --name serviceb --network kong-net serviceb
-e "SERVER_PORT=8082" \
-e "HOSTNAME=service b" 

docker run -d -p 8083:80 --name servicec --network kong-net servicec
-e "SERVER_PORT=8083" \
-e "HOSTNAME=service c" 
```

Service configure to docker host

```yml
- host: host.docker.internal
  name: servicea
  port: 8081
  protocol: http
  routes:
    - name: routea
      paths:
        - /a
      strip_path: true

- host: host.docker.internal
  name: serviceb
  port: 8082
  protocol: http
  routes:
    - name: routeb
      paths:
        - /b
      strip_path: true

- host: host.docker.internal
  name: servicec
  port: 8083
  protocol: http
  routes:
    - name: routec
      paths:
        - /c
      strip_path: true
```

Check if service a is working

```sh
curl -i http://localhost:8000/a
curl -i http://localhost:8000/b
curl -i http://localhost:8000/c
```


To Authenticate

Get the kid from kong jwt plugin

```sh
curl -i http://localhost:8001/consumers/{consumer}/jwt
```

Copy the key from the response:

```json
{
    "data": [
        {
            "algorithm": "HS256",
            "consumer": {
                "id": "54f0d637-219d-5e81-b874-8510b845955a"
            },
            "created_at": 1711029390,
            "id": "475f00ec-552f-4634-aa75-ae8a6edad439",
            "secret": "TEST",
            "tags": null,
            "rsa_public_key": null,
            "key": "H4n10pRzzci5mV4Ga120eO5mrOzWeCUn"
        }
    ],
    "next": null
}
```

Then the application should generate the jwt with:
- kid in headers
- exp in payload
- and should use the secret