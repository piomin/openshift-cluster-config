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