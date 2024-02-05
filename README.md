# mise en place server wireguard sous kube

## selon ce lien :
cf https://medium.com/@s.hameedakmal/wireguard-vpn-in-a-kubernetes-cluster-e306fdc69731

## details d'implementation

### conservation des peers avec leur config grace au PVC 
  - ainsi la modification de deploy.yaml n'entraine pas ka regeneration des cles du server ni de chaque PEER
  - en revanche, les cles ssh necessaires a la copie vers 10.13.13.1:23422 changent et le ssh-copy-id doit etre refait
### ajout des recettes Kube dans un nouveau repo par repertoire, ici "wireguard"
```shell
cd /Users/jnmartineau/eyesea-saison2/sail.eyeseateam.eu/wireguard
git init
git config --global init.defaultBranch main
git remote add origin https://github.com/strato3003/wireguard.git
git branch -M main
git add *
git commit -m 'first'
git push -u origin main
```
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
    "value": "32"
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

# TIMELAPSE
## capture timelapse depuis RPI Zero 2w via raspstill puis ssh
```shell
pi@raspvid:~ $ sudo cat /etc/systemd/system/timelapse.service
[Unit]
Description=My Video Service
After=multi-user.target

[Service]
Type=idle
User=pi
Group=pi
ExecStart=/bin/bash /home/pi/timelapse.sh 10

[Install]
WantedBy=multi-user.target
```
```shell
pi@raspvid:~ $ cat /home/pi/timelapse.sh
#!/bin/bash
SLEEPT=$1
while true
do
  DATE=$(date +"%Y-%m-%d_%H%M%S")
  raspistill -o /home/pi/eole/eole_$DATE.jpg -w 1920 -h 1080 -t 1000
  echo "pic /home/pi/eole/eole_$DATE.jpg shooted."
  #rclone copy /home/pi/eole/eole_$DATE.jpg gdrive:/eole && rm /home/pi/eole/eole_$DATE.jpg && \
  scp /home/pi/eole/eole_$DATE.jpg pi@10.13.13.1:eole && rm /home/pi/eole/eole_$DATE.jpg && \
  echo "pic /home/pi/eole/eole_$DATE.jpg uploaded and locally deleted."
  echo "sleep $SLEEPT seconds..."
  sleep $SLEEPT
done
pi@raspvid:~ $
```
## envoi des PNG
```shell
> ssh -p 23422 pi@ingress.sail.eyeseateam.eu ls -ltr eole |tail
pi@ingress.sail.eyeseateam.eu's password:
-rw-r--r--    1 pi       pi         1117148 Feb  5 22:38 eole_2024-02-05_233828.jpg
-rw-r--r--    1 pi       pi         1123253 Feb  5 22:38 eole_2024-02-05_233841.jpg
-rw-r--r--    1 pi       pi         1121060 Feb  5 22:38 eole_2024-02-05_233855.jpg
-rw-r--r--    1 pi       pi         1133588 Feb  5 22:39 eole_2024-02-05_233908.jpg
-rw-r--r--    1 pi       pi         1136719 Feb  5 22:39 eole_2024-02-05_233921.jpg
-rw-r--r--    1 pi       pi         1140324 Feb  5 22:39 eole_2024-02-05_233934.jpg
-rw-r--r--    1 pi       pi         1136971 Feb  5 22:39 eole_2024-02-05_233947.jpg
-rw-r--r--    1 pi       pi         1138518 Feb  5 22:40 eole_2024-02-05_234000.jpg
-rw-r--r--    1 pi       pi         1134810 Feb  5 22:40 eole_2024-02-05_234013.jpg
-rw-r--r--    1 pi       pi         1139135 Feb  5 22:40 eole_2024-02-05_234026.jpg
```
