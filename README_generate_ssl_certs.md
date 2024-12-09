# Generate Private RootCA, Server & Client SSL Certs
- NOTE: ed25519 will not work with browser TLS handshake. This guide is for a machine-to-machine communication.
- NOTE: for browser TLS handshake, use rsa:2048 or rsa:4096

## Generate a Root CA

```bash
openssl req -x509 -days 3650 \
  -subj "/O=TrustAsia Technologies, Inc./CN=TrustAsia OV TLS Pro CA G3" \
  -newkey ed25519 -keyout rootCA.key -nodes -out rootCA.crt
```

## See Cert's Content (Optional)
```bash
openssl x509 -in rootCA.crt -text
openssl x509 -in rootCA.crt -purpose -noout -text
```


## Generate a Server Cert

```bash
# Generate a Server Private Key and Certificate Signing Request (CSR)
# This is for accessing the server by an IP (192.168.2.60)
openssl req -newkey ed25519 \
  -subj "/O=Institute of Virology/OU=IT Department/CN=192.168.2.60" \
  -nodes -keyout server.key -out server.csr

# This is for accessing the server by an IP (192.168.2.60)
# If by hostname, replace IP address with hostname and remove IP.1
echo -en "subjectAltName = @alt_names\n[alt_names]\nDNS.1 = 192.168.2.60\nIP.1 = 192.168.2.60\n" > server.ext

# Signing the certificate with the RootCA cert
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in server.csr \
  -out server.crt -days 3650 -CAcreateserial -extfile server.ext

cat server.key server.crt > server.pem
```

## Generate a Client Cert

```bash
# Generate a Server Private Key and Certificate Signing Request (CSR)
openssl req -newkey ed25519 \
  -subj "/O=Central Intelligence Agency/OU=IT Department/CN=node1" \
  -nodes -keyout client.key -out client.csr

echo -en "subjectAltName = @alt_names\n[alt_names]\nDNS.1 = node1" > client.ext

# Signing the certificate with the RootCA cert
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in client.csr \
  -out client.crt -days 3650 -CAcreateserial -extfile client.ext

cat client.key client.crt > client.pem
```
