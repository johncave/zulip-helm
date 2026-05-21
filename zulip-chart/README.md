# Zulip Helm chart

## Prerequisites

- Kubernetes cluster with Ingressive Controller installed (`IngressClass` `ingressive` available).
- A default StorageClass configured (this chart omits `storageClassName` when `persistence.storageClass` is empty).
- DNS for `zulip.ingressive.dev` managed by Ingressive control plane.

## Create the secret

This chart references (but does not create) an externally-managed Secret.

```bash
kubectl create secret generic zulip-secrets -n zulip \
  --from-literal=secret_key=$(openssl rand -hex 32) \
  --from-literal=postgres_password='REPLACE_ME' \
  --from-literal=rabbitmq_password='REPLACE_ME' \
  --from-literal=redis_password='REPLACE_ME' \
  --from-literal=avatar_salt=$(openssl rand -hex 32) \
  --from-literal=shared_secret=$(openssl rand -hex 32) \
  --from-literal=initial_password_salt=$(openssl rand -hex 32)
```

`postgres_password`, `rabbitmq_password`, and `redis_password` should be real strong passwords.

## Install

```bash
helm install zulip ./zulip-chart -n zulip --create-namespace
```

## Post-install bootstrap

After pods are healthy, generate a realm creation link:

```bash
kubectl exec -n zulip deploy/zulip -- /home/zulip/deployments/current/manage.py generate_realm_creation_link
```

Open the printed URL to create the realm and first admin user.

## Out of scope

This chart intentionally does **not** include:

- SMTP configuration
- Backups/restore automation
- Multi-replica HA
- TLS certificates/cert-manager (TLS terminates at Ingressive edge)
