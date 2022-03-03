use  sts-mysql-pvc.yaml


#set default namespace
kubectl config set-context --current --namespace=lapis-cmn

kubectl apply -f cm-mysql.yaml
kubectl apply -f svc-headless.yaml
#password shoud be empty at first time to run it.
kubectl apply -f sts-mysql-pvc.yaml

#add labels after pod was started.
kubectl label pod mysql-0 mode=write
kubectl label pod mysql-1 mode=read
kubectl label pod mysql-2 mode=read




use mysql;
alter user 'root'@'localhost' identified by '123456';
insert into mysql.user(Host,User,Password) values('%','root',password('123456'));


grant all privileges on *.* to root@localhost identified by '123456';
grant all privileges on nacos_db.* to 'nacos'@'%' identified by 'nacos';
flush privileges;
