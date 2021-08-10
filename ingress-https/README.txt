test http
1. create deployment  demo-nginx-deploy-ssl.yaml
2. create svc         demo-nginx-svc-ssl.yaml
3. create ingress     demo-nginx-ingress-ssl-v1.yaml
4. check  controller  kubectl get pod -n ingress-nginx
5. create nodeport for ingress controller to test     demo-ingress-controller-svc.yaml
   test ssl.laisontech.com:30082 can be act   192.168.2.202
6. create crt
     openssl rand -writerand ~/.rnd
     openssl genrsa -out ssl.laisontech.com.key 2048
     openssl req -new -x509 -key ssl.laisontech.com.key -out ssl.laisontech.com.crt -subj /C=CN/ST=Shanghai/L=Shanghai/O=DevOps/CN=ssl.laisontech.com
7. create secret      kubectl create secret tls demo-secret --cert=ssl.laisontech.com.crt --key=ssl.laisontech.com.key
8. add secret to ingress
tls:
  - hosts:
      - ssl.laisontech.com
    secretName: demo-secret
9. check ssl site     access ssl.laisontech.com

test https
1. create crt
     openssl rand -writerand ~/.rnd
     openssl genrsa -out lapis.lab.laisontech.com.key 2048
     openssl req -new -x509 -key lapis.lab.laisontech.com.key -out lapis.lab.laisontech.com.crt -subj /C=CN/ST=Shanghai/L=Shanghai/O=DevOps/CN=lapis.lab.laisontech.com
2. create secret      kubectl -n lapis-lab create secret tls demo-secret --cert=lapis.lab.laisontech.com.crt --key=lapis.lab.laisontech.com.key
3. add secret to ingress lab


test lapis.lab.laisontech.com
#1. create configmap   kubectl -n lapis-lab create configmap nginx-ssl --from-file=lapis.lab.laisontech.com.crt --from-file=lapis.lab.laisontech.com.key
#2. check configmap    kubectl -n lapis-lab describe cm nginx-ssl
1. edit ingress add ssl , set port 80
