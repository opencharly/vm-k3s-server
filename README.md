# vm-k3s-server

The `k3s-server` candy — a single-node k3s **control plane** that boots with
ServiceLB, Traefik v2, and local-path-provisioner enabled by default — plus its
owning skill, projected into the marketplace corpus as
`/charly-infrastructure:k3s-server`.

The candy renders `/etc/rancher/k3s/config.yaml` (mode `0600`, `disable: []` so
the default addons install), runs `k3s server` as a managed systemd service, and
publishes the cluster kubeconfig back to the operator. The observable proof the
control plane came up: a live deploy reaches a `Ready` node with Traefik as the
default IngressClass and local-path as the default StorageClass.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `k3s-server` |
| Requires | `layer-k3s`, `layer-k3s-kernel`, `plugin-kube` |
| Secrets | `K3S_CLUSTER_TOKEN` — pre-shared token; auto-generated on first deploy and shared with every `k3s-agent` |
| Env accepts | `K3S_SERVER_HOSTNAME` (tls-san), `K3S_KUBECONFIG_SERVER` (optional kubeconfig server-URL override) |
| Service | `k3s.service` (system scope, enabled) |
| Artifact | `/etc/rancher/k3s/k3s.yaml` published back to the operator as `kubeconfig` |
| Addons | ServiceLB, Traefik v2, local-path-provisioner (all default) |

`K3S_CLUSTER_TOKEN` is provisioned automatically (shared with `k3s-agent`
through the credential store). `K3S_SERVER_HOSTNAME` goes into `tls-san:` so the
retrieved kubeconfig's server URL is valid; on a charly `vm:` deploy the
guest-side `:6443` is rewritten to the auto-allocated host port, so
`K3S_KUBECONFIG_SERVER` is only needed for a manual/non-charly port-forward.

## How to use it

Compose the candy into a VM deploy:

```yaml
k3s-srv-vm:
  vm:
    source: {kind: cloud_image, distro: arch, url: "…", base_user: arch}
    ram: 4G
    cpu: 2

k3s-srv:
  vm:
    from: k3s-srv-vm
    disposable: true
    add_candy: [k3s-server]
    env:
      K3S_SERVER_HOSTNAME: k3s-srv.lan
```

```bash
charly vm create k3s-srv
charly deploy add vm:k3s-srv
kubectl --context k3s-srv get nodes
```

The kubeconfig is auto-retrieved and merged into `~/.kube/config` under the
deploy's context; a matching ClusterProfile is written with
`ingress.class=traefik` and `storage.class_default=local-path`.

## Layout

- `charly.yml` — the `k3s-server:` candy entity and the `k3s-server-skill:`
  skill entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-infrastructure:k3s-server` — the candy and its checks.
- Workers: `/charly-infrastructure:k3s-agent` — nodes that join this server.
- Base binary: `/charly-infrastructure:k3s` — installs the `k3s` binary.
- Cluster probing: `/charly-kubernetes:check-k8s` — the `kube:` check verb.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder.
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella.
