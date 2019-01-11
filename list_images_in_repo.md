## Listing catalogues and images in a repo

To list the catalogues in a docker repo

```bash
$ curl -XGET http://docker.prod.aistemos.com:5000/v2/_catalog

with jq installed

$ curl -XGET http://docker.prod.aistemos.com:5000/v2/_catalog | jq
```

```bash
{
  "repositories": [
    "alerts-service",
    "apache-drill",
    "apache-zookeeper",
    "ccui",
    "choreographer-service",
    "comparables-service",
    "custom-classifiers-service",
    "custom-classifiers-tasks-service",
    "data-export-service",
    "domain-service",
    "frontend-service",
    "hyperscripts-service",
    "keycloak-service",
    "keycloak-standalone-snapshot",
    "ocypod",
    "portfolio-cluster-service",
    "portfolio-cluster-workers",
    "report-reader-service",
    "report-writer-service",
    "report-writer-workers",
    "text-search-service"
  ]
}
```

Once the catalogues have been returned the tags can be listed for a specific catalogue

```bash
$ curl -XGET http://docker.prod.aistemos.com:5000/v2/<catalogue>/tags/

$ curl -XGET http://docker.prod.aistemos.com:5000/v2/text-search-service/tags/list
```

```bash
{
  "name": "text-search-service",
  "tags": [
    "1.0.0",
    "1.0.1",
    "1.1.0",
    "1.1.1",
    "3",
    "beta1",
    "beta4"
  ]
}
```
