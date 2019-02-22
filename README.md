# Proxy Service

## Build

```console
make clean
make certs
make build
```

## Examples

The current example runs the two actual services as well as a sample client on one end and a sample destination for requests on the other. 
- *Proxy service:* The proxy service takes the API server requests and forwards them appropriately.
- *Agent service:* The agent service connects to the proxy and then allows traffic to be forwarded to it.

### mTLS Proxy with dial back Agent 

```
client ==> (:8090) proxy (:8091) <== agent ==> SimpleHTTPServer(:8000)
  |                                                    ^
  |                          Tunnel                    |
  +----------------------------------------------------+
```

- Start SimpleHTTPServer (Sample destination)
```console
python -m SimpleHTTPServer
```

- Start agent service
```
./bin/proxy-server --caCert=certs/issued/ca.crt --serverCert=certs/issued/proxy-master.crt --serverKey=certs/private/proxy-master.key
```

- Start proxy service
```
./bin/proxy-agent
```

- Run client (mTLS enabled sample client)
```
./bin/proxy-test-client --caCert=certs/issued/ca.crt --clientCert=certs/issued/proxy-client.crt --clientKey=certs/private/proxy-client.key
```
