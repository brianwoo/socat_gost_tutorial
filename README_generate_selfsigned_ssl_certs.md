# Generate Self Signed SSL Certs
- NOTE: ed25519 will not work with browser TLS handshake. This is for a machine-to-machine communication.
- NOTE: for browser TLS handshake, use rsa:2048 or rsa:4096

```bash
openssl req -newkey rsa:4096 -nodes \
    -subj "/O=Brookfield Wireless/OU=IT Department/ST=Blue Water/C=BW" \
    -x509 -days 3650 \
    -keyout selfSigned.key.pem \
    -out selfSigned.cert.pem
```

## See Cert's Content (Optional)
```bash
openssl x509 -in selfSigned.cert.pem -text
openssl x509 -in selfSigned.cert.pem -purpose -noout -text
```
