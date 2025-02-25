#!/usr/bin/make

# Default target. Want this to be first.
compile:
	go build ./...

# INPUT ENVIRONMENT VARIABLES
TIMEOUT?=168h

NAMESPACE?=
CONTEXT?=
FDB_VERSION?=7.1.31
PREVIOUS_FDB_VERSION?=6.3.25
## Expectation is that you are running standard build image which generates both regular and debug (Symbols) images.
FDB_IMAGE?=foundationdb/foundationdb:$(FDB_VERSION)
SIDECAR_IMAGE?=foundationdb/foundationdb-kubernetes-sidecar:$(FDB_VERSION)-1
OPERATOR_IMAGE?=foundationdb/fdb-kubernetes-operator:latest
REGISTRY?=docker.io
CLEANUP?=true
TEST_USERNAME?=$(USER)
ENABLE_CHAOS_TESTS?=true
CHAOS_NAMESPACE?=chaos-testing
STORAGE_CLASS?=
# Multiple versions can be specified for these upgrades by separating them with a, e.g. 6.2.25:7.1.25,7.1.23:7.1.25
UPGRADE_VERSIONS?="$(PREVIOUS_FDB_VERSION):$(FDB_VERSION)"
# Those are feature flags for the operator tests. Enable a feature if you want to run the operator tests with a specific
# feature enabled e.g. like DNS.
FEATURE_UNIFIED_IMAGE?=false
FEATURE_DNS?=false
FEATURE_LOCALITIES?=false

# Make bash pickier about errors.
SHELL=/bin/bash -euo pipefail

clean:
	@rm -f TEST-ginkgo*.xml
	@find . -name '*~' | xargs rm -f

# This target can be used to delete namespaces that were created during testing. If a namespace is provided the user and
# the namespace (as a prefix) must match.
destroy-my-namespaces:
	./remove_namespaces ${TEST_USERNAME} ${NAMESPACE}

# No one in the team can figure out how to get ginkgo's test launcher to properly
# manage concurrency, or control which tests are run in a predictable manner.

# This makefile reimplements that, and works around a few bugs in ginkgo itself.
ifndef QUIET
  VERBOSE=-v
  GINKGO_VERBOSE=-ginkgo.v
else
  VERBOSE=
  GINKGO_VERBOSE=
endif

# ginkgo doesn't understand unix output redirection, so we have to implement this bit for it.
ifdef MAKE_TERMOUT
	NO_COLOR=
else
	NO_COLOR=--ginkgo.no-color
endif

SUITES=$(wildcard test_*)
TARGETS=$(patsubst test_%,test_%.run,${SUITES})

run: ${TARGETS}

%.run: %
	go test -timeout=$(TIMEOUT) $(VERBOSE) ./$< \
	${NO_COLOR} ${GINKGO_VERBOSE} \
	  --ginkgo.junit-report=$@.xml \
	  --ginkgo.timeout=$(TIMEOUT) \
	  --timeout=$(TIMEOUT) \
	  --namespace="$(NAMESPACE)" \
	  --context="$(CONTEXT)" \
	  --fdb-image="$(FDB_IMAGE)" \
	  --sidecar-image="$(SIDECAR_IMAGE)" \
	  --operator-image="$(OPERATOR_IMAGE)" \
	  --registry="$(REGISTRY)" \
	  --fdb-version="$(FDB_VERSION)" \
	  --cleanup=$(CLEANUP) \
	  --username="$(TEST_USERNAME)" \
	  --chaos-namespace="$(CHAOS_NAMESPACE)" \
	  --storage-class="$(STORAGE_CLASS)" \
	  --enable-chaos-tests=$(ENABLE_CHAOS_TESTS) \
	  --upgrade-versions=$(UPGRADE_VERSIONS) \
	  --feature-unified-image=$(FEATURE_UNIFIED_IMAGE) \
	  --feature-localities=$(FEATURE_LOCALITIES) \
	  --feature-dns=$(FEATURE_DNS) \
	  | grep -v 'constructing many client instances from the same exec auth config can cause performance problems during cert rotation'
