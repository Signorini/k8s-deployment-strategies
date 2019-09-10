---
# Openrest - Simple nginx to expose prometheus metrics

It's a simple nginx expose a endpoint and prometheus metrics.

```
// nginx-prometheus-exporter-v1
docker build -t "nginx" .

// nginx-prometheus-exporter-v1
docker build -t "nginx" .
```

Root location
```
location / {
    content_by_lua_block {
        ngx.say("App - Version 1.0.0")
    }
}
```

Metrics endpoint
```
location /metrics {
    content_by_lua_block {
        if ngx.var.connections_active ~= nil then
            http_connections:set(ngx.var.connections_active, {"active"})
            http_connections:set(ngx.var.connections_reading, {"reading"})
            http_connections:set(ngx.var.connections_waiting, {"waiting"})
            http_connections:set(ngx.var.connections_writing, {"writing"})
        end
        prometheus:collect()
    }
}
```