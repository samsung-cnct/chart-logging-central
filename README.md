# chart-logging-central

[![Build Status](https://jenkins.migrations.cnct.io/buildStatus/icon?job=pipeline-central-logging/master)](https://jenkins.migrations.cnct.io/job/pipeline-central-logging/job/master)

`chart-logging-central` (this chart) and [`chart-logging-client`](https://github.com/samsung-cnct/chart-logging-client) implement our
central logging solution for Kubernetes clusters. These replace the deprecated `chart-logging`.

The logging-client will collect Kubernetes logs and events with fluentbit, enrich them with Kubernetes metadata and send them to this system. There they will be saved in an elasticsearch data store and made available for visualization and querying with kibana.  Curator will delete old indices after two weeks. 

## How to install on running Kubernetes cluster with [helm](https://github.com/kubernetes/helm/blob/master/docs/install.md)
Ensure that you have helm and [tiller](https://docs.helm.sh/using_helm/) installed. 
Make sure your cluster has at least 6 worker nodes
### From our chart repository
``` 
helm repo add cnct https://charts.migrations.cnct.io
helm repo update
helm install cnct/logging-central
```

This system is currently using [elasticsearch xpack](https://www.elastic.co/guide/en/x-pack/current/elasticsearch-security.html) for security. To view your logs, run ``` kubectl get svc kibana-logging -o wide ``` and navigate to `<EXTERNAL-IP>:5601`. You will need to log in using the default username `elastic` and password of `changeme`.

## Assets 
#### Queryable Datastore: [Elasticsearch](https://www.elastic.co/products/elasticsearch)
- [Docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Chart](https://github.com/samsung-cnct/chart-elasticsearch)
- [Xpack Security Docs](https://www.elastic.co/guide/en/x-pack/current/elasticsearch-security.html)

#### Index Manager for Elasticsearch: Curator 
- [Docs](https://www.elastic.co/guide/en/elasticsearch/client/curator/5.5/index.html)
- [Chart](https://github.com/samsung-cnct/chart-curator)

#### Data visualation: [Kibana](https://www.elastic.co/products/kibana)
- [Docs](https://www.elastic.co/guide/en/kibana/6.4/introduction.html)
- [Chart](https://github.com/samsung-cnct/chart-kibana)

### Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request :D