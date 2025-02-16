+++
title = 'Configuring ArgoCD with Zitadel via Helm'
subtitle = 'Using External Secrets for OIDC'
author = "0hlov3"
date = 2025-02-16T16:00:00
draft = false
tags = ['ArgoCD','Zitadel','Kubernetes','Helm','OIDC Authentication']
+++

When integrating ArgoCD with an OpenID Connect (OIDC) provider like Zitadel, managing secrets securely is crucial. While the {{< newtablink \"https://argo-cd.readthedocs.io/en/latest/operator-manual/user-management/zitadel/\" >}}official documentation{{< /newtablink >}} provides comprehensive guidance, this article focuses on a Helm-based setup where ArgoCD retrieves OIDC credentials from an external Kubernetes Secret.

In this guide, we'll configure ArgoCD's OIDC integration with Zitadel via Helm, ensuring that client credentials are stored securely in a Kubernetes Secret rather than being embedded in the ArgoCD ConfigMap.

## Prerequisites

Before you begin, ensure you have:

- A running Zitadel instance for authentication.
- A Kubernetes cluster with ArgoCD installed via Helm.
- Helm CLI installed (helm version to verify).
- The client credentials (Client ID and Client Secret) from Zitadel.

## Creating the Zitadel Secret
Instead of directly storing OIDC credentials in the `argocd-cm` ConfigMap, we store them in a Kubernetes Secret named `zitadel-secret`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: zitadel-secret
  namespace: argocd
type: Opaque
stringData:
  oidc.zitadel.clientId: "your-zitadel-client-id"
  oidc.zitadel.clientSecret: "your-zitadel-client-secret"
```
Apply the secret:
```bash
kubectl apply -f zitadel-secret.yaml
```

## Configuring ArgoCD via Helm
Modify the ArgoCD Helm values to reference the Zitadel OIDC provider and externalize secrets.
### Helm Values Configuration (`values.yaml`)
```yaml
configs:
  # General Argo CD configuration
  params:
    server.insecure: false
  cm:
    oidc.config: |
      name: Zitadel
      issuer: https://YOUR-ZITADEL-INSTANCE.com
      clientID: $zitadel-secret:oidc.zitadel.clientId
      clientSecret: $zitadel-secret:oidc.zitadel.clientSecret
      requestedScopes:
        - openid
        - profile
        - email
        - groups
      logoutURL: https://YOUR-ZITADEL-INSTANCE.com/oidc/v1/end_session
  rbac:
    policy.default: ''
    scopes: '[groups]'
    policy.csv: |
      g, argocd_administrators, role:admin
      g, argocd_users, role:readonly
```

Here’s how it works:
- `clientID` and `clientSecret` are referenced via `$zitadel-secret:<key>`, which means ArgoCD dynamically retrieves these values from the `zitadel-secret` Secret.
- RBAC settings ensure that users with specific Zitadel groups (`argocd_administrators, argocd_users`) get assigned roles in ArgoCD.

## Deploying ArgoCD with Helm

Run the following command to apply the configuration:
```bash
helm upgrade --install argocd argo/argo-cd -n argocd -f values.yaml
```
Verify the changes:
```bash
kubectl get secrets -n argocd | grep zitadel-secret
kubectl get cm argocd-cm -n argocd -o yaml | grep oidc
```

## Testing the Authentication Flow

1. Navigate to your ArgoCD UI.
2. Click Log in via Zitadel.
3. Authenticate with your Zitadel credentials.
4. If authentication succeeds, you should be redirected to ArgoCD’s dashboard.

## Conclusion
By referencing OIDC credentials from a Kubernetes Secret, this approach improves security by: 
- Avoiding hardcoded secrets in ArgoCD ConfigMaps.
- Enabling dynamic updates without restarting ArgoCD.
- Leveraging Kubernetes secrets management for better security.

This setup is easy to maintain and aligns with security best practices when integrating ArgoCD with an external identity provider like Zitadel.

## Sources & Further Reading
- {{< newtablink \"https://github.com/argoproj/argo-cd/issues/4188\" >}}OIDC config should reference dedicated secret instead of key in argocd-secret{{< /newtablink >}}
- {{< newtablink \"https://github.com/argoproj/argo-cd/pull/13475\" >}}feat: support referencing secret in any field of oidc config{{< /newtablink >}}
- {{< newtablink \"https://argo-cd.readthedocs.io/en/latest/operator-manual/user-management/zitadel/\" >}}Integrating Zitadel and ArgoCD{{< /newtablink >}}

## Don’t Trust Me — Seriously

The author takes no responsibility for any mishaps, broken servers, or existential crises caused by following this information.

If you spot a mistake, have a better way of doing things, or just want to chat about tech, feel free to reach out.

Also, this isn’t an ad — unless my enthusiasm and advocacy for cool stuff count as advertising.
