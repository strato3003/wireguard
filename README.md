# mise en place server wireguard sous kube
## selon ce lien :
cf https://medium.com/@s.hameedakmal/wireguard-vpn-in-a-kubernetes-cluster-e306fdc69731
## details d'implementation

### create a new client config from server
  - edit deploy and modify env PEERS to add clients...
```shell
kubectl get deploy wireguard -o json |jq '.spec.template.spec.containers[].env'
[
  {
    "name": "TZ",
    "value": "Europe/Paris"
  },
  {
    "name": "PEERS",
    "value": "pican,pi64"
  },
  {
    "name": "PUID",
    "value": "1000"
  },
  {
    "name": "PGID",
    "value": "1000"
  },
  {
    "name": "ALLOWEDIPS",
    "value": "0.0.0.0/0"
  },
  {
    "name": "SERVERPORT",
    "value": "51820"
  },
  {
    "name": "SERVERURL",
    "value": "51.178.142.116"
  }
]
kubectl exec -it po/wireguard-5fcb64c496-mpvhl -c sshd -- sh
```

### launch the new client

```shell
root@pican:~# wg genkey | tee privatekey | wg pubkey > publickey
root@pican:~# cat privatekey
yGNGOSMuxwa8dp7E5Z0ec9d4nm/OdnJYctWXv8Xwk2U=
root@pican:~# cat publickey
jeTgGANRbQcwCAiWtjBMbjyvJvEPWS2PMG5u+7VCURg=
root@pican:~#
```

## Tunneling

```shell
ssh -L 20206:192.168.200.6:443 -J pi@ingress.sail.eyeseateam.eu:23422 pi@10.13.13.3
```
access on starlink router : https://localhost:20206/cgi-bin/luci/admin/services/nlbw
