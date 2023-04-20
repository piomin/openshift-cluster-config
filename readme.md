# Demo for OpenShift Cluster Configuration

This configuration can be automatically applied to the OpenShift cluster by Argo CD.

## Scenario 1: CI/CD on the local cluster

1. Create ArgoCD `Application` that automatically manages existing namespaces on the local cluster

The configuration is provided inside the `clusters` directory via Helm chart:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cluster-config
spec:
  destination:
    server: 'https://kubernetes.default.svc'
  project: default
  source:
    path: clusters
    repoURL: 'https://github.com/piomin/openshift-cluster-config.git'
    targetRevision: HEAD
    helm:
      valueFiles:
        - values-local.yaml
  syncPolicy:
    automated:
      selfHeal: true
```

We use the `values-local.yaml` file to fill Helm template. Here are the current values:
```yaml
projects:
  - name: pminkows-test
    managedBy: pminkows-cicd
    group: app-owners
  - name: pminkows-stage
    managedBy: pminkows-cicd
    group: app-owners
  - name: pminkows-prod
    managedBy: pminkows-cicd
    group: app-owners
    quotas:
      pods: '8'
      requests.memory: 4Gi
      limits.memory: 10Gi
  - name: pminkows-cicd
    group: app-owners
    quotas:
      pods: '20'
      requests.cpu: '4'
      requests.memory: 4Gi
      limits.cpu: '20'
      limits.memory: 20Gi
    default:
      limits:
        cpu: 500m
        memory: 512Mi
      requests:
        cpu: 250m
        memory: 256Mi
```

2. Create ArgoCD `Application` that automatically manages components related to the CI/CD process

The configuration is provided inside the `cicd` directory:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cluster-config
spec:
  destination:
    server: 'https://kubernetes.default.svc'
  project: default
  source:
    path: cicd
    repoURL: 'https://github.com/piomin/openshift-cluster-config.git'
    targetRevision: HEAD
  syncPolicy:
    automated:
      selfHeal: true
```

3. xxx

Why `SealedSecret` stays in Progressing status:
https://argo-cd.readthedocs.io/en/stable/faq/#why-are-resources-of-type-sealedsecret-stuck-in-the-progressing-state

Use Kustomize for patching resource:
https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/

Secrets with ArgoCD:
https://argo-cd.readthedocs.io/en/stable/operator-manual/secret-management/

# Step 1
Add cluster-admin to the `openshift-gitops-argocd-application-controller`

```shell
oc adm policy add-cluster-role-to-user cluster-admin -z openshift-gitops-argocd-application-controller -n openshift-gitops
```

Argo CD as a cluster config tool:
https://github.com/argoproj/argo-cd/issues/5886