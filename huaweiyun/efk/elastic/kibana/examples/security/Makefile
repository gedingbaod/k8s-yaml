default: test

include ../../../helpers/examples.mk

RELEASE := multi-kibana
NAMESPACE := lapis-es

install:
	helm -n $(NAMESPACE) upgrade --wait --timeout=$(TIMEOUT) --install --values values.yaml $(RELEASE) ../../

test: secrets install goss

purge:
	helm -n $(NAMESPACE) del $(RELEASE) || true
	kubectl -n $(NAMESPACE) delete secret/kibana || true

del:
	kubectl -n $(NAMESPACE) delete deployment.apps/$(RELEASE)-kibana || true
	kubectl -n $(NAMESPACE) delete service/$(RELEASE)-kibana || true
	kubectl -n $(NAMESPACE) delete secret/kibana || true
	kubectl -n $(NAMESPACE) delete secret/$(RELEASE)-credentials || true

secrets:
	encryptionkey=$$(docker run --rm busybox:1.31.1 /bin/sh -c "< /dev/urandom tr -dc _A-Za-z0-9 | head -c50") && \
	kubectl -n $(NAMESPACE) create secret generic kibana --from-literal=encryptionkey=$$encryptionkey
