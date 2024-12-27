# Setup environment
Install Raspberry Pi Os Lite 64 bit Debian bookworm

```
sudo apt update
sudo apt upgrade
```

Add  `cgroup_memory=1 cgroup_enable=memory` to `/boot/firmware/cmdline.txt`  
Add to /etc/ssh/sshd_config:
```
PermitRootLogin no
```
Restart  
```
sudo mkdir /mnt/arvidsson
```

Install k3s with
```
curl -o k3s.sh https://get.k3s.io
bash k3s.sh - --default-local-storage-path=/mnt/arvidsson \
--disable-cloud-controller \
--secrets-encryption \
--tls-san rpi5.local \
--token TOKEN
```

Copy config from k3s server with `sudo k3s kubectl view --raw` 
and copy to local configuration and change server IP address

Deploy sc.yaml to get persistent storage paths



```
sudo adduser runner
sudo passwd -d runner
```
Create /runner/_work and chown runner:runner
```
sudo loginctl enable-linger runner
```
Follow the "Create self-hosted runner" on github with the runner user, using `/mnt/_work` as the work folder, and stop before the "run" step  
Do `./svc.sh install runner` as root with su  
```
sudo systemctl start  actions.runner.ragelberries.ragelberries.service
```

```
openssl genrsa -out runner.pem 4096
openssl req -new -key runner.pem -out runner.csr -subj "/CN=runner"
```
`base64 -w 0 runner.csr` and add it as the request in the runner-csr.example.yaml file
```
kubectl apply -f runner-csr.yaml
kubectl certificate approve runner
```
use microk8s config as a base and add private key as base64 and this as client authority:
```
kubectl get csr runner -o jsonpath={.status.certificate}
```
add `export KUBECONFIG=~/.kube/config` to .env in runner folder



