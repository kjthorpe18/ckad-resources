# K8s Objects Configuration YAML

## SecurityContexts
For pod-level security context:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000     # Run any container processes as a user 1000
    runAsGroup: 3000    # specifies the primary group ID of 3000 for all processes within any containers
    fsGroup: 2000       # if field is specified, all processes of the container are also part of the supplementary group
  containers:
  - name: sec-ctx-demo
    image: busybox:1.28
    securityContext:
      allowPrivilegeEscalation: false
```
or for container-level security context:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  containers:
  - name: sec-ctx-4
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      runAsUser: 2000   # Run processes as a certain user
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]  # grant certain privileges to a process without granting all the privileges of the root user
```

## Readiness/Liveness/Startup Probes
- Options: for probes include httpGet, tcpSocket, and exec commands

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-example
spec:
  containers:
  - name: liveness-example
    image: nginx
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      failureThreshold: 1
      periodSeconds: 60
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

## Container Resource Management
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

## Container with commands
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
  restartPolicy: OnFailure
```
or, to run a command in a shell:
```yaml
...
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 10; done"]
  restartPolicy: OnFailure
```

## Pod Placement
Place a pod on a node using NodeSelector:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
  nodeSelector:
    label: value1  # only place on nodes with this label-value pair
```

or, using NodeAffinity:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
  affinity:
    nodeAffinity:
      # Defines whether this is required or preferred for placement.
      # See preferredDuringSchedulingIgnoredDuringExecution.
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions: # place on nodes that match this criteria
          - key: label
            operator: In
            values:
            - value1
```

## Taints and Tolerations
Taint a node:
```sh
kubectl taint nodes <node-name> <label>=<value>:<taint-effect>
```

Tolerate a taint. This will ensure a pod is placed on controlplane, tolerating the taint that prevents pods from being scheduled there.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  image:
    name: nginx
    image: nginx
  nodeSelector:
    kubernetes.io/hostname: controlplane
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
```


## InitContainer
A container that is run on pod startup. Pretty much the same structure as a regular container.
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-example
spec:
  containers:
  - name: liveness-example
    image: nginx
  initContainers:
  - name: init-container-example
    image: busybox
    command:
    - "sleep"
    - "3600"
```

## Deployments
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: default
spec:
  replicas: 4               # number of pods to deploy
  selector:
    matchLabels:            # should match the label in the pods, to identify which belong to the deployment
      name: webapp
  strategy:                 # defines how to perform udpates/rollbacks to the pods
    type: RollingUpdate
    rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
  template:                 # the Pod spec
    metadata:
      labels:
        name: webapp
    spec:
      containers:
      - image: kodekloud/webapp-color:v2
        name: simple-webapp
        ports:
        - containerPort: 8080
          protocol: TCP
```

Or, for a Recreate strategy (all at once)
```yaml
spec:
  strategy:
    type: Recreate
```

## Jobs and CronJobs
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-name
spec:
  completions: 3    # Num of times the job needs to complete for job to succeed
  parallelism: 3    # Num of jobs to run in parallel
  backoffLimit: 25  # Num of times to retry before considering the job failed
  template:
    spec:
      containers:
      - name: container-name
        image: image/name
      restartPolicy: Never  # Policy to use when the overall job fails. Never, OnFailure, Always (should not be used for Job, but is the default if not set)
```

```yaml
apiVersion: batch/v1
kind: CronJob # This and spec.schedule are the only difference between a CronJob and a Job
metadata:
  name: job-name
spec:
  schedule: "30 21 * * *"   # Schedule to run the CronJob on
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      backoffLimit: 25
      template:
        spec:
          containers:
          - name: container-name
            image: image/name
          restartPolicy: Never
```

## Services
See types of services: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP       # Default ClusterIP type if not specified
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80          # Port the service will receive cluster traffic on
      targetPort: 9376  # Port on the pod to direct traffic to
```
Note: the `POD spec.containers[0].ports[0].containerPort` property is mostly informational, and does not affect connectivity

## ConfigMaps
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  my-key: <base64-encoded-value>
```

## Secrets
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  my-key: <base64-encoded-value>
```

## Ingress Controller
A special nginx (or other) image, Service, ServiceAccount, and ConfigMap can be used to allow the use of Ingress resources.

## Ingress Resource
May be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting. These resources are essentially routing rules for your cluster. An Ingress Controller must be configured for these to work.
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx # to find: kubectl get ingressclass
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - path: /testpath      # For URLs foo.bar.com/testpath
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
      - path: /anotherpath   # For URLs with foo.bar.com/anotherpath
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 80
```
Or, for host/domain-based routing:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wildcard-host
spec:
  rules:
  - host: "foo.bar.com"       # For this specific host
    http:
      paths:
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: "*.foo.com"         # Everything else
    http:
      paths:
      - pathType: Prefix
        path: "/foo"
        backend:
          service:
            name: service2
            port:
              number: 80
```

## Network Policies
Used to define rules for ingress and egress traffic of one or more pods.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes: # Need to have either defined for that policy type to take effect
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:                # Selects particular external IP CIDR ranges to allow
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:      # Selects particular namespaces for which all Pods should be allowed as ingress sources or egress destinations
            matchLabels:
              project: myproject
        - podSelector:            # Selects particular Pods in the same namespace
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
# The above rules are joined by OR - a certain IP range OR a certain namespace OR a set of pods with a label

# Can combine types under one element to create an AND relationship (https://kubernetes.io/docs/concepts/services-networking/network-policies/#behavior-of-to-and-from-selectors)
```

## Volumes and Mounts
Persistent storage between containers in a Pod. If a container restarts, the volume is still available to a new one.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-ebs
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-ebs
      name: test-volume
  volumes:
  - name: test-volume
    # This AWS EBS volume must already exist.
    awsElasticBlockStore:
      volumeID: "<volume id>"
      fsType: ext4
```

### Options for volumes
Different types of volumes are available: https://kubernetes.io/docs/concepts/storage/volumes/#volume-types. Some examples:
```yaml
volumes:
- name: test-volume
  hostPath:         # Should not use hostPath in production, only for single node cluster
    path: /data     # directory location on host
    type: Directory # this field is optional

- name: cache-volume
  emptyDir:             # initially empty volume
    sizeLimit: 500Mi

- name: config-vol
    configMap:          # volume from a ConfigMap
      name: log-config
      items:
        - key: log_level
          path: log_level
  ...
```

## Persistent Volumes (PV)
Cluster-wide pool of storage volumes. The benefit of Persistent Volumes over regular Volumes is that a user does not have to configure storage for each pod, and update them all when changes are made.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce   # Options: ReadWriteOnce, ReadWriteMany, ReadOnlyMany
  awsElasticBlockStore:
    volumeID: "<volume id>"
    fsType: ext4
  persistentVolumeReclaimPolicy: Recycle #  Options: Retain, Delete, Recycle. Define what happens to the PV when a claim is relinquished.
  storageClassName: aws-ebs-storage    # StorageClass to use for dynamic provisioning
```

## Persistent Volume Claims (PVC)
Requests for storage that are bound to PVs by K8s based on capacity, access mode, volume mode, storage class, and labels/selectors. There is a 1 to 1 relationship between PVs and PVCs, so no two PVCs can be bound to the same PV.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi
  storageClassName: aws-ebs-storage    # StorageClass to use for dynamic provisioning
  selector:
    matchLabels:
      release: "stable"
```

## Using PVs and PVCs in a Pod/Deployment/ReplicaSet
Use it as you would with any other kind of volume:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: myvolume
  volumes:
    - name: myvolume
      persistentVolumeClaim:
        claimName: myclaim
```

## StorageClasses
Depending on the type of PV, you have to have already created the volume in the provider(for example, an awsElasticBlockStore volume would have required you to create an EBS volume before creation). This is called Static Provisioning.

StorageClass allows you to perform Dynamic Provisioning, where a volume will be created when requested. The PV definition is no longer needed, since a PV will be created automatically.

Many options for provisioner: https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: aws-ebs-storage
provisioner: kubernetes.io/aws-ebs  # The provisioner to use. See options.
parameters:                         # Provisioner dependent.
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```

## StatefulSets
StatefulSet manages a set of stateful pods that have unique identities and a stable hostname, and ensures that pods are created or terminated in a predictable order. Each pod in a StatefulSet can be associated with a specific PVC that provides stable storage for the pod, even if it is rescheduled to a different node.

Useful for working with stateful applications, where ReplicaSets are for stateless apps.
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-statefulset
spec:
  replicas: 3
  serviceName: my-service
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: nginx
          ports:
            - containerPort: 80
          volumeMounts:
            - name: my-pvc
              mountPath: /data
  volumeClaimTemplates:
    - metadata:
        name: my-pvc
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
```

## Roles
Scoped to namespace
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: example-role        # The name of the role
  namespace: default        # The namespace to which the role applies
rules:
- apiGroups: [""]           # Blank means core group
  resources: ["pods"]       # The resources to which the role grants access
  verbs: ["get", "watch", "list"] # The permissions granted to the role
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
  resourceNames: ["blue"]   # Grant permissions on specific resources
```

## RoleBindings
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: example-rolebinding  # The name of the role binding
  namespace: default         # The namespace to which the role binding applies
subjects:
- kind: User
  name: john.doe@example.com # The user or group to which the role is bound
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: example-role        # The name of the role to which the role binding is bound
```

## ClusterRoles
Like Roles, but scoped to the entire cluster, regardless of namespace.

Some resources are "cluster-scoped", like nodes, PVs, cluster roles, cluster role bindings, and namespaces.
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: example-clusterrole  # The name of the cluster role
rules:
- apiGroups: [""]           # Blank means core group
  resources: ["pods"]       # The resources to which the cluster role grants access
  verbs: ["get", "watch", "list"] # The permissions granted to the cluster role
```

## ClusterRoleBindings
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: example-clusterrolebinding # The name of the cluster role binding
subjects:
- kind: User
  name: john.doe@example.com       # The user or group to which the cluster role is bound
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: example-clusterrole        # The name of the cluster role to which the cluster role binding is bound
  apiGroup: rbac.authorization.k8s.io
```

## AdmissionControllers
Allow, deny, or perform actions after authentication, RBAC, etc. There are two types:
- Validating: Ensures a request meets certain criteria
- Mutating: Changes an object/request. Generally occur before validating a.c.

To view those enabled by default, run: `kube-apiserver -h | grep enable-admission-plugins`. To add more, edit the `kube-apiserver.service` to add to the comma-delimited list of enable-admission-plugins.

```
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

`--enable-admission-plugins` can be used to add/enable admission controllers

`--disable-admission-plugins` can be used to disable admission controllers

For custom Admission Controllers, a Webhook server needs to be deployed which will mutate a request and/or determine whether a request is allowed.

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: example-validating-webhook-config
webhooks:
  - name: example.validating.webhook
    clientConfig:
      service:          # Service that directs traffic to the webhook server
        name: example-webhook-svc
        namespace: default
        path: "/validate"           # path to call on the service
      caBundle: "example-bundle"    # CA bundle for secure communication
    rules:
      - apiGroups: [""]
        apiVersions: ["v1"]    # API version of the resource to validate
        operations: ["CREATE", "UPDATE"] # operations on the resource to validate
        resources: ["pods"]         # type of resource to validate
```

## API Versions/Deprecations
To enable other versions for API groups:

1. Backup `kube-apiserver.yaml`
    ```sh
    cp -v /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.backup
    ```
2. Edit `kube-apiserver.yaml` to enable version
    ```sh
    vi /etc/kubernetes/manifests/kube-apiserver.yaml
    ```
    ```yaml
    - command:
      - kube-apiserver
      ...
      - --runtime-config=rbac.authorization.k8s.io/v1alpha1 # Add here
    ```
3. Wait for apiserver to restart
    ```
    kubectl get po -n kube-system
    ```

Can convert resources to newer API version with the `kubectl-convert` plugin: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin

## Custom Resource Definition
Used to define your own resources that can be created with the Kubernetes API. These resources, like all K8s resources, are stored in the etcd datastore.

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: mycrd.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true  # Enable/disable versions
      storage: true # Only one version can be marked as the storage version
  scope: Namespaced
  names:
    plural: mycrds
    singular: mycrd
    kind: MyCRD
    shortNames:
      - mc
      - mcrd
  validation:  # Validation rules for the CRD
    openAPIV3Schema:
      type: object
      properties:
        spec:
          type: object
          properties:
            replicas:
              type: integer
              minimum: 1
              maximum: 10
            image:
              type: string
      required:
        - spec
```

## Custom Controller
Handles the lifecycle of custom resources. See https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#custom-controllers

See https://github.com/kubernetes/sample-controller for an example.

The CKAD exam does not expect you to code a custom controller, but it is important to understand how CRDs work.

## ResourceQuota
Limit aggregate resource consumption per namespace. In practice, an administrator would make one of these per namespace, and different users would stay in their namespaces. The ResourceQuotas would prevent users of one namespace from consuming too many shared resources.
```yaml
 apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: test-cpu-quota
    namespace: quota-demo
  spec:
    hard:
      requests.cpu: "200m"
      limits.cpu: "300m"
```

## Limit Ranges
A pod can still consume all the resources it wants within a namespace with a resource quota. A LimitRange is a policy to constrain the resource allocations (limits and requests) that you can specify for each resource type in a namespace.
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
  namespace: my-namespace # the namespace to apply the limit to
spec:
  limits:
  - default:        # default limits
      cpu: 500m
      memory: 500Mi
    defaultRequest: # default requests
      cpu: 500m
      memory: 500Mi
    max:            # maximum allowed limits
      cpu: "1"
    min:            # minimum allowed limits
      cpu: 100m
    type: Container
```
