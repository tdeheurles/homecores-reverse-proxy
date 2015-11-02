# Kubernetes reverse Proxy

Simple reverse proxy running as a pod.
It uses the public network (not the flannel one). It takes the entering connections and direct them to the good kubernetes service.

The nginx config file is defined to read multiple confd. The container is sharing a volume to the confd directory.

Start that simple nginx.
Define a confd each time you define a new service.

As an example:

```
server {
  server_name kubeui.tdeheurles.com;

  location / {
    proxy_pass              http://10.2.57.8:8080;
    proxy_set_header        X-Real-IP       $remote_addr;
    proxy_set_header        Host            $host;
    proxy_redirect          off;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

For now I don't use the k8s service name as I fall in that issue : 
```bash
2015/11/02 08:45:50 [emerg] 76#76: host not found in upstream "kube-ui" in /etc/nginx/conf.d/kube-ui.conf:6
nginx: [emerg] host not found in upstream "kube-ui" in /etc/nginx/conf.d/kube-ui.conf:6
```

The `k8s-rp.yml` is directly copied in the kubelet manifest folder `/etc/kubernetes/manifests/` defined at kubelet start time. I don't use `kubectl` to start as the `hostNetwork spec` is not allowed:

```yaml
spec:
  hostNetwork:     true
```

```bash
kubectl describe po k8s-rp
Name:                           k8s-rp
Namespace:                      default
Image(s):                       tdeheurles/homecores-reverse-proxy
Node:                           192.168.1.13/192.168.1.13
Labels:                         <none>
Status:                         Pending
Reason:
Message:
IP:                             192.168.1.13
Replication Controllers:        <none>
Containers:
  k8s-rp:
    Image:              tdeheurles/homecores-reverse-proxy
    State:              Waiting
      Reason:           Image: tdeheurles/homecores-reverse-proxy is ready, container is creating
    Ready:              False
    Restart Count:      0
Conditions:
  Type          Status
  Ready         False
Events:
  FirstSeen                             LastSeen                        Count   From                    SubobjectPath   Reason          Message
  Mon, 02 Nov 2015 08:54:10 +0000       Mon, 02 Nov 2015 08:54:10 +0000 1       {scheduler }                            scheduled       Successfully assigned k8s-rp to 192.168.1.13
  Mon, 02 Nov 2015 08:54:10 +0000       Mon, 02 Nov 2015 08:55:30 +0000 9       {kubelet 192.168.1.13}                  failedSync      Error syncing pod, skipping: pod with UID "4890625f-813f-11e5-8425-086266113e7e" specified host networking, but is disallowed

```

`host networking` should be activable ...