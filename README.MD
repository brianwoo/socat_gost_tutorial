# Socat Tutorial

- Socat links from one data source to another.
- Data sources:
    - TCP
    - UDP
    - FILE
    - Unix socket
    - Standard input (- or STDIO)    
- Socat supports two-way communication by default
- Socat also supports uni-directional communication
- Socat does NOT support SOCKS5 proxy connections (only SOCKS4 & HTTP proxy are supported)

## Basic syntax
```bash
socat [from-ds] [to-ds]
```

## Forwarding Examples
```bash
# stdin to localhost:1234
socat - TCP4:localhost:9999

# Forward 0.0.0.0:9999 to localhost:8888
# fork,reuseaddr is to fork the listeners
socat TCP4-LISTEN:9999,fork,reuseaddr TCP4:localhost:8888

# *Option*: Only binding on localhost
# Forward 127.0.0.1:9999 to localhost:8888
socat TCP4-LISTEN:9999,fork,reuseaddr,bind=127.0.0.1 TCP4:localhost:8888
```

## Unidirectional Examples
```bash
# stdin to file test.txt
socat -u STDIO OPEN:test.txt,create
```

## Transfer a File
```bash
# Rx side (-d for debug print)
socat -d -d TCP-LISTEN:1234 OPEN:savedFile.txt,create

####################

# Tx side
socat -d -d -u OPEN:/etc/passwd TCP-CONNECT:127.0.0.1:1234
```

## Create a SSL Shell (Remote shell)
```bash
# Generate a self-signed cert
openssl req -newkey rsa:2048 -nodes -keyout cert.key -x509 -days 1000 -out cert.crt

cat cert.key cert.crt > sslcert.pem

# Remote side
socat -d -d OPENSSL-LISTEN:1234,cert=sslcert.pem,verify=0 -

####################

# Local side, giving the remote side a bash shell
socat -d -d OPENSSL-CONNECT:127.0.0.1:1234,cert=sslcert.pem,verify=0 EXEC:/bin/bash
```

## TLS Offloading (HTTPS -> HTTP)
```bash
# A webserver listening localhost:8080
python3 -m http.server --bind 127.0.0.1 8080

# Listening on 1234 and forwarding to localhost:8080
socat -dd OPENSSL-LISTEN:1234,fork,reuseaddr,cert=sslcert.pem,verify=0 tcp:localhost:8080

####################

# Access the HTTPS endpoint
curl -k https://127.0.0.1:1234
```

## Local Port Forward through an HTTP Proxy
#### Useful on Android:
1. Chrome on Android does NOT support proxy settings. 
2. Shadowsocks on Android does NOT support local-tunnel.
```bash
# HTTP Server listening on localhost only, port 8888
python3 -m http.server --bind 127.0.0.1 8888

# Server is protected by a Shadowsock Server
./ssserver -c server_config.json
>>> INFO shadowsocks tcp server listening on 0.0.0.0:8388, inbound address 0.0.0.0:8388

####################

# Client creates a HTTP Proxy
./sslocal -c client1.json
>>> INFO shadowsocks HTTP listening on 127.0.0.1:8890

# socat creates a connection which uses the HTTP Proxy
socat -dd tcp-listen:1234,fork,reuseaddr PROXY:127.0.0.1:localhost:8888,proxyport=8890

# Access the protected web server
curl http://localhost:1234

# Equivalent to 
# curl --proxy http://127.0.0.1:8890 http://127.0.0.1:8888
```

## Local Port Forward, SSH through an HTTP Proxy
#### Useful on Android:
1. Shadowsocks on Android does NOT support local-tunnel.
```bash
# Server is protected by a Shadowsock Server
./ssserver -c server_config.json
>>> INFO shadowsocks tcp server listening on 0.0.0.0:8388, inbound address 0.0.0.0:8388

####################

# Client creates a HTTP Proxy
./sslocal -c client1.json
>>> INFO  shadowsocks HTTP listening on 127.0.0.1:8890

# socat creates a connection to server's SSH port
socat -dd tcp-listen:1234,fork,reuseaddr PROXY:127.0.0.1:127.0.0.1:22,proxyport=8890

# Make SSH connection
ssh 127.0.0.1 -p 1234

# Equivalent to 
# ssh 127.0.0.1 -o "ProxyCommand=nc -X connect -x 127.0.0.1:8890 %h %p"
```