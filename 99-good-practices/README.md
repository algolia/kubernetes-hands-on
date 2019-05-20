# Good practices

## Introduction

This section is a summary, a cheat sheet, of good practices for Kubernetes. It is mostly a summary of previous sections.

## Cheat Sheet

In no particular order:

### Do not use root user in containers

The container paradigm, and how it is implemented on linux, was not built with security in mind. It’s only to restrict resources, think CPU and RAM. The documentation of Docker explains [this in more detail](https://docs.docker.com/engine/security/security/).

This implies that your container should not use the user “root” to run commands, to the why see [here](https://medium.com/@mccode/processes-in-containers-should-not-run-as-root-2feae3f0df3b).

So on all your images add those two lines to make your application run with a dedicated user. Replace `algolia` with a name more relevant for you.

```docker
RUN groupadd -g 999 algolia && useradd -r -u 999 -g algolia algolia
USER algolia
```

### Linting manifests

YAML can be a [tricky](https://docs.saltstack.com/en/latest/topics/troubleshooting/yaml_idiosyncrasies.html) format.

We recommand to use [`yamllint`](https://github.com/adrienverge/yamllint). Compared to other YAML linter. It has the nice feature of supporting multi-documents in a single file. The file [yamllint](./yamllint) is a good configuration for this tool.

You can also use Kubernetes specifics linter. [kube-score](https://github.com/zegl/kube-score) lints your manifests and enforce good practices. [kubeval](https://github.com/instrumenta/kubeval) also lints the manifests, but only checks if they are valid.

In Kubernetes 1.13 the option [`--dry-run`](https://Kubernetes.io/blog/2019/01/14/apiserver-dry-run-and-kubectl-diff/) appeared on “kubectl”. You could also use this feature to know if your YAML are valid for Kubernetes.

### Handle `SIGTERM` signal in your applications

Kubernetes sends this signal when it wants to stop a container. You should listen to it and react accordingly to your application (close connections, save a state, etc.).

### Probes

Define [liveness and readiness probes](../12-probes) for your containers.

### Resources request and limits

Define [resources](../13-resources) for your containers.

### Pod (anti-)affinity

Specify an [anti-affinity](../14-affinity-anti-affinity) for the pods of your deployements.

### PDB

Specify a [PDB](../15-pdb) for your deployments.

## Other good practices

Not directly related to Kubernetes, but still usefull:

1. If you are in the cloud, use [`terraform`](https://www.terraform.io/) to configure your clusters.
