default: test


RELEASE := multi-fluentd
TIMEOUT := 1200s
NAMESPACE := lapis-es

install:
	helm upgrade -n $(NAMESPACE) --wait --timeout=$(TIMEOUT) --install $(RELEASE) .

test: install

purge:
	helm -n $(NAMESPACE) del $(RELEASE) || true
