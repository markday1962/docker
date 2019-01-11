## Listing catalogues and images in a repo

To list the catalogues in a docker repo

```bash
$ curl -XGET http://docker.prod.aistemos.com:5000/v2/_catalog
```

```bash
curl -XGET http://docker.prod.aistemos.com:5000/v2/<catalogue>/tags/list
```
