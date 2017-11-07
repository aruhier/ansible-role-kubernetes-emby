Ansible Role: Emby for Kubernetes
========================================

Ansible role to install Emby on Kubernetes.

Role Variables
--------------

```yaml
# Namespace
kubernetes_emby_namespace: "default"
# App name (used as selector)
kubernetes_emby_app: "emby"
# Deployment name
kubernetes_emby_deployment: "emby-deployment"
# Service name
kubernetes_emby_service: "emby"

# Number of replicas
kubernetes_emby_replicas: 1
kubernetes_emby_revision_history: 1

# Node selector
kubernetes_emby_node_selector: {}

# Add custom labels in the deployment metadata section
kubernetes_emby_deployment_labels: {}
# Add custom annotations in the deployment metadata section
kubernetes_emby_deployment_annotations: {}

kubernetes_emby_resources:
  limits:
    memory: "1.5Gi"
  requests:
    memory: "512Mi"

# Setup ingress for emby
kubernetes_emby_setup_ingress: false
kubernetes_emby_ingress:
  name: "emby-ingress"
  host: "emby.example.com"
  annotations:
  tls:

### Volumes ###

# For each volume, `definition` and `subPath` can be used. Their content is
# exactly what would be in a Kkubernetes pod manifest.

# Downloads volumes. Mounted in /media/ (see examples for more details)
kubernetes_emby_media_volumes: {}

# Enby config volume. Contains the database, arts cache and config.
kubernetes_emby_config_volume:
  definition:
```

Dependencies
------------

Kubectl needs to be installed on the host targeted by the role.


Example Playbook
----------------

```yaml

- hosts: kube-master
  run_once: true
  vars:
    kubernetes_emby_media_volumes:
      # This volume will be mounted as /media/tv
      tv:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-videos
            readOnly: false
        subPath: "tv"
      movies:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-videos
            readOnly: false
        subPath: "movies"
      music:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-music
            readOnly: false

    kubernetes_emby_config_volume:
      definition:
        glusterfs:
          endpoints: gluster-example-cluster
          path: volume-emby
          readOnly: false
        subPath: "config"

    kubernetes_emby_setup_ingress: true
    kubernetes_emby_ingress:
      name: "emby-ingress"
      host: "emby.example.com"
      annotations:
        kubernetes.io/tls-acme: "true"
      tls:
        - secretName: "emby-ingress-tls"
          hosts:
            - "emby.example.com"
  roles:
    - role: Anthony25.kubernetes-emby
```

Use `run_once` to run the role on only one available master in the cluster.

If the emby web interface is not reachable, please check that it is listening
on `0.0.0.0`, and not only `localhost`. Also check that the public http port is
setup as 8096.

License
-------

Tool under the BSD license. Do not hesitate to report bugs, ask me some
questions or do some pull request if you want to!
