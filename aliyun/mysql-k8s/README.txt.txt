use  sts-mysql-pvc.yaml

add labels after pod was started.
kubectl label pod mysql-0 mode=write
kubectl label pod mysql-1 mode=read
kubectl label pod mysql-2 mode=read
