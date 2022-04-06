default: test

include ../../../helpers/examples.mk

PREFIX := multi
RELEASE := multi-master
ELASTICSEARCH_IMAGE := docker.elastic.co/elasticsearch/elasticsearch:$(STACK_VERSION)
TIMEOUT := 1200s
NAMESPACE := lapis-es

install:
	helm upgrade -n $(NAMESPACE) --wait --timeout=$(TIMEOUT) --install --values master.yaml $(PREFIX)-master ../../ 
	helm upgrade -n $(NAMESPACE) --wait --timeout=$(TIMEOUT) --install --values data.yaml $(PREFIX)-data ../../ 
	helm upgrade -n $(NAMESPACE) --wait --timeout=$(TIMEOUT) --install --values client.yaml $(PREFIX)-client ../../ 

test: secrets install goss

del: purge delpvc

purge:
	helm -n $(NAMESPACE) del $(PREFIX)-master || true
	helm -n $(NAMESPACE) del $(PREFIX)-data || true
	helm -n $(NAMESPACE) del $(PREFIX)-client || true
	kubectl -n $(NAMESPACE) delete secrets elastic-certificates elastic-certificate-pem elastic-certificate-crt || true
	kubectl -n $(NAMESPACE) delete secrets multi-master-certs || true

delpvc:
	kubectl -n $(NAMESPACE) delete persistentvolumeclaim/multi-data-multi-data-0 || true
	kubectl -n $(NAMESPACE) delete persistentvolumeclaim/multi-data-multi-data-1 || true
	kubectl -n $(NAMESPACE) delete persistentvolumeclaim/multi-data-multi-data-2 || true
	kubectl -n $(NAMESPACE) delete persistentvolumeclaim/multi-master-multi-master-0 || true
	kubectl -n $(NAMESPACE) delete persistentvolumeclaim/multi-master-multi-master-1 || true
	kubectl -n $(NAMESPACE) delete persistentvolumeclaim/multi-master-multi-master-2 || true

pull:
	docker pull $(ELASTICSEARCH_IMAGE)

secrets:
	docker rm -f elastic-helm-charts-certs || true
	rm -f elastic-certificates.p12 elastic-certificate.pem elastic-certificate.crt elastic-stack-ca.p12 || true
	docker run --name elastic-helm-charts-certs -i -w /tmp \
		$(ELASTICSEARCH_IMAGE) \
		/bin/sh -c " \
			elasticsearch-certutil ca --out /tmp/elastic-stack-ca.p12 --pass '' && \
			elasticsearch-certutil cert --name multi-master --dns multi-master --ca /tmp/elastic-stack-ca.p12 --pass '' --ca-pass '' --out /tmp/elastic-certificates.p12" && \
	docker cp elastic-helm-charts-certs:/tmp/elastic-certificates.p12 ./ && \
	docker rm -f elastic-helm-charts-certs && \
	openssl pkcs12 -nodes -passin pass:'' -in elastic-certificates.p12 -out elastic-certificate.pem && \
	openssl x509 -outform der -in elastic-certificate.pem -out elastic-certificate.crt && \
	kubectl -n $(NAMESPACE) create secret generic elastic-certificates --from-file=elastic-certificates.p12 && \
	kubectl -n $(NAMESPACE) create secret generic elastic-certificate-pem --from-file=elastic-certificate.pem && \
	kubectl -n $(NAMESPACE) create secret generic elastic-certificate-crt --from-file=elastic-certificate.crt && \
	rm -f elastic-certificates.p12 elastic-certificate.pem elastic-certificate.crt elastic-stack-ca.p12
