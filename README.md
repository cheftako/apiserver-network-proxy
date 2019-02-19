# Proxy Service

## Examples

There are 2 modes to run the examples:
- client/server mode
- agent dial back mode

### Client/Server mode

```
client ==> server(:8090) ==> SimpleHTTPServer(:8000)
```

- Start SimpleHTTPServer
```console
python -m SimpleHTTPServer
```

- Start server
```
go run examples/server/main.go
```

- Run client
```
go run examples/client/main.go
```

### Agent dial back mode

```
client ==> (:8090) agentserver (:8091) <== agentclient ==> SimpleHTTPServer(:8000)
  |                                                                           ^
  |                               Tunnel                                      |
  +---------------------------------------------------------------------------+
```

- Start SimpleHTTPServer
```console
python -m SimpleHTTPServer
```

- Start agentserver
```
go run examples/agentserver/main.go
```

- Start agentclient
```
go run examples/agentclient/main.go
```

- Run client
```
go run examples/client/main.go
```
