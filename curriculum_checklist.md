# Exam Curriculum Checklist
Exam curriculum as of May 2023 (`v1.26`)

## Application Design and Build (20%)
- [ ] Define, build and modify container images
- [ ] Understand Jobs and CronJobs
- [ ] Understand multi-container Pod design patterns (e.g. sidecar, init and others)
- [ ] Utilize persistent (PV, PVC) and ephemeral volumes (emptyDir, configMap, secret, etc.)

## Application Deployment (20%)
- [ ] Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary)
- [ ] Understand Deployments and how to perform rolling updates
- [ ] Use the Helm package manager to deploy existing packages

## Application Observability and Maintenance (15%)
- [ ] Understand API deprecations
- [ ] Implement probes and health checks
- [ ] Use provided tools to monitor Kubernetes applications
- [ ] Utilize container logs
- [ ] Debugging in Kubernetes

## Application Environment, Configuration, and Security (25%)
- [ ] Discover and use resources that extend Kubernetes (CustomResourceDefinition, CRD)
- [ ] Understand authentication, authorization and admission control (e.g. RBAC, modifying the apiserver to enable/disable admission plugins)
- [ ] Understanding and defining resource requirements, limits and quotas
- [ ] Understand ConfigMaps
- [ ] Create & consume Secrets
- [ ] Understand ServiceAccounts
- [ ] Understand SecurityContexts

## Services & Networking (20%)
- [ ] Demonstrate basic understanding of NetworkPolicies
- [ ] Provide and troubleshoot access to applications via services
- [ ] Use Ingress rules to expose applications
