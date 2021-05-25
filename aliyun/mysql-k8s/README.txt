
#set default namespace
kubectl config set-context --current --namespace=lapis-cmn

kubectl patch storageclass alicloud-disk-essd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

kubectl apply -f cm-mysql.yaml
kubectl apply -f svc-headless.yaml
#password shoud be empty at first time to run it.
kubectl apply -f sts-mysql-pvc.yaml

#add labels after pod was started.
kubectl label pod mysql-0 mode=write
kubectl label pod mysql-1 mode=read
kubectl label pod mysql-2 mode=read
