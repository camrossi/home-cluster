To use traefik ingress controller we need to enable `trusted_proxies` and `use_x_forwarded_for` in the `configuration.yaml` file. 
For now I just opened the file from the NFS share and added this manually and re-started the POD. There might be a way to do it directly with argo ?
```
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 10.42.0.0/16
```