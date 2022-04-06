default: test

include ../../../helpers/examples.mk

RELEASE := helm-es-multi
NAMESPACE := lapis-es

install:
	helm -n $(NAMESPACE) upgrade --wait --timeout=$(TIMEOUT) --install $(RELEASE)-kibana ../../

test: install goss

purge:
	helm -n $(NAMESPACE) del $(RELEASE)
