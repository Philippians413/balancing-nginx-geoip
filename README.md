### Preparations

Run command: `./geo2nginx.pl < GeoIPCountryWhois.csv > geo_default.conf`

Check config: `sudo nginx -t -c ${PWD}/nginx.conf` 

Create network for all containers: `docker network create --subnet=10.0.0.0/8 projector`

### Test default backend and backup

1. Start container with Load Balancer
```
docker run --name lb \
 --net projector --ip 10.0.0.2 \
 -v ${PWD}/nginx.conf:/etc/nginx/nginx.conf:ro \
 -v ${PWD}/geo.conf:/etc/nginx/geo.conf \
 -p 80:80 -d nginx
```

2. Start default backend
```
docker run --name default_backend \
 --net projector --ip 10.0.0.3 \
 -v ${PWD}/default.html:/usr/share/nginx/html/index.html \
 -d nginx
```
Check that request reach default.backend:
```
$ curl localhost:80
<h4>This is default.backend server</h4>
```

3. Start backup
```
docker run --name backup \
 --net projector --ip 10.0.0.4 \
 -v ${PWD}/backup.html:/usr/share/nginx/html/index.html \
 -d nginx
```
Stop default backend
```
docker stop default_backend
```
Check that request reach backup
```
$ curl localhost:80
<h4>This is backup server</h4>
```

### Test US backend

1. Start US backend containers
```
docker run --name us_backend6 \
 --net projector --ip 10.0.0.6 \
 -v ${PWD}/us6.html:/usr/share/nginx/html/index.html \
 -d nginx

docker run --name us_backend7 \
 --net projector --ip 10.0.0.7 \
 -v ${PWD}/us7.html:/usr/share/nginx/html/index.html \
 -d nginx
```

2. Run ngrok `ngrok http 80`
Check connection
```
$ curl http://1149b63581fe.ngrok.io
<h4>This is default.backend server</h4>
```

3. Connect through VPN from USA
```
$ curl http://1149b63581fe.ngrok.io
<h4>This is default.backend server</h4>
```
With ngrock it's not successful.
For test case let's add `10.0.0.0/8 US` to `geo{}` block. Then restart nginx from container `nginx -s reload` and check from machine.
```
$ curl localhost:80
<h4>This is US.backend 10.0.0.6 server</h4>
```

So everything works, but something wrong with IP ranges in geo.conf :)
