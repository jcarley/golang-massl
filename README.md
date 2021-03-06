# golang-massl

This project provides some simple examples of configuring mutual authentication (MASSL) over TLS using x509 certificates.

# MASSL?

The purpose of mutual authentication with TLS using x509 certificates is to enable verification of incoming connections using a robust and reliable mechanism. Doing it at the TLS level also reduces exposure of the service to probing by third parties as without a signed client certificate they won't get past the handshake.

One downside to this approach is that it can be difficult to debug connection failures as the `alert` which TLS server sends back may not be surfaced by the client.

# setup 

Before you can run the server and the client you need to generate some certs using [cfssl](https://github.com/cloudflare/cfssl).

To pull install all the cfssl commands in your `$GOPATH/bin` run the following, note make sure this directory is in your `$PATH`!.

```
go get -u github.com/cloudflare/cfssl/cmd/...
```

Navigate to provided CSR files provided.

```
cd certs
```

Generate the CA certificate and private key.

```
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

Generate a server cert using the CSR provided.

```
cfssl gencert  \
    -ca=ca.pem \
    -ca-key=ca-key.pem \
    -config=ca-config.json \
    -hostname=localhost,127.0.0.1 \
    -profile=massl server-csr.json | cfssljson -bare server
```

Generate a client cert using the CSR provided.

```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=massl \
  client-csr.json | cfssljson -bare client
```

# running TCP/TLS echo

From the root of the project you can start the simple echo server in one terminal.

```
go run cmd/mass-tls-server/main.go
```

Then run the client in another.

```
go run cmd/mass-tls-client/main.go
```

The output of the server should look something like.

```
2018/03/04 10:27:52 listen:  127.0.0.1:2222
2018/03/04 10:27:54 [127.0.0.1:2222 -> 127.0.0.1:50388] accept
2018/03/04 10:27:54 [127.0.0.1:2222 -> 127.0.0.1:50388] client common name: system:client
2018/03/04 10:27:54 [127.0.0.1:2222 -> 127.0.0.1:50388] line: abc123
```

The output of the client should look something like.

```
2018/03/04 10:27:54 [127.0.0.1:50388 -> 127.0.0.1:2222] connect
2018/03/04 10:27:54 [127.0.0.1:50388 -> 127.0.0.1:2222] client common name: system:server
2018/03/04 10:27:54 [127.0.0.1:50388 -> 127.0.0.1:2222] write
2018/03/04 10:27:54 [127.0.0.1:50388 -> 127.0.0.1:2222] line: abc123
```

# running HTTPS echo

From the root of the project you can start the simple https server in one terminal.

```
go run cmd/mass-https/*.go server
```

Then run the client in another.

```
go run cmd/mass-https/*.go client
```

The output of the server should look something like.

```
2018/03/04 15:44:10 listen: https://localhost:2223
2018/03/04 15:46:57 [/echo -> 127.0.0.1:53277] accept
2018/03/04 15:46:57 [/echo -> 127.0.0.1:53277] client common name: system:client
2018/03/04 15:46:57 [/echo -> 127.0.0.1:53277] line: abc123
```

The output of the client should look something like.

```
2018/03/04 15:46:57 url: https://localhost:2223/echo
2018/03/04 15:46:57 [-> localhost:2223] accept
2018/03/04 15:46:57 [-> localhost:2223] client common name: system:server
2018/03/04 15:46:57 [-> localhost:2223] line: abc123
```


# references

* [wikipedia: Mutual authentication](https://en.wikipedia.org/wiki/Mutual_authentication)
* [Traffic Analysis of an SSL/TLS Session](http://blog.fourthbit.com/2014/12/23/traffic-analysis-of-an-ssl-slash-tls-session)

# license
This code is released under MIT License, and is copyright Mark Wolfe.  