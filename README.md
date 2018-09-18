# chart-logging-central

[![Build Status](https://jenkins.migrations.cnct.io/buildStatus/icon?job=pipeline-central-logging/master)](https://jenkins.migrations.cnct.io/job/pipeline-central-logging/job/master)

`chart-logging-central` (this chart) and [`chart-logging-client`](https://github.com/samsung-cnct/chart-logging-client) implement our
central logging solution for Kubernetes clusters. These replace the deprecated `chart-logging`.

The logging-client will collect Kubernetes logs and events with fluentbit, enrich them with Kubernetes metadata and send them to this system. There they will be saved in an elasticsearch data store and made available for visualization and querying with kibana.  Curator will delete old indices after two weeks. 


## AWS Setup for Route53
This uses external-dns in a very constrained setup is used to create/maintain the ELB to FQDN connection.

Create a logging-dns AWS user if one is not present, with only programatic access (no console).

Create the following route53 only IAM as described here:[ https://github.com/kubernetes-incubator/external-dns/blob/master/docs/tutorials/aws.md#iam-permissions]() and assign it to the user as the only policy for that user.

```{
 "Version": "2012-10-17",
 "Statement": [
   {
     "Effect": "Allow",
     "Action": [
       "route53:ChangeResourceRecordSets"
     ],
     "Resource": [
       "arn:aws:route53:::hostedzone/*"
     ]
   },
   {
     "Effect": "Allow",
     "Action": [
       "route53:ListHostedZones",
       "route53:ListResourceRecordSets"
     ],
     "Resource": [
       "*"
     ]
   }
 ]
}
```

Note the Access and Secret Keys for this user, as they are needed by the program to manipulate route53.  They will be passed in to the chart installation.

Create your domain entry in route53.  e.g. `newdomain.cluster.cnct.io`.  
Add an NS record to the lower domain for your new domain to allow naming resolution.  e.g. in `cluster.cnct.io` add a new NS record and copy the values from the  `newdomain.cluster.cnct.io` NS record to that.
Modify `values.yaml` or set the values on the helm install:

```
--set external-dns.domainFilters[0]="newdomain.cluster.cnct.io"
--set elasticsearch-chart.services.es.annotations[0].key="external-dns.alpha.kubernetes.io/hostname"
--set elasticsearch-chart.services.es.annotations[0].value=es.newdoamin.cluster.cnct.io
```

## How to install on running Kubernetes cluster with [helm](https://github.com/kubernetes/helm/blob/master/docs/install.md)
Ensure that you have helm and [tiller](https://docs.helm.sh/using_helm/) installed. 
Make sure your cluster has at least 6 worker nodes
### From our chart repository
``` 
helm repo add cnct https://charts.migrations.cnct.io
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo update
helm install cnct/logging-central --set external-dns.aws.secretKey="$AWS_SECRET_ACCESS_KEY",external-dns.aws.accessKey="$AWS_ACCESS_KEY_ID"
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

## Contributing

1. Fork it! 
2. Clone your fork locally: `git clone <your fork>`
3. Go into that new local copy: `cd chart-logging-central`
3. Set the upstream remote to be the source repo: `git remote add upstream git@github.com:samsung-cnct/chart-logging-central.git`
4. Go into the chart subdirectory: `cd charts/logging-central/`
5. Make sure your local helm repos are up to date: `helm repo add cnct https://charts.migrations.cnct.io; 
helm repo add stable https://kubernetes-charts.storage.googleapis.com/; 
helm repo update`
5. Pull in all the dependent helm charts: `helm dependency update`
2. Create your feature branch: `git checkout -b my-new-feature`
3. Do your changes, then deploy: `helm install . --name=<myname>-central-test --namespace=logging --debug`
4. Test: `helm test <myname>-central-test --debug --cleanup`
5. Remove deployment when not needed: `helm delete <myname>-central-test --purge`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request :D
