# rabbitmq-kubernetes-ha

A template to run RabbitMQ on Kubernetes in High Availability supporting TLS and MQTT and automatic peer discovery.

Kubernetes 1.9 or later is required.

## TLS

Both the management interface run on https and TLS is enabled on both MQTT and AMQP.

To accomplish this we are using Let's Encrypt via https://github.com/PalmStoneGames/kube-cert-manager which creates
a secret called `cert-mq-c1` which is mounted by the statefulset.

You will want to setup kube-cert-manager as well in order to use this template.

## Deploy

* Create your configmap. Make sure you review the values in configmap.yaml and change as appropriate.

``` bash
kubectl create -f configmap.yaml
```

* Assuming RBAC is enabled on your cluster which is the default, you need to configure some RBAC stuff for auto peer discovery to work

``` bash
kubectl create -f rbac.yaml
```

* Create a service to allow the rabbitmq PODs to contact each other

``` bash
kubectl create -f services.yaml
```

* Finally create the statefulset for RabbitMQ itself

``` bash
kubectl create -f statefulset.yaml
```

Now wait a few seconds and you should be able to run

``` bash
kubectl exec rabbitmq-0 rabbitmqctl cluster_status
```

And get back something like this:

``` json
Cluster status of node rabbit@rabbitmq-0.rabbitmq.default.svc.cluster.local ...
[{nodes,[{disc,['rabbit@rabbitmq-0.rabbitmq.default.svc.cluster.local',
                'rabbit@rabbitmq-1.rabbitmq.default.svc.cluster.local']}]},
 {running_nodes,['rabbit@rabbitmq-1.rabbitmq.default.svc.cluster.local',
                 'rabbit@rabbitmq-0.rabbitmq.default.svc.cluster.local']},
 {cluster_name,<<"rabbit@rabbitmq.default.svc.cluster.local">>},
 {partitions,[]},
 {alarms,[{'rabbit@rabbitmq-1.rabbitmq.default.svc.cluster.local',[]},
          {'rabbit@rabbitmq-0.rabbitmq.default.svc.cluster.local',[]}]}]
```

* Scaling:

``` bash
kubectl scale statefulset/rabbitmq --replicas=3
```