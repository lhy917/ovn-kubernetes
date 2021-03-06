# build image for ovn overlay network cni plugin

# ovnkube-db.yaml, ovnkube-node.yaml, and onvkube-master.yaml use this image.
# This image is built from files in this directory and pushed to
# a docker registry that is accesseble on each node.

# For a user created registry, the registry must be setup ahead of time.
# The registry is configured in /etc/containers/registries.conf
# on each node in both "registries:" and "insecure_registries:" sections.

all: ubuntu centos fedora

SLASH = -
ARCH = $(subst aarch64,arm64,$(subst x86_64,amd64,$(patsubst i%86,386,$(shell uname -m))))
IMAGE_ARCH = $(SLASH)$(ARCH)
DOCKERFILE_ARCH =
ifeq ($(ARCH),arm64)
        DOCKERFILE_ARCH=.arm64
endif

# The image of ovnkube/ovn-daemonset-u should be multi-arched before using it on arm64
ubuntu: bld
	docker build -t ovn-kube-u$(IMAGE_ARCH) -f Dockerfile.ubuntu$(DOCKERFILE_ARCH) .
ifeq ($(ARCH),amd64)
	docker tag "ovn-kube-u$(IMAGE_ARCH):latest" \
              "ovn-kube-u:latest"
endif
	# This is the default in the ovnkube*.yaml files
	# docker login -u ovnkube docker.io/ovnkube
	# docker push docker.io/ovnkube/ovn-daemonset-u:latest
	./daemonset.sh --image=docker.io/ovnkube/ovn-daemonset-u:latest

centos: bld
	docker build -t ovn-daemonset .
	docker tag ovn-daemonset docker.io/ovnkube/ovn-daemonset:latest
	# docker login -u ovnkube docker.io/ovnkube
	# docker push docker.io/ovnkube/ovn-daemonset:latest
	./daemonset.sh --image=docker.io/ovnkube/ovn-daemonset:latest

fedora: bld
	docker build -t ovn-kube-f -f Dockerfile.fedora .
	# docker login -u ovnkube docker.io/ovnkube
	# docker push docker.io/ovnkube/ovn-daemonset-f:latest
	./daemonset.sh --image=docker.io/ovnkube/ovn-daemonset-f:latest

fedora-dev: bld
	# To build (OVN Datapath)  with specific kernel version, please override KERNEL_VERSION.
	# By default it will build with the host machine's kernel version.

	docker build --build-arg KERNEL_VERSION=`uname -r` -t ovn-kube-f-dev -f Dockerfile.fedora.dev .
	# docker login -u ovnkube docker.io/ovnkube
	# docker push docker.io/ovnkube/ovn-daemonset-f:latest
	./daemonset.sh --image=docker.io/ovnkube/ovn-daemonset-f:latest \
                    --net-cidr=10.244.0.0/16 \
                    --svc-cidr=10.96.0.0/12 \
                    --gateway-mode="local" \
                    --master-loglevel="5" \
                    --node-loglevel="5" \
                    --ovn-loglevel-northd="-vconsole:info -vfile:info" \
                    --ovn-loglevel-nb="-vconsole:info -vfile:info" \
                    --ovn-loglevel-sb="-vconsole:info -vfile:info" \
                    --ovn-loglevel-controller="-vconsole:info" \
                    --ovn-loglevel-nbctld="-vconsole:info" \
                    --ovn_nb_raft_election_timer="1000" \
                    --ovn_sb_raft_election_timer="1000"

DOCKER_IMAGE_TAG = latest

# Multi-arch the ubuntu based image with fat-manifest
ubuntu-image-multi-arch:
	./push_manifest.sh ovn-daemonset-u $(DOCKER_IMAGE_TAG)

# This target expands the daemonset yaml templates into final form
# Use CLI flags or environment variables to customize its behavior.
daemonsetyaml:
	./daemonset.sh

.PHONY: ../../go-controller/_output/go/bin/ovnkube

../../go-controller/_output/go/bin/ovnkube:
	cd ../../go-controller ; make

BRANCH = $(shell git rev-parse  --symbolic-full-name HEAD)
COMMIT = $(shell git rev-parse  HEAD)
bld: ../../go-controller/_output/go/bin/ovnkube
	cp -r ../../go-controller/_output/go/bin/* .
	echo "ref: ${BRANCH}  commit: ${COMMIT}" > git_info
