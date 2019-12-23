## Helm
### A package manager for Kubernetes

**Chart** - Packaged Kubernetes resource.

A chart contains two models:
* Template
* Values

**Chart repository** - Dockerhub but for `helm` charts.
**Release** - Deployed instance of a chart. Even if the source code wasn't modified, the release

#### Helm Components

Client- `helm client`  - Command line templating engine.

Tiller - Lives inside the cluster, manages releases, history and introspection.

To create a chart:
`helm create myChart`