# Kubernetes reverse Proxy

Simple reverse proxy running as a pod.
It uses the public network (not the flannel one). It takes the entering connections and direct them to the good kubernetes service.

The nginx config file is defined to read multiple confd. The container is sharing a volume to the confd directory.

Start that simple nginx.
Define a confd each time you define a new service.

As an example:

```
server {
    server_name   example.com
    proxy_pass    example.com
}
```
