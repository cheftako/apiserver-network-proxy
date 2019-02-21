# Proxy Service

## Examples

The current example runs the two actual services as well as a sample client on one end and a sample destination for requests on the other. 
- *Proxy service:* The proxy service takes the API server requests and forwards them appropriately.
- *Agent service:* The agent service connects to the proxy and then allows traffic to be forwarded to it.

### Proxy with dial back Agent 

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
go run cmd/agent/main.go
```

- Start proxy service
```
go run cmd/proxy/main.go
```

- Run client (Sample client)
```
go run cmd/client/main.go
```
