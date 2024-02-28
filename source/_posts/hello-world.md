系统：CentOS 8.4
Kubernetes 版本：v1.27.2
Calico 版本：v1.35
Sealos 版本：v4.3.0

安装 Sealos
wget https://jdt-1251146898.cos.ap-shanghai.myqcloud.com/sealos_4.3.0_linux_amd64.tar.gz
tar zxvf sealos_4.3.0_linux_amd64.tar.gz sealos
chmod +x sealos
mv sealos /usr/bin
上传 pem 文件到服务器
scp -i qcloud_common.pem qcloud_common.pem root@111.229.177.204:~
生成 Clusterfile 文件
sealos gen \
	labring/kubernetes:v1.27.2 \
 	labring/helm:v3.12.0 \
  labring/calico:v3.25.1 \
  --masters 172.20.5.15 \
  --nodes 172.20.5.16 \
  --user='root' \
  --pk "/root/qcloud_common.pem" > Clusterfile
修改 Clusterfile 文件
apiVersion: apps.sealos.io/v1beta1
kind: Cluster
metadata:
  creationTimestamp: null
  name: default
spec:
  hosts:
  - ips:
    - 172.20.5.15:22
    roles:
    - master
    - amd64
  - ips:
    - 172.20.5.16:22
    roles:
    - node
    - amd64
  image:
  - labring/kubernetes:v1.27.2
  - labring/helm:v3.12.0
  - labring/calico:v3.25.1
  ssh:
    pk: /root/qcloud_common.pem
    port: 22
    user: root
status: {}

---
BootstrapTokens: null
CertificateKey: ""
LocalAPIEndpoint:
  AdvertiseAddress: 172.20.5.15
  BindPort: 6443
NodeRegistration:
  CRISocket: /run/containerd/containerd.sock
  IgnorePreflightErrors: null
  KubeletExtraArgs: null
  Name: ""
  Taints: null
Patches: null
SkipPhases: null
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration

---
APIServer:
  CertSANs:
  - 127.0.0.1
  - apiserver.cluster.local
  - 10.103.97.2
  - 172.20.5.15
  - 2402:4e00:140a:1405:0:99bf:21ed:8aec #增加控制节点的可以公网访问ipv6地址，如果你需要使用此IP访问apiserver
  ExtraArgs:
    audit-log-format: json
    audit-log-maxage: "7"
    audit-log-maxbackup: "10"
    audit-log-maxsize: "100"
    audit-log-path: /var/log/kubernetes/audit.log
    audit-policy-file: /etc/kubernetes/audit-policy.yml
    enable-aggregator-routing: "true"
    feature-gates: ""
    service-cluster-ip-range: 10.96.0.0/22,fd85:ee78:d8a6:8607::1000/116 #增加svc IPv6地址段
  ExtraVolumes:
  - HostPath: /etc/kubernetes
    MountPath: /etc/kubernetes
    Name: audit
    PathType: DirectoryOrCreate
    ReadOnly: false
  - HostPath: /var/log/kubernetes
    MountPath: /var/log/kubernetes
    Name: audit-log
    PathType: DirectoryOrCreate
    ReadOnly: false
  - HostPath: /etc/localtime
    MountPath: /etc/localtime
    Name: localtime
    PathType: File
    ReadOnly: true
  - HostPath: /etc/kubernetes
    MountPath: /etc/kubernetes
    Name: audit
    PathType: DirectoryOrCreate
    ReadOnly: false
  - HostPath: /var/log/kubernetes
    MountPath: /var/log/kubernetes
    Name: audit-log
    PathType: DirectoryOrCreate
    ReadOnly: false
  - HostPath: /etc/localtime
    MountPath: /etc/localtime
    Name: localtime
    PathType: File
    ReadOnly: true
  TimeoutForControlPlane: null
CIImageRepository: ""
CIKubernetesVersion: ""
CertificatesDir: ""
ClusterName: ""
ComponentConfigs: null
ControlPlaneEndpoint: apiserver.cluster.local:6443
ControllerManager:
  ExtraArgs:
    bind-address: 0.0.0.0
    cluster-signing-duration: 876000h
    feature-gates: ""
    node-cidr-mask-size-ipv6: 120 #默认为64
    node-cidr-mask-size-ipv4: 24  #默认为24
  ExtraVolumes:
  - HostPath: /etc/localtime
    MountPath: /etc/localtime
    Name: localtime
    PathType: File
    ReadOnly: true
  - HostPath: /etc/localtime
    MountPath: /etc/localtime
    Name: localtime
    PathType: File
    ReadOnly: true
DNS:
  ImageRepository: ""
  ImageTag: ""
  Type: ""
Etcd:
  External: null
  Local:
    DataDir: ""
    ExtraArgs:
      listen-metrics-urls: http://0.0.0.0:2381
    ImageRepository: ""
    ImageTag: ""
    PeerCertSANs: null
    ServerCertSANs: null
FeatureGates: null
ImageRepository: ""
KubernetesVersion: v1.27.2
Networking:
  DNSDomain: ""
  PodSubnet: 100.64.0.0/10,fd85:ee78:d8a6:8607::1:0000/112 #增加pod IPv6地址段
  ServiceSubnet: 10.96.0.0/22,fd85:ee78:d8a6:8607::1000/116 #增加svc IPv6地址段
Scheduler:
  ExtraArgs:
    bind-address: 0.0.0.0
    feature-gates: ""
  ExtraVolumes:
  - HostPath: /etc/localtime
    MountPath: /etc/localtime
    Name: localtime
    PathType: File
    ReadOnly: true
  - HostPath: /etc/localtime
    MountPath: /etc/localtime
    Name: localtime
    PathType: File
    ReadOnly: true
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration

---
CACertPath: /etc/kubernetes/pki/ca.crt
ControlPlane:
  CertificateKey: ""
  LocalAPIEndpoint:
    AdvertiseAddress: ""
    BindPort: 6443
Discovery:
  BootstrapToken: null
  File: null
  TLSBootstrapToken: ""
  Timeout: 5m0s
NodeRegistration:
  CRISocket: /run/containerd/containerd.sock
  IgnorePreflightErrors: null
  KubeletExtraArgs: null
  Name: ""
  Taints: null
Patches: null
SkipPhases: null
apiVersion: kubeadm.k8s.io/v1beta3
kind: JoinConfiguration

---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
bindAddressHardFail: false
clientConnection:
  acceptContentTypes: ""
  burst: 10
  contentType: application/vnd.kubernetes.protobuf
  kubeconfig: ""
  qps: 5
clusterCIDR: ""
configSyncPeriod: 15m0s
conntrack:
  maxPerCore: 32768
  min: 131072
  tcpCloseWaitTimeout: 1h0m0s
  tcpEstablishedTimeout: 24h0m0s
detectLocal:
  bridgeInterface: ""
  interfaceNamePrefix: ""
detectLocalMode: ""
enableProfiling: false
healthzBindAddress: 0.0.0.0:10256
hostnameOverride: ""
iptables:
  masqueradeAll: false
  masqueradeBit: 14
  minSyncPeriod: 1s
  syncPeriod: 30s
ipvs:
  excludeCIDRs:
  - 10.103.97.2/32
  minSyncPeriod: 0s
  scheduler: ""
  strictARP: false
  syncPeriod: 30s
  tcpFinTimeout: 0s
  tcpTimeout: 0s
  udpTimeout: 0s
kind: KubeProxyConfiguration
clusterCIDR: "100.64.0.0/10,fd85:ee78:d8a6:8607::1:0000/112" #增加pod IPv6地址段
metricsBindAddress: 0.0.0.0:10249
mode: ipvs
nodePortAddresses: null
oomScoreAdj: -999
portRange: ""
showHiddenMetricsForVersion: ""
udpIdleTimeout: 250ms
winkernel:
  enableDSR: false
  forwardHealthCheckVip: false
  networkName: ""
  rootHnsEndpointName: ""
  sourceVip: ""

---
address: 0.0.0.0
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
cgroupDriver: cgroupfs
cgroupsPerQOS: true
clusterDomain: cluster.local
configMapAndSecretChangeDetectionStrategy: Watch
containerLogMaxFiles: 5
containerLogMaxSize: 10Mi
contentType: application/vnd.kubernetes.protobuf
cpuCFSQuota: true
cpuCFSQuotaPeriod: 100ms
cpuManagerPolicy: none
cpuManagerReconcilePeriod: 10s
enableControllerAttachDetach: true
enableDebugFlagsHandler: true
enableDebuggingHandlers: true
enableProfilingHandler: true
enableServer: true
enableSystemLogHandler: true
enforceNodeAllocatable:
- pods
- pods
eventBurst: 10
eventRecordQPS: 5
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
evictionPressureTransitionPeriod: 5m0s
failSwapOn: true
fileCheckFrequency: 20s
hairpinMode: promiscuous-bridge
healthzBindAddress: 0.0.0.0
healthzPort: 10248
httpCheckFrequency: 20s
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
imageMinimumGCAge: 2m0s
iptablesDropBit: 15
iptablesMasqueradeBit: 14
kind: KubeletConfiguration
kubeAPIBurst: 10
kubeAPIQPS: 5
localStorageCapacityIsolation: true
logging:
  flushFrequency: 5000000000
  format: text
  options:
    json:
      infoBufferSize: "0"
  verbosity: 0
makeIPTablesUtilChains: true
maxOpenFiles: 1000000
maxPods: 110
memoryManagerPolicy: None
memorySwap: {}
memoryThrottlingFactor: 0.8
nodeLeaseDurationSeconds: 40
nodeStatusMaxImages: 50
nodeStatusReportFrequency: 10s
nodeStatusUpdateFrequency: 10s
oomScoreAdj: -999
podPidsLimit: -1
port: 10250
registerNode: true
registryBurst: 10
registryPullQPS: 5
rotateCertificates: true
runtimeRequestTimeout: 2m0s
seccompDefault: false
serializeImagePulls: true
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 4h0m0s
syncFrequency: 1m0s
topologyManagerPolicy: none
topologyManagerScope: container
volumePluginDir: /usr/libexec/kubernetes/kubelet-plugins/volume/exec/
volumeStatsAggPeriod: 1m0s
---
# 添加Calico双栈配置
apiVersion: apps.sealos.io/v1beta1
kind: Config
metadata:
  name: calico
spec:
  path: charts/calico/values.yaml
  strategy: merge
  data: |
    installation:
      enabled: true
      kubernetesProvider: ""
      calicoNetwork:
        bgp: Disabled
        ipPools:
        - blockSize: 22
          cidr: 100.64.0.0/10
          encapsulation: VXLAN
          natOutgoing: Enabled
          nodeSelector: all()
        - blockSize: 122
          cidr: fd85:ee78:d8a6:8607::1:0000/112 #增加pod IPv6地址段
          encapsulation: VXLAN
          natOutgoing: Enabled
          nodeSelector: all()
        nodeAddressAutodetectionV4:
          interface: "eth.*|en.*|em.*"
        nodeAddressAutodetectionV6:
          interface: "eth.*|en.*|em.*"
创建集群
sealos apply -f Clusterfile