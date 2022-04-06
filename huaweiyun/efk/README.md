1. deploy local-storage and StorageClass

2. deploy es cluster
elastic/elasticsearch/examples/multi/make test

use make purge to delete resource
use meke delpvc to delete pvc

adminstrator is elastic/Admin123

check health
curl -k -u elastic:Admin123  https://ip:9200/_cluster/health?pretty

3. deploy kibana
elastic/kibana/examples/security/meke test
use make purge to delete resource

set kibana_system password

curl -k -XPUT -u  elastic:Admin123 https://ip:9200/_security/user/kibana_system/_password?pretty -H "Content-Type: application/json" -d '{ "password": "Admin123" }'

use elastic user login


4. deploy fluentd
fluentd/charts/fluentd-elasticsearch/make test
use make purge to delete resource

config es infomation

