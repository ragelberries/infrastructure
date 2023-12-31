# Setup environment
Install Raspberry Pi Os Lite 64 bit Debian bookword

Add `testing` to `/etc/apt/sources.list` in addition to testing and create `/etc/apt/preferences:`
```
Package: *
Pin: release a=testing
Pin-Priority: -2
```

sudo apt update
sudo apt upgrade
sudo apt install -t testing podman

sudo apt install snapd
Add  `cgroup_memory=1 cgroup_enable=memory ipv6.disable=1`to `/boot/cmdline.txt`
sudo snap install microk8s --classic
create /mnt/ragelberries as/for root
microk8s enable ingress hostpath-storage rbac
sudo apt install letsencrypt
sudo certbot certonly --manual --preferred-challenges=dns --email william.arvidsson@gmail.com --server https://acme-v02.api.letsencrypt.org/directory --agree-tos -d ragelberries.net -d *.ragelberries.net
create kubernetes secret cert from ragelberries fullchain and privkey
create keycloak secret with admin-name and admin-password
POSSIBLY DO THESE BEFORE ENABLING HOSTPATH?
Deploy runner-role.yaml, runner-rolebinding.yaml and sc.yaml manually as privileged user
Before first infrastructure deploy, change standard storage class in microk8s

Add to /etc/ssh/sshd_config:
```
PermitRootLogin no
```
sudo systemctl restart sshd
sudo adduser runner
Create /mnt/_work, /mnt/registry and chown runner:runner
sudo loginctl enable-linger runner
add infrastructure registry.container to `/home/runner/.config/container/systemd/`
as runner: `systemctl --user daemon-reload`
`systemctl --user start registry.container`
Add to `/etc/containers/registries.conf`:
```
[registries.insecure]
registries = ['localhost:5000']
```
Follow the "Create self-hosted runner" on github with the runner user, using `/mnt/_work` as the work folder, and stop before the "run" step
Do `./svc.sh install runner` as root with su
sudo systemctl start  actions.runner.ragelberries.ragelberries.service

openssl genrsa -out runner.pem 2048
openssl req -new -key runner.pem -out runner.csr -subj "/CN=runner"
`base64 -w 0 runner.csr` and add it as the request in the runner-csr.example.yaml file
kubectl apply -f runner-csr.yaml
kubectl certificate pprove runner
use microk8s config as a base and add private key as base64 and this as client authority:
kubectl get csr runner -o jsonpath={.status.certificate}

