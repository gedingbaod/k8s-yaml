jenkinsfile

echo ${branch}
source /etc/profile
cd lapis-ui
cnpm cache verify
cnpm install
#cnpm run build:prod --dev
#zip -r dist.zip dist
#构建docker
cnpm run build:docker --dev
cd docker
ret=$(docker images | grep lapis-ui-test | wc -l)
if [ $ret -eq 0 ];
  then
    echo "no such image."
else
    docker rmi lapis-ui-test
    echo "delete old image."
fi

docker build -t lapis-ui-test:latest .
docker tag lapis-ui-test:latest harbor.laisontech.com/lapis/lapis-ui-test:latest
docker push harbor.laisontech.com/lapis/lapis-ui-test:latest
#部署k8s
kubectl apply -n lapis-dev -f deploy.yaml