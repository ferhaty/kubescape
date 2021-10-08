<img src="docs/kubescape.png" width="300" alt="logo" align="center">

[![build](https://github.com/armosec/kubescape/actions/workflows/build.yaml/badge.svg)](https://github.com/armosec/kubescape/actions/workflows/build.yaml)
[![Go Report Card](https://goreportcard.com/badge/github.com/armosec/kubescape)](https://goreportcard.com/report/github.com/armosec/kubescape)

Kubescape is the first tool for testing if Kubernetes is deployed securely as defined in [Kubernetes Hardening Guidance by NSA and CISA](https://www.nsa.gov/Press-Room/News-Highlights/Article/Article/2716980/nsa-cisa-release-kubernetes-hardening-guidance/)

Use Kubescape to test clusters or scan single YAML files and integrate it to your processes.

<img src="docs/demo.gif">

# TL;DR
## Install:
```
curl -s https://raw.githubusercontent.com/armosec/kubescape/master/install.sh | /bin/bash
```

[Install on windows](#install-on-windows)

[Install on macOS](#install-on-macos)

## Run:
```
kubescape scan framework nsa --exclude-namespaces kube-system,kube-public
```

If you wish to scan all namespaces in your cluster, remove the `--exclude-namespaces` flag.

<img src="docs/summary.png">

### Click [👍](https://github.com/armosec/kubescape/stargazers) if you want us to continue to develop and improve Kubescape 😀

# Being part of the team

We invite you to our team! We are excited about this project and want to return the love we get.

Want to contribute? Want to discuss something? Have an issue?

* Open a issue, we are trying to respond within 48 hours
* [Join us](https://armosec.github.io/kubescape/) in a discussion on our discord server!

[<img src="docs/discord-banner.png" width="100" alt="logo" align="center">](https://armosec.github.io/kubescape/)

# Options and examples

## Install on Windows

**Requires powershell v5.0+**

``` powershell
iwr -useb https://raw.githubusercontent.com/armosec/kubescape/master/install.ps1 | iex
```

Note: if you get an error you might need to change the execution policy (i.e. enable Powershell) with

``` powershell
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
```

## Install on macOS
```
brew tap armosec/kubescape
brew install kubescape
```

## Flags

| flag |  default | description | options |
| --- | --- | --- | --- |
| `-e`/`--exclude-namespaces` | Scan all namespaces | Namespaces to exclude from scanning. Recommended to exclude `kube-system` and `kube-public` namespaces |
| `-s`/`--silent` | Display progress messages | Silent progress messages |
| `-t`/`--fail-threshold` | `0` (do not fail) | fail command (return exit code 1) if result bellow threshold| `0` -> `100` |
| `-f`/`--format` | `pretty-printer` | Output format | `pretty-printer`/`json`/`junit` |
| `-o`/`--output` | print to stdout | Save scan result in file |
| `--use-from` | | Load local framework object from specified path. If not used will download latest |
| `--use-default` | `false` | Load local framework object from default path. If not used will download latest | `true`/`false` |
| `--exceptions` | | Path to an [exceptions obj](examples/exceptions.json). If not set will download exceptions from Armo management portal |
| `--results-locally` | `false` | Kubescape sends scan results to Armo management portal to allow users to control exceptions and maintain chronological scan results. Use this flag if you do not wish to use these features | `true`/`false`|

## Usage & Examples

### Examples

* Scan a running Kubernetes cluster with [`nsa`](https://www.nsa.gov/News-Features/Feature-Stories/Article-View/Article/2716980/nsa-cisa-release-kubernetes-hardening-guidance/) framework
```
kubescape scan framework nsa --exclude-namespaces kube-system,kube-public
```


* Scan a running Kubernetes cluster with [`mitre`](https://www.microsoft.com/security/blog/2020/04/02/attack-matrix-kubernetes/) framework
```
kubescape scan framework mitre --exclude-namespaces kube-system,kube-public
```

* Scan local `yaml`/`json` files before deploying
```
kubescape scan framework nsa *.yaml
```


* Scan `yaml`/`json` files from url
```
kubescape scan framework nsa https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/kubernetes-manifests.yaml
```

* Output in `json` format
```
kubescape scan framework nsa --exclude-namespaces kube-system,kube-public --format json --output results.json
```

* Output in `junit xml` format
```
kubescape scan framework nsa --exclude-namespaces kube-system,kube-public --format junit --output results.xml
```

* Scan with exceptions, objects with exceptions will be presented as `warning` and not `fail`  <img src="docs/new-feature.svg">
```
kubescape scan framework nsa --exceptions examples/exceptions.json
```

### Helm Support

* Render the helm chart using [`helm template`](https://helm.sh/docs/helm/helm_template/) and pass to stdout
```
helm template [NAME] [CHART] [flags] --dry-run | kubescape scan framework nsa -
```

for example:
```
helm template bitnami/mysql --generate-name --dry-run | kubescape scan framework nsa -
```
### Offline Support

It is possible to run Kubescape offline!

First download the framework and then scan with `--use-from` flag

* Download and save in file, if file name not specified, will store save to `~/.kubescape/<framework name>.json`
```
kubescape download framework nsa --output nsa.json
```

* Scan using the downloaded framework
```
kubescape scan framework nsa --use-from nsa.json
```

Kubescape is an open source project, we welcome your feedback and ideas for improvement. We’re also aiming to collaborate with the Kubernetes community to help make the tests themselves more robust and complete as Kubernetes develops.

# How to build

## Build using python script

Kubescpae can be built using:

``` sh
python build.py
```

Note: In order to built using the above script, one must set the environment
variables in this script:

+ RELEASE
+ ArmoBEServer
+ ArmoERServer
+ ArmoWebsite


## Build using go

Note: development (and the release process) is done with Go `1.17`

1. Clone Project
```
git clone https://github.com/armosec/kubescape.git kubescape && cd "$_"
```

2. Build
```
go build -o kubescape .
```

3. Run
```
./kubescape scan framework nsa --exclude-namespaces kube-system,kube-public
```

4. Enjoy :zany_face:

## How to build in Docker

1. Clone Project
```
git clone https://github.com/armosec/kubescape.git kubescape && cd "$_"
```

2. Build
```
docker build -t kubescape -f build/Dockerfile .
```

# Under the hood

## Tests
Kubescape is running the following tests according to what is defined by [Kubernetes Hardening Guidance by NSA and CISA](https://www.nsa.gov/News-Features/Feature-Stories/Article-View/Article/2716980/nsa-cisa-release-kubernetes-hardening-guidance/)
* Non-root containers
* Immutable container filesystem
* Privileged containers
* hostPID, hostIPC privileges
* hostNetwork access
* allowedHostPaths field
* Protecting pod service account tokens
* Resource policies
* Control plane hardening
* Exposed dashboard
* Allow privilege escalation
* Applications credentials in configuration files
* Cluster-admin binding
* Exec into container
* Dangerous capabilities
* Insecure capabilities
* Linux hardening
* Ingress and Egress blocked
* Container hostPort
* Network policies
* Symlink Exchange Can Allow Host Filesystem Access (CVE-2021-25741)



## Technology
Kubescape based on OPA engine: https://github.com/open-policy-agent/opa and ARMO's posture controls.

The tools retrieves Kubernetes objects from the API server and runs a set of [regos snippets](https://www.openpolicyagent.org/docs/latest/policy-language/) developed by [ARMO](https://www.armosec.io/).

The results by default printed in a pretty "console friendly" manner, but they can be retrieved in JSON format for further processing.

Kubescape is an open source project, we welcome your feedback and ideas for improvement. We’re also aiming to collaborate with the Kubernetes community to help make the tests themselves more robust and complete as Kubernetes develops.
