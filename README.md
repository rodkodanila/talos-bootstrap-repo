# to-do
```
[] talos bootstrap process 
[] base install (cilium,flux-operator,flux)
[] flux repo sync
[] keycloak (integration with ldap, 2fa)
[] okd dahsboard or headlamp or other dashboard
```

# configure mirror if airgapped for:
1. ghcr.io
2. registry.k8s.io

# gen secrets 
```bash
talosctl gen secrets -o secrets.yaml
```
# set up ips 
```bash
CONTROL_PLANE_IP=("<control-plane-ip-1>" "<control-plane-ip-2>" "<control-plane-ip-3>")
WORKER_IP=("<worker-ip-1>")
VIP=dns
CLUSTER_NAME=<your_cluster_name>
```

# generate patch files for mirroring registry
```bash
cat << EOF > ./01-patch-registry-mirrors.yaml
---
apiVersion: v1alpha1
kind: RegistryMirrorConfig
name: ghcr.io
endpoints:
    - url: https://nexus.danila.home/air-gapped-docker-proxy
      overridePath: true
skipFallback: true

---
apiVersion: v1alpha1
kind: RegistryMirrorConfig
name: registry.k8s.io
endpoints:
    - url: https://nexus.danila.home/air-gapped-docker-proxy
      overridePath: true
skipFallback: true

---
apiVersion: v1alpha1
kind: RegistryAuthConfig
name: nexus.danila.home
username: talos
password: talos
EOF
```

```bash
cat << EOF > ./01-patch-dns-config.yaml
---
apiVersion: v1alpha1
kind: ResolverConfig
nameservers:
    - address: 192.168.10.101
EOF
```

```bash
cat << EOF > ./01-patch-ntp-config.yaml
---
apiVersion: v1alpha1
kind: TimeSyncConfig
ntp:
    servers:
        - 192.168.10.101
EOF
```

```bash
cat << EOF > ./01-patch-custom-ca-config.yaml
---
apiVersion: v1alpha1
kind: TrustedRootsConfig
name: danila-home-ca
certificates: | 
	-----BEGIN CERTIFICATE-----
	MIIEoTCCAwmgAwIBAgIQTAecfE8A32E8XbmNdg0DDDANBgkqhkiG9w0BAQsFADBp
	MR4wHAYDVQQKExVta2NlcnQgZGV2ZWxvcG1lbnQgQ0ExHzAdBgNVBAsMFnJvb3RA
	ZGViaWFuMTItdGVtcGxhdGUxJjAkBgNVBAMMHW1rY2VydCByb290QGRlYmlhbjEy
	LXRlbXBsYXRlMB4XDTI2MDEwOTE3NDYzNVoXDTM2MDEwOTE3NDYzNVowaTEeMBwG
	A1UEChMVbWtjZXJ0IGRldmVsb3BtZW50IENBMR8wHQYDVQQLDBZyb290QGRlYmlh
	bjEyLXRlbXBsYXRlMSYwJAYDVQQDDB1ta2NlcnQgcm9vdEBkZWJpYW4xMi10ZW1w
	bGF0ZTCCAaIwDQYJKoZIhvcNAQEBBQADggGPADCCAYoCggGBAOAcOQtMhfYN9eXL
	iu1PLggoS5tAR+G/MYxPJ1GZ7iGy9fDharpTzBHy4ecibMIryQTmulVcJawloyH6
	4ogcC+38sDpEqfP2fYsJbokAZUrQZFsyAB9K1E6ONATNaaEVHvqLf1HQkKUiGzbl
	rUvf41hV50De/RMuZ7mGH/PqzGCrW4cSwFHGqsJwL1esN7+IbwUkLTXV4URZ3HMn
	0rdaL0P3Zye9vn0mMvpQoZyz0XBPvCztYTQWrvqQpgs0SA5F1TSWwXZ3WrW5cu8n
	2KzPJNT31jg/9D4b0YoP6Cvkb9heoI9z8NUfAVmAZmeGMRJz4SlT2d/aOUSIqY9y
	cSVMiBo/PDYdW51thCrSlP5oBjpMAHXrbavdA6uEkND4EiMJgXUUi1dkAsSCBEf5
	62Yaiqb7vGrfUmoozMzuVfOyRLP8NBmQSJ5SI7eD1wAVjpTkhOoTqXGdD3YA0Vz6
	v5vqPo+11BYV8nhY9k1hauIIchKMXa0WnUa6QeJFTgtOeOdV6QIDAQABo0UwQzAO
	BgNVHQ8BAf8EBAMCAgQwEgYDVR0TAQH/BAgwBgEB/wIBADAdBgNVHQ4EFgQUlBp5
	JuBIPYnZgLnZHlilUQNI0ocwDQYJKoZIhvcNAQELBQADggGBANhR4M0o8M2SQVnO
	XNbGR/R8/ad7/CBtKeftmTYdWANvcn4nKKQ3bSD7LlaSy7LNi/P2+7tP8k+rxqTQ
	x6M6sXyxIh/GzZxNl/C5ONg3N/f+GiXwDX6/j6iKAlxNrFY0p9/9L2IOaiMyBbto
	4kGjNSpFnOQFIXHJf0ug83N+P3bDxRyYJsxcjnFg8b/jkSHTRm3z++2ChjCoqsyJ
	DeeQ9nKP5v87RQl0x//lWUfdpCOlEiUkSrU4Pu6zh6ZDKJl9WFKdBWr5MbXut+m7
	1yg+Jjr/xcp3AkyLHDL7UDodY+xwxn4MN5T+/MP4JZH3dwIeyjlX223fz6hQBR5l
	aZtM4XgsHMVXicCM30Qhe7HmkXlWva9z0/szPkf6wJllIN1+yyjIXZbn//EAy9Y7
	PyiFK9WH1DLn2e43XexQ73KXunxGGB/p7/CX7vE/+SE+IXnXHOKv4p/WzwV+hbnK
	9Ho/RgZVUz5ryrwxz0JMuDv6CjK81IdUPtmqWrLh6toPZxEarg==
	-----END CERTIFICATE-----
EOF
```

# cluster settings
```bash
cat << EOF > ./01-patch-cluster-settings.yaml
---
machine:
  kubelet:
    nodeIP:
      validSubnets:
        # use valid network interface for k8s communication
        - 192.168.10.0/24
cluster:
  network:
    dnsDomain: cluster.local
    podSubnets:
      - 10.244.0.0/16
    serviceSubnets:
      - 10.96.0.0/12
    cni:
      name: none
EOF
```

# combine patches 
```bash
cat ./01-patch-registry-mirrors.yaml ./01-patch-dns-config.yaml ./01-patch-ntp-config.yaml ./01-patch-custom-ca-config.yaml ./01-patch-cluster-settings.yaml > 00-common-for-all-patch.yaml
```

# Generate control plane patch
```bash
cat << EOF > ./02-patch-cp-vip-settings.yaml
---
apiVersion: v1alpha1
kind: Layer2VIPConfig
name: 192.168.10.10
link: ens18
EOF
```

```bash
cat << EOF > ./02-patch-cp-settings.yaml
---
cluster:
  allowSchedulingOnControlPlanes: false
  controllerManager:
    extraArgs:
      bind-address: 0.0.0.0
  scheduler:
    extraArgs:
      bind-address: 0.0.0.0
  apiServer:
    admissionControl:
      - name: PodSecurity
        configuration:
          apiVersion: pod-security.admission.config.k8s.io/v1alpha1
          defaults:
            audit: restricted
            audit-version: latest
            enforce: baseline
            enforce-version: latest
            warn: restricted
            warn-version: latest
          exemptions:
            namespaces:
              - kube-system
            runtimeClasses: []
            usernames: []
          kind: PodSecurityConfiguration
  proxy:
    disabled: true
  discovery:
    # scan network: broadcasting  and other staff to find another talos nodes automatically (if false just vip:6443 to communicate and create cluster)
    enabled: true 
  etcd:
    advertisedSubnets:
    - 192.168.10.0/24
EOF
```

```bash
cat ./02-patch-cp-vip-settings.yaml ./02-patch-cp-settings.yaml  > 00-common-cp-patch.yaml
```

```bash
talosctl gen config \
	$CLUSTER_NAME \
	https://$VIP:6443 \
	--with-secrets secrets.yaml \
	--config-patch @00-common-for-all-patch.yaml \
	--config-patch-control-plane @00-common-cp-patch.yaml 
```

# check network and disk
```bash
talosctl --nodes <node-ip-address> get links --insecure
talosctl get disks --insecure --nodes <node-ip-address>
```

# configure static addressing
```bash
cat << EOF > ./03-cp1-patch.yaml
machine:
  network:
    interfaces:
      - interface: ens18  
        dhcp: false
        addresses:
          - 192.168.10.11/24
        #routes:
        #  - network: 0.0.0.0/0
        #    gateway: 192.168.10.1
  install:
    disk: /dev/sda
---
apiVersion: v1alpha1
kind: HostnameConfig
hostname: cp1
auto: off
EOF
```
```bash
cat << EOF > ./03-cp2-patch.yaml
machine:
  network:
    interfaces:
      - interface: ens18  
        dhcp: false
        addresses:
          - 192.168.10.12/24
        routes:
          - network: 0.0.0.0/0
            gateway: 192.168.10.1
  install:
    disk: /dev/sda
---
apiVersion: v1alpha1
kind: HostnameConfig
hostname: cp2
auto: off
EOF
```
```bash
cat << EOF > ./03-cp3-patch.yaml
machine:
  network:
    interfaces:
      - interface: ens18  
        dhcp: false
        addresses:
          - 192.168.10.13/24
        routes:
          - network: 0.0.0.0/0
            gateway: 192.168.10.1
  install:
    disk: /dev/sda
---
apiVersion: v1alpha1
kind: HostnameConfig
hostname: cp3
auto: off
EOF
```
```bash
cat << EOF > ./04-w1-patch.yaml
machine:
  network:
    interfaces:
      - interface: ens18  
        dhcp: false
        addresses:
          - 192.168.10.15/24
        routes:
          - network: 0.0.0.0/0
            gateway: 192.168.10.1
  install:
    disk: /dev/sda
---
apiVersion: v1alpha1
kind: HostnameConfig
hostname: w1
auto: off
EOF
```

# patch main configs 
```bash
for i in $(seq 1 ${#CONTROL_PLANE_IP[@]}); do
    talosctl machineconfig patch controlplane.yaml \
        --patch @03-cp${i}-patch.yaml \
        --output controlplane-${i}.yaml
done
```

```bash
for i in $(seq 1 ${#WORKER_IP[@]}); do
    talosctl machineconfig patch worker.yaml \
        --patch @04-w${i}-patch.yaml \
        --output worker-${i}.yaml
done
```

# configure initial bootstrap file
```bash
cat << EOF > ./init-bootstrap-patch.yaml
cluster:
  inlineManifests:
  - name: cilium-values
    contents: |
      apiVersion: v1
      kind: ConfigMap
      metadata:
        name: cilium-values
        namespace: kube-system
      data:
        values.yaml: |
          cluster:
            name: test-cluster
          securityContext:
            capabilities:
                ciliumAgent: "{CHOWN,KILL,NET_ADMIN,NET_RAW,IPC_LOCK,SYS_ADMIN,SYS_RESOURCE,DAC_OVERRIDE,FOWNER,SETGID,SETUID}"
                cleanCiliumState: "{NET_ADMIN,SYS_ADMIN,SYS_RESOURCE}"
          cgroup:
            autoMount:
                enabled: false
            hostRoot: /sys/fs/cgroup
          gatewayAPI:
            enabled: true
            enableAlpn: true
            enableAppProtocol: true
          k8sServiceHost: 192.168.8.192
          k8sServicePort: 6443
          kubeProxyReplacement: true
          hostFirewall:
            enabled: false
          hubble:
            enabled: false
          externalIPs:
            enabled: true
          nodePort:
            enabled: true
            addresses: 192.168.8.0/24
          loadBalancer:
            algorithm: maglev
          ipam:
            mode: kubernetes
          routingMode: native
          ipv4NativeRoutingCIDR:  10.244.0.0/16
          enableIPv4Masquerade: true
          autoDirectNodeRoutes: true
          bpf:
            masquerade: true
          ipMasqAgent:
            enabled: true
            config:
              nonMasqueradeCIDRs:
                -  10.244.0.0/16
          l2announcements:
            enabled: true
          envoy:
            enabled: false          
  - name: bootstrap-rbac
    contents: |
      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: bootstrap
        namespace: kube-system
      ---
      apiVersion: rbac.authorization.k8s.io/v1
      kind: ClusterRoleBinding
      metadata:
        name: bootstrap
      roleRef:
        apiGroup: rbac.authorization.k8s.io
        kind: ClusterRole
        name: cluster-admin
      subjects:
        - kind: ServiceAccount
          name: bootstrap
          namespace: kube-system
  - name: cluster-bootstrap
    contents: |
      apiVersion: batch/v1
      kind: Job
      metadata:
        name: cluster-bootstrap
        namespace: kube-system
      spec:
        backoffLimit: 5
        template:
          spec:
            hostNetwork: true
            serviceAccountName: bootstrap
            restartPolicy: OnFailure
            nodeSelector:
              node-role.kubernetes.io/control-plane: ""
            tolerations:
              - key: node-role.kubernetes.io/control-plane
                operator: Exists
                effect: NoSchedule
              - key: node.kubernetes.io/not-ready
                operator: Exists
                effect: NoSchedule
            volumes:
              - name: cilium-values
                configMap:
                  name: cilium-values
            containers:
              - name: bootstrap
                image: alpine/k8s:1.35.0
                imagePullPolicy: IfNotPresent

                volumeMounts:
                  - name: cilium-values
                    mountPath: /values
                #- name: flux-instance
                #  mountPath: /flux

                command:
                  - /bin/bash
                  - -ceu
                  - |
                      APISERVER="https://192.168.8.192:6443"
                      TOKEN="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"
                      CA="/var/run/secrets/kubernetes.io/serviceaccount/ca.crt"

                      cat > /tmp/kubeconfig <<EOF
                      apiVersion: v1
                      kind: Config
                      clusters:
                      - name: c
                        cluster:
                          certificate-authority: ${CA}
                          server: ${APISERVER}
                      users:
                      - name: sa
                        user:
                          token: ${TOKEN}
                      contexts:
                      - name: ctx
                        context:
                          cluster: c
                          user: sa
                      current-context: ctx
                      EOF

                      export KUBECONFIG=/tmp/kubeconfig
                      kubectl get nodes
                      echo "==> Installing Cilium"

                      helm repo add cilium https://helm.cilium.io
                      helm repo update

                      helm upgrade --install cilium cilium/cilium \
                        --namespace kube-system \
                        --values /values/values.yaml

                      echo "==> Waiting for Cilium"
                      kubectl -n kube-system rollout status ds/cilium --timeout=5m

                      echo "==> Installing Fluxcd Operator"
                      helm upgrade --install flux-operator oci://ghcr.io/controlplaneio-fluxcd/charts/flux-operator \
                        --namespace flux-system \
                        --create-namespace
EOF
```

```bash
talosctl machineconfig patch controlplane.yaml \
	--patch @init-bootstrap-patch.yaml \
	--output controlplane.yaml
```


# apply configs to the nodes
```bash
for i in $(seq 1 ${#WORKER_IP[@]}); do
    talosctl apply-config --insecure \
    --nodes $WORKER_IP[${i}] \
    --file worker-${i}.yaml
done
```
```bash
for i in $(seq 1 ${#CONTROL_PLANE_IP[@]}); do
    talosctl apply-config --insecure \
    --nodes $CONTROL_PLANE_IP[${i}] \
    --file controlplane-${i}.yaml
done
```

```bash
export TALOSCONFIG=./talosconfig
talosctl config endpoint <control_plane_IP_1> <control_plane_IP_2> <control_plane_IP_3>
talosctl bootstrap --nodes <control-plane-IP>
talosctl kubeconfig kubeconfig --nodes <control-plane-IP>
export KUBECONFIG=./kubeconfig
k get nodes
```
