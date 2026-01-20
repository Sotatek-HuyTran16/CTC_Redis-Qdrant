# 1. Fetch bootstrap images

## Config docker

```bash
# nano /etc/docker/daemon.json

{
  "insecure-registries": ["registry.dev-dify.ctc.local:8080"]
}
```

## Create sync-bootstrap-images.yaml file

```bash
version: 1

defaults:
  skipDockerConfig: true
  interval: 60m

creds:
  - registry: ghcr.io
  - registry: registry.dev-dify.ctc.local:8080
    user: admin
    pass: sotatek
    tls: disabled

sync:
  - source: ghcr.io/canonical/cilium-operator-generic:1.17.9-ck4
    target: registry.dev-dify.ctc.local:8080/canonical/cilium-operator-generic:1.17.9-ck4
    type: image

  - source: ghcr.io/canonical/cilium:1.17.9-ck9
    target: registry.dev-dify.ctc.local:8080/canonical/cilium:1.17.9-ck9
    type: image

  - source: ghcr.io/canonical/coredns:1.12.4-ck1
    target: registry.dev-dify.ctc.local:8080/canonical/coredns:1.12.4-ck1
    type: image

  - source: ghcr.io/canonical/csi-node-driver-registrar:2.11.1-ck7
    target: registry.dev-dify.ctc.local:8080/canonical/csi-node-driver-registrar:2.11.1-ck7
    type: image

  - source: ghcr.io/canonical/csi-provisioner:5.0.2-ck1
    target: registry.dev-dify.ctc.local:8080/canonical/csi-provisioner:5.0.2-ck1
    type: image

  - source: ghcr.io/canonical/csi-resizer:1.11.2-ck1
    target: registry.dev-dify.ctc.local:8080/canonical/csi-resizer:1.11.2-ck1
    type: image

  - source: ghcr.io/canonical/csi-snapshotter:8.0.2-ck1
    target: registry.dev-dify.ctc.local:8080/canonical/csi-snapshotter:8.0.2-ck1
    type: image

  - source: ghcr.io/canonical/frr:9.1.3-ck5
    target: registry.dev-dify.ctc.local:8080/canonical/frr:9.1.3-ck5
    type: image

  - source: ghcr.io/canonical/k8s-snap/pause:3.10
    target: registry.dev-dify.ctc.local:8080/canonical/k8s-snap/pause:3.10
    type: image

  - source: ghcr.io/canonical/metallb-controller:v0.14.9-ck4
    target: registry.dev-dify.ctc.local:8080/canonical/metallb-controller:v0.14.9-ck4
    type: image

  - source: ghcr.io/canonical/metallb-speaker:v0.14.9-ck4
    target: registry.dev-dify.ctc.local:8080/canonical/metallb-speaker:v0.14.9-ck4
    type: image

  - source: ghcr.io/canonical/metrics-server:0.8.0-ck4
    target: registry.dev-dify.ctc.local:8080/canonical/metrics-server:0.8.0-ck4
    type: image

  - source: ghcr.io/canonical/rawfile-localpv:0.8.2-ck3
    target: registry.dev-dify.ctc.local:8080/canonical/rawfile-localpv:0.8.2-ck3
    type: image
```

## Check

```
regsync check -c sync-bootstrap-images.yaml
```

## Fetch

```
regsync once -c sync-bootstrap-images.yaml
```

# 2. Fetch Calico images

## Get list images

```
docker run quay.io/tigera/operator:v1.40.3 --print-images=list
```

## Create regsync file (Calico only)

```bash
version: 1

defaults:
  skipDockerConfig: true
  interval: 60m

creds:
  - registry: quay.io
  - registry: registry.dev-dify.ctc.local:8080
    user: admin
    pass: sotatek   # nên chuyển sang file/secret
    tls: disabled

sync:
  - source: quay.io/tigera/operator:v1.40.3
    target: registry.dev-dify.ctc.local:8080/tigera/operator:v1.40.3
    type: image

  - source: quay.io/calico/ctl:master
    target: registry.dev-dify.ctc.local:8080/calico/ctl:master
    type: image

  - source: quay.io/calico/cni
    target: registry.dev-dify.ctc.local:8080/calico/cni
    type: repository
    tags:
      allow:
      - "v3.31.3"
      - "v3.31.3-fips"
    referrers: true
    digestTags: true
    fastCopy: true

  - source: quay.io/calico/cni-windows:v3.31.3
    target: registry.dev-dify.ctc.local:8080/calico/cni-windows:v3.31.3
    type: image

  - source: quay.io/calico/key-cert-provisioner:v3.31.3
    target: registry.dev-dify.ctc.local:8080/calico/key-cert-provisioner:v3.31.3
    type: image

  - source: quay.io/calico/kube-controllers
    target: registry.dev-dify.ctc.local:8080/calico/kube-controllers
    type: repository
    tags:
      allow:
      - "v3.31.3"
      - "v3.31.3-fips"
    referrers: true
    digestTags: true
    fastCopy: true

  - source: quay.io/calico/node
    target: registry.dev-dify.ctc.local:8080/calico/node
    type: repository
    tags:
      allow:
      - "v3.31.3"
      - "v3.31.3-fips"
    referrers: true
    digestTags: true
    fastCopy: true

  - source: quay.io/calico/node-windows:v3.31.3
    target: registry.dev-dify.ctc.local:8080/calico/node-windows:v3.31.3
    type: image

  - source: quay.io/calico/typha
    target: registry.dev-dify.ctc.local:8080/calico/typha
    type: repository
    tags:
      allow:
      - "v3.31.3"
      - "v3.31.3-fips"
    referrers: true
    digestTags: true
    fastCopy: true

  - source: quay.io/calico/pod2daemon-flexvol:v3.31.3
    target: registry.dev-dify.ctc.local:8080/calico/pod2daemon-flexvol:v3.31.3
    type: image

  - source: quay.io/calico/apiserver
    target: registry.dev-dify.ctc.local:8080/calico/apiserver
    type: repository
    tags:
      allow:
      - "v3.31.3"
      - "v3.31.3-fips"
    referrers: true
    digestTags: true
    fastCopy: true

  - source: quay.io/calico/csi
    target: registry.dev-dify.ctc.local:8080/calico/csi
    type: repository
    tags:
      allow:
      - "v3.31.3"
      - "v3.31.3-fips"
    referrers: true
    digestTags: true
    fastCopy: true

  - source: quay.io/calico/node-driver-registrar
    target: registry.dev-dify.ctc.local:8080/calico/node-driver-registrar
    type: repository
    tags:
      allow:
      - "v3.31.3"
      - "v3.31.3-fips"
    referrers: true
    digestTags: true
    fastCopy: true

  - source: quay.io/calico/goldmane:v3.31.3
    target: registry.dev-dify.ctc.local:8080/calico/goldmane:v3.31.3
    type: image

  - source: quay.io/calico/whisker:v3.31.3
    target: registry.dev-dify.ctc.local:8080/calico/whisker:v3.31.3
    type: image

  - source: quay.io/calico/whisker-backend:v3.31.3
    target: registry.dev-dify.ctc.local:8080/calico/whisker-backend:v3.31.3  
    type: image

  - source: quay.io/calico/envoy-gateway:v3.31.3
    target: registry.dev-dify.ctc.local:8080/calico/envoy-gateway:v3.31.3
    type: image

  - source: quay.io/calico/envoy-proxy:v3.31.3
    target: registry.dev-dify.ctc.local:8080/calico/envoy-proxy:v3.31.3
    type: image

  - source: quay.io/calico/envoy-ratelimit:v3.31.3
    target: registry.dev-dify.ctc.local:8080/calico/envoy-ratelimit:v3.31.3
    type: image

  - source: quay.io/calico/guardian:v3.31.3
    target: registry.dev-dify.ctc.local:8080/calico/guardian:v3.31.3
    type: image
```

## Create regsync file (Calico enterprise with tigera)

```bash
version: 1

defaults:
  skipDockerConfig: true
  interval: 60m

creds:
  - registry: quay.io
  - registry: registry.dev-dify.ctc.local:8080
    user: admin
    pass: sotatek   # nên chuyển sang file/secret
    tls: disabled

sync:
  - source: quay.io/tigera/operator:v1.40.3
    target: registry.dev-dify.ctc.local:8080/tigera/operator:v1.40.3
    type: image

  - source: quay.io/tigera/apiserver:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/apiserver:release-calient-v3.22
    type: image

  - source: quay.io/tigera/compliance-benchmarker:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/compliance-benchmarker:release-calient-v3.22
    type: image

  - source: quay.io/tigera/compliance-controller:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/compliance-controller:release-calient-v3.22
    type: image

  - source: quay.io/tigera/compliance-reporter:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/compliance-reporter:release-calient-v3.22
    type: image

  - source: quay.io/tigera/compliance-server:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/compliance-server:release-calient-v3.22
    type: image

  - source: quay.io/tigera/compliance-snapshotter:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/compliance-snapshotter:release-calient-v3.22
    type: image

  - source: quay.io/tigera/key-cert-provisioner:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/key-cert-provisioner:release-calient-v3.22
    type: image

  - source: quay.io/tigera/deep-packet-inspection:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/deep-packet-inspection:release-calient-v3.22
    type: image

  - source: quay.io/tigera/intrusion-detection-job-installer:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/intrusion-detection-job-installer:release-calient-v3.22
    type: image

  - source: quay.io/tigera/elasticsearch:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/elasticsearch:release-calient-v3.22
    type: image

  - source: quay.io/tigera/eck-operator:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/eck-operator:release-calient-v3.22
    type: image

  - source: quay.io/tigera/ui-apis:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/ui-apis:release-calient-v3.22
    type: image

  - source: quay.io/tigera/fluentd:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/fluentd:release-calient-v3.22
    type: image

  - source: quay.io/tigera/fluentd-windows:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/fluentd-windows:release-calient-v3.22
    type: image

  - source: quay.io/tigera/guardian:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/guardian:release-calient-v3.22
    type: image

  - source: quay.io/tigera/intrusion-detection-controller:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/intrusion-detection-controller:release-calient-v3.22
    type: image

  - source: quay.io/tigera/waf-http-filter:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/waf-http-filter:release-calient-v3.22
    type: image

  - source: quay.io/tigera/webhooks-processor:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/webhooks-processor:release-calient-v3.22
    type: image

  - source: quay.io/tigera/kibana:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/kibana:release-calient-v3.22
    type: image

  - source: quay.io/tigera/manager:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/manager:release-calient-v3.22
    type: image

  - source: quay.io/tigera/dex:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/dex:release-calient-v3.22
    type: image

  - source: quay.io/tigera/voltron:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/voltron:release-calient-v3.22
    type: image

  - source: quay.io/tigera/packetcapture:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/packetcapture:release-calient-v3.22
    type: image

  - source: quay.io/tigera/policy-recommendation:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/policy-recommendation:release-calient-v3.22
    type: image

  - source: quay.io/tigera/egress-gateway:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/egress-gateway:release-calient-v3.22
    type: image

  - source: quay.io/tigera/l7-collector:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/l7-collector:release-calient-v3.22
    type: image

  - source: quay.io/tigera/gateway-l7-collector:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/gateway-l7-collector:release-calient-v3.22
    type: image

  - source: quay.io/tigera/envoy:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/envoy:release-calient-v3.22
    type: image

  - source: quay.io/tigera/prometheus:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/prometheus:release-calient-v3.22
    type: image

  - source: quay.io/tigera/prometheus-service:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/prometheus-service:release-calient-v3.22
    type: image

  - source: quay.io/tigera/alertmanager:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/alertmanager:release-calient-v3.22
    type: image

  - source: quay.io/tigera/queryserver:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/queryserver:release-calient-v3.22
    type: image

  - source: quay.io/tigera/kube-controllers:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/kube-controllers:release-calient-v3.22
    type: image

  - source: quay.io/tigera/node:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/node:release-calient-v3.22
    type: image

  - source: quay.io/tigera/node-windows:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/node-windows:release-calient-v3.22
    type: image

  - source: quay.io/tigera/typha:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/typha:release-calient-v3.22
    type: image

  - source: quay.io/tigera/cni:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/cni:release-calient-v3.22
    type: image

  - source: quay.io/tigera/cni-windows:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/cni-windows:release-calient-v3.22
    type: image

  - source: quay.io/tigera/elasticsearch-metrics:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/elasticsearch-metrics:release-calient-v3.22
    type: image

  - source: quay.io/tigera/es-gateway:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/es-gateway:release-calient-v3.22
    type: image

  - source: quay.io/tigera/linseed:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/linseed:release-calient-v3.22
    type: image

  - source: quay.io/tigera/dikastes:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/dikastes:release-calient-v3.22
    type: image

  - source: quay.io/tigera/l7-admission-controller:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/l7-admission-controller:release-calient-v3.22
    type: image

  - source: quay.io/tigera/pod2daemon-flexvol:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/pod2daemon-flexvol:release-calient-v3.22
    type: image

  - source: quay.io/tigera/csi:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/csi:release-calient-v3.22
    type: image

  - source: quay.io/tigera/node-driver-registrar:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/node-driver-registrar:release-calient-v3.22
    type: image

  - source: quay.io/tigera/envoy-gateway:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/envoy-gateway:release-calient-v3.22
    type: image

  - source: quay.io/tigera/envoy-proxy:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/envoy-proxy:release-calient-v3.22
    type: image

  - source: quay.io/tigera/envoy-ratelimit:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/envoy-ratelimit:release-calient-v3.22
    type: image

  - source: quay.io/tigera/istio-pilot:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/istio-pilot:release-calient-v3.22
    type: image

  - source: quay.io/tigera/istio-install-cni:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/istio-install-cni:release-calient-v3.22
    type: image

  - source: quay.io/tigera/istio-ztunnel:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/istio-ztunnel:release-calient-v3.22
    type: image

  - source: quay.io/tigera/istio-proxyv2:release-calient-v3.22
    target: registry.dev-dify.ctc.local:8080/tigera/istio-proxyv2:release-calient-v3.22
    type: image

  - source: quay.io/tigera/operator:v1.40.3
    target: registry.dev-dify.ctc.local:8080/tigera/operator:v1.40.3
    type: image
```


## Check

```
regsync check -c sync-calico-images.yaml
```

## Fetch

```
regsync once -c sync-calico-images.yaml
```