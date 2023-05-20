# Imperative Commands

Please see the [kubectl commands reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands) for more information on commands.

## Helpful Aliases/Variables
Edit your `~/.bashrc` file to persist aliases and variables between shell sessions
```sh
# Open and edit the file
vi ~/.bashrc
```

Variables and Aliases to add:
```sh
export do="--dry-run=client -o yaml"    # k create deploy nginx --image=nginx $do > out.yaml
export now="--force --grace-period 0"   # k delete pod ngninx $now

# Switch the current namespace to avoid needing `-n dev-namespace` in every command
alias kn='kubectl config set-context --current --namespace' # kn dev-namespace

# Run a temporary pod for testing
alias tmp='k run tmp --image nginx:alpine -i --rm --restart Never'
# tmp -n blue-namespace -- sh -c `curl 10.109.139.54:8080`
```

```sh
# Execute the commands in the current shell. You can also start a new shell.
source ~/.bashrc
```

## Create Spec YAML for a Resource
Append to a command to output the yaml without executing real command. Can add this as a shell variable like above.
```sh
kubectl <commmand> [options] --dry-run=client -o yaml > pod.yaml
```

## Get a Resources's Spec in YAML
```sh
kubectl get <resource> <name> -o yaml > pod.yaml
```

## Describe a Resource
```sh
kubectl describe <resource> <name>
```

## Create a Pod
```sh
kubectl run <name> --image=<image>
```

### Options:
```sh
--labels key=val
--port <port> --expose # same as imperative command to create service

--dry-run=client    # do not perform the command for real
-o yaml             # output in yaml
```

```sh
kubectl set resources <resource> <name> --limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256M
```

## Add Label
```sh
kubectl label <resource> <name> label='value'  # add label
kubectl label <resource> <name> description-   # remove label
kubectl label <resource> <name> label='value' --overwrite
```

## Get a resource with label
```sh
# Get resources with a label/value pair
kubectl get <resource> -l label=value

# Get resources with a label/value in a set of values
kubectl get <resource> -l "label in(value1,value2)"

# Get resources but show label as a column
kubectl get kubectl get <resource> --label-columns=label
kubectl get <resource> -L label
```

## Annotations
Annotations are similar to labels, but serve as metadata not used in the selector/label paradigm
```sh
kubectl annotate <resource> <name> label='value'
kubectl annotate <resource> <name> description-
kubectl annotate <resource> <name> label='value' --overwrite
```


## Create a Deployment
```sh
kubectl create deployment <name> --image=<image-name> --replicas=3
```

## Edit a Deployment
```sh
kubectl edit deployment <name>
```

## Update a Deployment's Image
```sh
kubectl set image <deployment-name> <container>=<image>
```

## Deployment Rollouts
```sh
kubectl rollout history deploy <deploy-name>
kubectl rollout status deploy <deploy-name>

kubectl rollout restart deploy <deploy-name>

kubectl rollout pause deploy <deploy-name>
kubectl rollout resume deploy <deploy-name>

kubectl rollout undo deploy <deploy-name> --to-revision=<revision>
```

## Update the number of Replicas in a ReplicaSet
```sh
kubectl scale deploy <name> --replicas=5

# Autoscale the deploy with a target CPU utilization. Creates a HorizontalPodAutoscaler
kubectl autoscale deploy <name> --min=5 --max=10 --cpu-percent=80
```

## Create a Service exposing a pod
```sh
# on an existing pod:
kubectl expose pod <pod-name> --port=<port> --name <service-name> --type=<type>

# or when creating a pod:
kubectl run <pod-name> --image <image> --port <port --expose
```

This creates a service and an endpoint. See them with

```sh
kubectl get svc
kubeclt get endpoints
```

## Set Resources
```sh
kubectl set resources <resource> <name> --limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256M
```

## Set Env on a Resource
```sh
# Set a single env var
kubectl set env <resource> <name> ENV_VAR=value
# Set env from a ConfigMap
kubectl set env <resource> <name> --from=configmap/myconfigmap
```

## Edit a Service
```sh
kubectl edit service <service-name>
```

## Create a ConfigMap
```sh
kubectl create configmap
kubectl create cm
```

### Options:
```sh
--from-literal=<KEY>=<value>
--from-file=<filename>      # File of `key=val` pairs
--from-file=<key>=<filename>      # Specify a ConfigMap key
--from-env-file=<filename>  # Parses an env file differently than a regular file
```

## Create a Secret
```sh
kubectl create secret generic <name>

### Options:
```sh
--from-literal=<KEY>=<value>
--from-file=<filename>
```

## Create a ServiceAccount
```sh
kubectl create sa <name>
```

## Create a Token for a ServiceAccount
```sh
kubectl generate token <serviceaccountname>
```

```sh
# to decode a secret
kubectl get secret <name>   # grab the encoded secret from the output
echo -n <encodedsecret> | base64 -d
```

## Taint a Node
```sh
kubectl taint nodes <node-name> <label>=<value>:<taint-effect>  # Add taint
kubectl taint nodes <node-name> <label>=<value>:<taint-effect>- # Remove taint
```

### Options
```sh
For taint effect: NoSchedule, NoExecute, PreferNoSchedule
```

## Create an Ingress
```sh
kubectl create ingress <ingress-name> --rule="host/path=service:port"
```

## Replace a Running Pod
- Force deletes and recreates a pod
```sh
kubectl replace -f pod.yaml --force
```

## Create a Job
```sh
kubectl create job <name> --image=<image>

kubectl create job --from=cronjob/sample-cron-job <name>
```

## Create a CronJob
```sh
$ kubectl create cronjob <name> --image=image --schedule='0/5 * * * ?' -- [COMMAND] [args...]
```

## Create a Role & RoleBinding
```sh
kubectl create role <name> --namespace=default --verb=list,create,delete --resource=pods

kubectl create rolebinding <name> --namespace=default --role=<role-name> --user=<user-name>
kubectl create rolebinding <name> --namespace=default --role=<role-name> --serviceaccount=<sa>
```

## Create a ClusterRole and ClusterRoleBinding
```sh
kubectl create clusterrole <name> --verb=get,list,watch --resource=pods

kubectl create clusterrolebinding <name> --clusterrole=<rolename> --user=<user>
kubectl create clusterrolebinding <name> --clusterrole=<rolename> --serviceaccount=<sa>
```

# Other kubectl Commands

## Get Events
```sh
kubectl get events
```

## Get basic docs for kubectl commands
```sh
kubectl explain <command>
```

## Copy file from a pod/to a pod
```sh
# Copy a file from a pod to the local machine
kubectl cp <pod>:<remote-path> <local-path>
# Copy a file from a local machine to a pod
kubectl cp <local-path> <pod>:<remote-path>
```

## View the Kube-Apiserver
```sh
kubectl describe pod kube-apiserver-controlplane -n kube-system

vi /etc/kubernetes/manifests/kube-apiserver.yaml # to edit the config

ps -ef | grep kube-apiserver | grep <something> # to view while running
```

## View api-resources
```sh
kubectl api-resources
```

## Get Resources With a Label
```sh
kubectl get <resource> --selector label=value

kubectl get all -A --selector label=value
```

### Options
```sh
--selector label1=value1,label2=value2,label3=value3,...
```

## Logs
```sh
kubectl logs <pod>
```
### Options
```sh
-f # Stream logs
-c <container> # Specify a container if multi-container pod
-p # view logs of previous version of the specified pod
```

## Metrics
Depending on if a monitoring solution is present, such as the metrics-server
```sh
kubectl top node
kubectl top pod
```

## Edit an Ingress resource
```sh
kubectl edit ingress --namespace <ns>
```

## KubeConfig
A file at ` ~/.kube/config` that contains Users, Clusters, and Contexts. A Context specifies a User/Cluster to use.
### kubectl config commands
```sh
kubectl config view # view the kubeconfig file
kubectl config current-context
kubectl config use-context <context> # Use the specified context
kubectl config delete-context <context> # Delete context from the kubeconfig
kubectl config set-context # Set a context entry in kubeconfig

kubectl config get-clusters # Get clusters in the kubeconfig
kubectl config delete-cluster <cluster> # Delete cluster from the kubeconfig

kubectl config get-users # view the users in the kubeconfig
kubectl config delete-user <user> # Delete a user from the kubeconfig

kubectl config --kubeconfig=/path/to/config/file <command>
...
```
### Specify a config to use in a command
```sh
kubectl <command> --kubeconfig <config-name>
```

## Check permissions with auth can-i
```sh
kubectl auth can-i <command> <resource> --as <user> --namespace <ns>
```

### Options
```sh
kubectl auth can-i create deployments
kubectl auth can-i delete nodes
...
```

## kubectl convert
With the `kubectl convert` plugin (https://jamesdefabia.github.io/docs/user-guide/kubectl/kubectl_convert/) installed:
```sh
# Convert 'pod.yaml' to latest version and print to stdout.
kubectl-convert -f pod.yaml --output-version <group>/<version>
```

Alternatively, check the current api version with `kubectl explain <resource>` and then change the yaml file to use that version. Also, attempting to create a resource with a deprecated version will show which version to use instead.

```sh
kubectl create -f <file.yaml>

Warning: flowcontrol.apiserver.k8s.io/v1beta2 FlowSchema is deprecated in v1.26+, unavailable in v1.29+; use flowcontrol.apiserver.k8s.io/v1beta3 FlowSchema
```

## kubectl Aliases
```sh
po      # for PODs
rs      # for ReplicaSets
deploy  # for Deployments
svc     # for Services
sa      # for ServiceAccounts
ns      # for Namespaces
netpol  # for Network policies
ing     # for Ingresses
pv      # for Persistent Volumes
pvc     # for PersistentVolumeClaims sa for service accounts
```

View all available resources and aliases with `kubectl api-resources`
