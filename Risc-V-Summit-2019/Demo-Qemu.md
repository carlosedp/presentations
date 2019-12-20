# Risc-V Summit Container Workshop

```bash
ssh -p 22222 -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no carlosedp@localhost

sudo ethtool --offload eth0 rx off tx off
sudo ethtool -K eth0 rx off tx off
sudo ethtool -K eth0 gso off
sudo ethtool -K eth0 gro off
sudo ethtool -K eth0 tso off

uname -a
```

Golang

```bash
# Install
wget https://github.com/carlosedp/riscv-bringup/releases/download/v1.0/go-1.13dev-riscv64.tar.gz
tar vxf go-1.13dev-riscv64.tar.gz -C /usr/local/

# Use
go version

cat hello.go

go run hello.go
```

Docker Info / Demo

```bash
# Install
wget https://github.com/carlosedp/riscv-bringup/releases/download/v1.0/docker-19.03.5-dev_riscv64.deb
sudo apt install ./docker-19.03.5-dev_riscv64.deb

docker info

docker images

cd ~/riscv-bringup/echo-demo
cat main.go
make docker

docker run --name hello -d -p 8085:8080 carlosedp/echo-riscv

docker ps

docker logs -f hello

curl http://localhost:8085

docker rm -f hello
cd -
```

OpenFaaS

```bash
wget https://github.com/carlosedp/riscv-bringup/releases/download/v1.0/faas-cli-riscv64.gz
gzip -d faas-cli-riscv64.gz
sudo mv faas-cli-riscv64 /usr/local/bin/faas-cli
sudo chmod +x /usr/local/bin/faas-cli

docker swarm init

git clone https://github.com/carlosedp/faas
cd faas
git checkout riscv64
./deploy_stack.sh

# Recover password

docker service rm print-password \
 ; docker service create --detach --secret basic-auth-password \
   --restart-condition=none --name=print-password \
   carlosedp/debian:sid-slim cat /run/secrets/basic-auth-password

docker service logs print-password

# On another window
watch docker service ls

echo -n testsecret | faas-cli login --username=admin --password-stdin

faas-cli ls

# Open URLs
http://localhost:8080
http://localhost:9090

# Login: admin
# Password: testsecret

faas-cli deploy --image=carlosedp/faas-figlet:riscv64 --name=figlet
faas-cli deploy --image=carlosedp/faas-markdownrender:riscv64 --name=markdown
faas-cli deploy --image=carlosedp/faas-qrcode:riscv64 --name=qrcode

echo "Hello Risc-V Summit\!" | faas-cli invoke figlet
echo "#Hello Risc-V Summit\n How are you doing today\?" | faas-cli invoke markdown

faas-cli list

# Remove all

docker service ls --filter="label=function" -q | xargs docker service rm

docker stack rm func && \
   docker secret rm basic-auth-user && \
   docker secret rm basic-auth-password

```

## Demo

```bash
ssh -p 22222 -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no carlosedp@localhost

uname -a

# Golang
go version

cat hello.go

go run hello.go

# Docker
docker info
docker version

docker images

cd ~/riscv-bringup/echo-demo
cat main.go
make docker

docker run --name hello -d -p 8085:8080 carlosedp/echo-riscv

docker ps

docker logs -f hello

curl http://localhost:8085

docker rm -f hello
cd -

# OpenFaas

watch docker service ls

echo -n f8f6c2b41cc64e14b29eb9bc8a298186dc2d1176fea2bac2ac3e24538cf90e4d  | faas-cli login --username=admin --password-stdin
faas-cli ls

# Open URLs
http://localhost:8080
http://localhost:9090
http://localhost:9090/graph?g0.range_input=15m&g0.stacked=0&g0.expr=gateway_function_invocation_total&g0.tab=0

faas-cli deploy --image=carlosedp/faas-figlet:riscv64 --name=figlet
faas-cli deploy --image=carlosedp/faas-markdownrender:riscv64 --name=markdown
faas-cli deploy --image=carlosedp/faas-qrcode:riscv64 --name=qrcode

echo "Hello Risc-V Summit\!" | faas-cli invoke figlet
echo '#Hello Risc-V Summit\n How are you doing today?' | faas-cli invoke markdown

curl -s http://localhost:8080/function/qrcode -d "riscv.org" > ./Desktop/qrcode.png
open ./Desktop/qrcode.png

echo 'Hello Risc-V Summit!\n #riscvsummit' | curl -s -d @- http://localhost:8080/function/figlet

faas-cli list

## Echo app on Kubernetes

```bash
#!/bin/bash

# Create deployment
kubectl create deployment echo -n default --image=carlosedp/echo-riscv

# Expose service inside the cluster
kubectl expose deploy echo -n default --type=NodePort --port=80 --target-port=8080

# Create an ingress
cat <<EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: echo
  namespace: default
spec:
  rules:
  - host: echo.192.168.15.16.nip.io
    http:
      paths:
      - path: /
        backend:
          serviceName: echo
          servicePort: 80
EOF

echo "Deployment done!"
```
