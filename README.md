# katafygio

[![Build Status](https://github.com/bpineau/katafygio/workflows/CI/badge.svg)](https://github.com/bpineau/katafygio/actions)
[![Coverage Status](https://coveralls.io/repos/github/bpineau/katafygio/badge.svg?branch=master)](https://coveralls.io/github/bpineau/katafygio?branch=master)
[![Go Report Card](https://goreportcard.com/badge/github.com/bpineau/katafygio)](https://goreportcard.com/report/github.com/bpineau/katafygio)

**katafygio** discovers Kubernetes objects (deployments, services, ...),
and continuously save them as yaml files in a git repository.
This provides real time, continuous backups, and keeps detailled changes history.

## Usage

To dump the cluster content once and exit:
```bash
katafygio --no-git --dump-only --local-dir /tmp/clusterdump/
```

To create a local git repository and continuously save the cluster content:
```bash
katafygio --local-dir /tmp/clusterdump/
```

To continuously push changes to a remote git repository:
```bash
katafygio --git-url https://user:token@github.com/myorg/myrepos.git --local-dir /tmp/clusterdump/
```

Filtering out irrelevant objects (esp. ReplicaSets and Pods) with `-w`, `-x`, `-y`
and `-z` is useful to keep a concise git history.

```bash
# Filtering out objects having an owner reference (eg. managed pods or replicasets,
# from Deployments, Daemonsets etc that we already archive), secrets (confidential),
# events and nodes (irrelevant), helm secrets/configmap releases, and a configmap
# named "leader-elector" that has low value and is causing commits churn:

katafygio \
  --local-dir /tmp/clusterdump/ \
  --git-url https://user:token@github.com/myorg/myrepos.git \
  --exclude-having-owner-ref \
  --exclude-kind secrets,events,nodes,endpoints \
  --exclude-object configmap:kube-system/leader-elector \
  --filter 'owner!=helm'
```

You can also use the [docker image](https://hub.docker.com/r/bpineau/katafygio/).

## CLI options

```
Backup Kubernetes cluster as yaml files in a git repository.
--exclude-kind (-x), --exclude-object (-y) and --exclude-namespaces (-z)
may be specified several times, or once with several comma separated values.

Usage:
  katafygio [flags]
  katafygio [command]

Available Commands:
  help        Help about any command
  version     Print the version number

Flags:
  -s, --api-server string            Kubernetes api-server url
  -c, --config string                Configuration file (default "/etc/katafygio/katafygio.yaml")
  -q, --context string               Kubernetes configuration context
  -d, --dry-run                      Dry-run mode: don't store anything
  -m, --dump-only                    Dump mode: dump everything once and exit
  -w, --exclude-having-owner-ref     Exclude all objects having an Owner Reference
  -x, --exclude-kind strings         Ressource kind to exclude. Eg. 'deployment'
  -z, --exclude-namespaces strings   Namespaces to exclude. Eg. 'temp.*' as regexes. This collects all namespaces and then filters them. Don't use it with the namespace flag.
  -y, --exclude-object strings       Object to exclude. Eg. 'configmap:kube-system/kube-dns'
  -l, --filter string                Label selector. Select only objects matching the label
  -t, --git-timeout duration         Git operations timeout (default 5m0s)
  -g, --git-url string               Git repository URL
  -p, --healthcheck-port int         Port for answering healthchecks on /health url
  -h, --help                         help for katafygio
  -k, --kube-config string           Kubernetes configuration path
  -e, --local-dir string             Where to dump yaml files (default "./kubernetes-backup")
  -v, --log-level string             Log level (default "info")
  -o, --log-output string            Log output (default "stderr")
  -r, --log-server string            Log server (if using syslog)
  -a, --namespace string             Only dump objects from this namespace
  -n, --no-git                       Don't version with git
  -i, --resync-interval int          Full resync interval in seconds (0 to disable) (default 900)
```

## Configuration file and env variables

All settings can be passed by command line options, or environment variable, or in
[a yaml configuration file](https://github.com/bpineau/katafygio/blob/master/assets/katafygio.yaml)
The environment are the same as command line options, in uppercase, prefixed by "KF_", and with underscore instead of dashs. ie.:

```
export KF_GIT_URL=https://user:token@github.com/myorg/myrepos.git
export KF_LOCAL_DIR=/tmp/clusterdump
export KF_LOG_LEVEL=info
export KF_EXCLUDE_KIND="pod ep rs clusterrole"

# non-prefixed KUBECONFIG works the same as for kubectl
export KUBECONFIG=/tmp/kconfig
```

## Installation

You can find pre-built binaries in the [releases](https://github.com/bpineau/katafygio/releases) page,
ready to run on your desktop or in a Kubernetes cluster.

We also provide a docker image on [docker hub](https://hub.docker.com/r/bpineau/katafygio/)
and on [quay.io](https://quay.io/bpineau/katafygio).

On MacOS, you can use the brew formula:
```bash
brew install bpineau/tap/katafygio
```

You can also deploy with the provided [helm](https://helm.sh/) chart and/or repository:
```shell
helm repo add katafygio https://bpineau.github.io/katafygio
helm repo update

helm install kube-backups katafygio/katafygio
```

## See Also

* [Heptio Velero](https://github.com/heptio/velero) does sophisticated clusters backups, including volumes
* [Stash](https://github.com/appscode/stash) backups volumes
* [etcd backup operator](https://github.com/coreos/etcd-operator) save etcd dumps (archived project)

