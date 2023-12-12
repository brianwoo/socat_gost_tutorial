# Gost Tutorial

- Gost supports both SOCKS4, SOCKS5 and HTTP proxy connections

## Chaining Proxies

```bash
# Chaining http proxy -> upstream socks5
./gost -L http://:8888 -F socks5://127.0.0.1:8890

curl --proxy 'http://localhost:8888' http://myip.wtf/json

# Chaining socks5 proxy -> upstream socks5
./gost -L socks5://:8888 -F socks5://localhost:8890

# Can even chain multiple upstream proxies
# ./gost -L socks5://:8888 \
#   -F socks5://localhost:8890 -F socks5://localhost:9999

curl --proxy 'socks5://localhost:8888' http://myip.wtf/json
```

## TLS Offloading #1 (HTTPS -> HTTP)
```bash
# A webserver listening localhost:8080
python3 -m http.server --bind 127.0.0.1 8080

# Listening on 8443 and forwarding to localhost:8080
# Using the CA Cert, Server Public Cert + private key
./gost -L 'tls://:8443/127.0.0.1:8080?caFile=rootCA.crt&certFile=server.crt&keyFile=server.key'

####################

# Access the HTTPS endpoint
curl https://192.168.2.60:8443 --cacert rootCA.crt --cert client.pem
```

## TLS Offloading #2 (HTTP -> HTTPS -> HTTPS -> HTTP)
```bash
# A webserver listening localhost:8080
python3 -m http.server --bind 127.0.0.1 8080

# Listening on 8443 and forwarding to localhost:8080
# Using the CA Cert, Server Public Cert + private key
./gost -L 'tls://:8443/127.0.0.1:8080?caFile=rootCA.crt&certFile=server.crt&keyFile=server.key'

####################
# listen on 0.0.0.0:8080, connect to server:8443
./gost -L tcp://:8080 -F 'forward+tls://192.168.2.60:8443?caFile=rootCA.crt&certFile=client.crt&keyFile=client.key'

# Access the server
curl http://localhost:8080
```


## Simple Local Port Forward
```bash
# tcp-listen 1234 -> localhost:22
./gost -L tcp://:1234/localhost:22
```

## Local Port Forward through an SOCKS5 Proxy

```bash
# HTTP Server listening on localhost only, port 8888
python3 -m http.server --bind 127.0.0.1 8888

# Server is protected by a Shadowsock Server
./ssserver -c server_config.json
>>> INFO shadowsocks tcp server listening on 0.0.0.0:8388, inbound address 0.0.0.0:8388

####################

# Client creates a SOCKS5 Proxy
./sslocal -c client1.json
>>> INFO shadowsocks socks TCP listening on 127.0.0.1:8890

# gost creates a connection which uses the HTTP Proxy
# local listen on 1234 -> proxy -> server:8888
./gost -L tcp://:1234/localhost:8888  -F socks5://127.0.0.1:8890

# Access the protected web server
curl http://localhost:1234

# Equivalent to using SOCKS5 with curl
# curl --proxy socks5://127.0.0.1:8890 http://127.0.0.1:8888
```


## Local Port Forward, SSH through an HTTP Proxy

```bash
# Server is protected by a Shadowsock Server
./ssserver -c server_config.json
>>> INFO shadowsocks tcp server listening on 0.0.0.0:8388, inbound address 0.0.0.0:8388

####################

# Client creates a SOCKS5 Proxy
./sslocal -c client1.json
>>> INFO  shadowsocks socks TCP listening on 127.0.0.1:8890

# gost creates a connection to server's SSH port
# local listen on 1234 -> proxy -> server:22
./gost -L tcp://:1234/localhost:22 -F socks5://127.0.0.1:8890

# Make SSH connection
ssh 127.0.0.1 -p 1234

# Equivalent to 
# ssh 127.0.0.1 -o "ProxyCommand=nc -X connect -x 127.0.0.1:8890 %h %p"
```

## Relay (proprietary protocol) + TLS
#### TLS Tunnel with SOCKS5 proxy
```bash
# Server listening on port 12345
./gost -L 'relay+tls://username:password@:12345?caFile=rootCA.crt&certFile=server.crt&keyFile=server.key'

####################

# Client running SOCKS5 at 8888, connect to server with TLS
./gost -L socks5://:8888 -F 'relay+tls://username:password@192.168.2.60:12345?caFile=rootCA.crt&certFile=client.crt&keyFile=client.key&nodelay=false'

# Using the proxy
curl --proxy 'socks5://127.0.0.1:8888' http://myip.wtf/json
```


## Relay (proprietary protocol) + WSS
#### Websocket Secure (wss) Tunnel with SOCKS5 proxy
```bash
# Server listening on port 12345
./gost -L 'relay+wss://username:password@:12345?caFile=rootCA.crt&certFile=server.crt&keyFile=server.key'

####################

# Client running SOCKS5 at 8888, connect to server with WSS
./gost -L socks5://:8888 -F 'relay+wss://username:password@192.168.2.60:12345?caFile=rootCA.crt&certFile=client.crt&keyFile=client.key&nodelay=false'

# Using the proxy
curl --proxy 'socks5://127.0.0.1:8888' http://myip.wtf/json
```