# Helm3学习笔记
## 1.安装

```shell
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

```shell
root@ubuntu20:~# helm version
version.BuildInfo{Version:"v3.9.0", GitCommit:"7ceeda6c585217a19a1131663d8cd1f7d641b2a7", GitTreeState:"clean", GoVersion:"go1.17.5"}
```



## 2.helm的三个概念

### 1.Chart

*Chart* 代表着 Helm 包。k8s的**包管理工具**，它包含在 k8s 内部运行应用程序、工具或服务所需的所有**资源定义**。就是my_chart目录下的那些文件。

相当于ubuntu的apt包管理工具。

### 2.Repository

*Repository（仓库）* 是用来存放和共享 charts 的地方。会有很多仓库源，需要手动添加。

相当于uubuntu的apt仓库。

### 3.Release

*Release* 是运行在 Kubernetes 集群中的 chart 的实例。一个 chart 通常可以在同一个集群中安装多次。每一次安装都会创建一个新的 *release*。以 MySQL chart为例，如果你想在你的集群中运行两个数据库，你可以安装该chart两次。每一个数据库都会拥有它自己的 *release* 和 *release name*。



### 总结：

Helm 安装 *charts* 到 Kubernetes 集群中，每次安装都会创建一个新的 *release*。你可以在 Helm 的 chart 仓库 中寻找新的 chart。



## 3.使用

### 1.查找hub

例如查找所有包含nginx的chart的hub

```shell
root@ubuntu20:~# helm search hub nginx
URL                                               	CHART VERSION	APP VERSION                            	DESCRIPTION                                       
https://artifacthub.io/packages/helm/cloudnativ...	3.2.0        	1.16.0                                 	Chart for the nginx server                        
https://artifacthub.io/packages/helm/ashu-nginx...	0.1.0        	1.16.0                                 	A Helm chart for Kubernetes                       
https://artifacthub.io/packages/helm/jfrog/nginx  	15.1.5       	1.25.2                                 	NGINX Open Source is a web server that can be a...
https://artifacthub.io/packages/helm/dysnix/nginx 	7.1.8        	1.19.4                                 	Chart for the nginx server                        
https://artifacthub.io/packages/helm/bitnami-ak...	13.2.12      	1.23.2                                 	NGINX Open Source is a web server that can be a...
```

上述命令从 Artifact Hub 中搜索所有的 `nginx` charts。

artifacthub 是一个开源的社区仓库，Helm 3 中整合的官方软件包索引。它的网址是 https://artifacthub.io/ 。



### 2.添加repo

查看本地已经添加的repo

```shell
root@ubuntu20:~# helm repo list
NAME           	URL                                                 
koderover-chart	https://koderover.tencentcloudcr.com/chartrepo/chart
bitnami        	https://charts.bitnami.com/bitnami
```



```shell
#添加仓库，如果已存在就不需要添加了
root@ubuntu20:~# helm repo add bitnami https://charts.bitnami.com/bitnami 
"bitnami" already exists with the same configuration, skipping
#更新仓库
root@ubuntu20:~# helm repo update	
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "koderover-chart" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈

```



### 3.下载chart到本地，并解压

```shell
root@ubuntu20:~# helm fetch bitnami/nginx --untar  #并解压
root@ubuntu20:~# cd nginx/
root@ubuntu20:~/nginx# ls
Chart.lock  charts  Chart.yaml  README.md  templates  values.schema.json  values.yaml
```



### 3.安装

```shell
helm install my-nginx . --namespace=default
```

```shell
root@ubuntu20:~/nginx# helm install my-nginx . --namespace=default
NAME: my-nginx
LAST DEPLOYED: Mon Mar 25 10:54:35 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 15.14.0
APP VERSION: 1.25.4

** Please be patient while the chart is being deployed **
NGINX can be accessed through the following DNS name from within your cluster:

    my-nginx.default.svc.cluster.local (port 80)

To access NGINX from outside the cluster, follow the steps below:

1. Get the NGINX URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w my-nginx'

    export SERVICE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].port}" services my-nginx)
    export SERVICE_IP=$(kubectl get svc --namespace default my-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo "http://${SERVICE_IP}:${SERVICE_PORT}"

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - cloneStaticSiteFromGit.gitSync.resources
  - resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

```

`WARNING` 有警告，而且服务也没起来，说需要进行资源设置。修改 `values.yaml`

### 4.修改value.yaml

`resourcesPreset` 部分

```yaml
resourcesPreset: "none"
## @param resources Set container requests and limits for different resources like CPU or memory (essential for production workloads)
## Example:
## resources:
##   requests:
##     cpu: 2
##     memory: 512Mi
##   limits:
##     cpu: 3
##     memory: 1024Mi
##
resources:
  requests:
    cpu: 2
    memory: 512Mi
  limits:
    cpu: 3
    memory: 1024Mi
```



`cloneStaticSiteFromGit` 部分

```yaml
    resourcesPreset: "none"
    ## @param cloneStaticSiteFromGit.gitSync.resources Set container requests and limits for different resources like CPU or memory (essential for production workloads)
    ## Example:
    ## resources:
    ##   requests:
    ##     cpu: 2
    ##     memory: 512Mi
    ##   limits:
    ##     cpu: 3
    ##     memory: 1024Mi
    ##
    resources:
      requests:
        cpu: 2
        memory: 512Mi
      limits:
        cpu: 3
        memory: 1024Mi
```



`service`部分

```yaml
service:
  ## @param service.type Service type
  ##
  type: NodePort #LoadBalancer改为NodePort
  ## @param service.ports.http Service HTTP port
  ## @param service.ports.https Service HTTPS port
  ##
  ports:
    http: 80
    https: 443
  ##
  ## @param service.nodePorts [object] Specify the nodePort(s) value(s) for the LoadBalancer and NodePort service types.
  ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
  ##
  nodePorts:
    http: 30080
    https: 30443
```





### 5.更新

```shell
root@ubuntu20:~/nginx# helm upgrade my-nginx ./ -f values.yaml 
Release "my-nginx" has been upgraded. Happy Helming!
NAME: my-nginx
LAST DEPLOYED: Mon Mar 25 11:31:03 2024
NAMESPACE: default
STATUS: deployed
REVISION: 4
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 15.14.0
APP VERSION: 1.25.4

** Please be patient while the chart is being deployed **
NGINX can be accessed through the following DNS name from within your cluster:

    my-nginx.default.svc.cluster.local (port 80)

To access NGINX from outside the cluster, follow the steps below:

1. Get the NGINX URL by running these commands:

    export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services my-nginx)
    export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
    echo "http://${NODE_IP}:${NODE_PORT}"

```



```shell
root@ubuntu20:~/nginx# kubectl get svc my-nginx -n default
NAME       TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
my-nginx   NodePort   10.233.36.35   <none>        80:30080/TCP   36m

```

这样就可以通过`节点:端口`访问ngix了。在浏览器打开 http://192.168.71.49:30080/



### 7.练习安装kafka

查询

```shell
root@ubuntu20:~# helm search repo kafka
NAME                    	CHART VERSION	APP VERSION	DESCRIPTION                                       
bitnami/kafka           	27.1.2       	3.7.0      	Apache Kafka is a distributed streaming platfor...
bitnami/dataplatform-bp2	12.0.5       	1.0.1      	DEPRECATED This Helm chart can be used for the ...
bitnami/schema-registry 	17.1.1       	7.6.0      	Confluent Schema Registry provides a RESTful in...

```

下载

```
root@ubuntu20:~# helm fetch bitnami/kafka --untar
root@ubuntu20:~# cd kafka
root@ubuntu20:~/kafka# ls
Chart.lock  charts  Chart.yaml  README.md  templates  values.yaml
```

修改配置文件`values.yaml`

修改`broker.resources`

```yaml
  resources:
    requests:
      cpu: 2
      memory: 512Mi
    limits:
      cpu: 3
      memory: 1024Mi
```

`externalAccess.controller.service.type`

```yaml
service:
  ## @param service.type Kubernetes Service type
  ##
  type: ClusterIP
  ## @param service.ports.client Kafka svc port for client connections
  ## @param service.ports.controller Kafka svc port for controller connections. It is used if "kraft.enabled: true"
  ## @param service.ports.interbroker Kafka svc port for inter-broker connections
  ## @param service.ports.external Kafka svc port for external connections
  ##
  ports:
    client: 9092
    controller: 9093
    interbroker: 9094
    external: 9095
```



```
helm install my-kafka . --namespace=default
helm install my-kafka  --namespace=default .
```

```
root@ubuntu20:~/kafka# helm install my-kafka . --namespace=default
NAME: my-kafka
LAST DEPLOYED: Mon Mar 25 12:00:15 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: kafka
CHART VERSION: 27.1.2
APP VERSION: 3.7.0

** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    my-kafka.default.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    my-kafka-controller-0.my-kafka-controller-headless.default.svc.cluster.local:9092
    my-kafka-controller-1.my-kafka-controller-headless.default.svc.cluster.local:9092
    my-kafka-controller-2.my-kafka-controller-headless.default.svc.cluster.local:9092

The CLIENT listener for Kafka client connections from within your cluster have been configured with the following security settings:
    - SASL authentication

To connect a client to your Kafka, you need to create the 'client.properties' configuration files with the content below:

security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
    username="user1" \
    password="$(kubectl get secret my-kafka-user-passwords --namespace default -o jsonpath='{.data.client-passwords}' | base64 -d | cut -d , -f 1)";

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run my-kafka-client --restart='Never' --image docker.io/bitnami/kafka:3.7.0-debian-12-r0 --namespace default --command -- sleep infinity
    kubectl cp --namespace default /path/to/client.properties my-kafka-client:/tmp/client.properties
    kubectl exec --tty -i my-kafka-client --namespace default -- bash

    PRODUCER:
        kafka-console-producer.sh \
            --producer.config /tmp/client.properties \
            --broker-list my-kafka-controller-0.my-kafka-controller-headless.default.svc.cluster.local:9092,my-kafka-controller-1.my-kafka-controller-headless.default.svc.cluster.local:9092,my-kafka-controller-2.my-kafka-controller-headless.default.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \
            --consumer.config /tmp/client.properties \
            --bootstrap-server my-kafka.default.svc.cluster.local:9092 \
            --topic test \
            --from-beginning

```



### 6.创建chart

```shell
root@ubuntu20:~# helm create my_chart
Creating my_chart
```

查看chart包含的文件

```shell
root@ubuntu20:~# tree my_chart/
my_chart/
├── charts	#这个目录用来存放当前chart所依赖的其他chart。每个子目录就是一个依赖项，里面同样包含了完整的chart结构。
├── Chart.yaml	#这是chart的元数据文件，其中包含了chart的基本信息，如名称、版本、描述、依赖关系等。这些信息对于理解和管理chart至关重要。
├── templates	#这个目录包含了所有Kubernetes资源清单的模板文件，比如Deployment、Service、ConfigMap等。Helm使用Go模板语言编写这些文件，并根据values.yaml中的参数动态生成实际的资源配置。
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests #这个目录下存放的是chart的测试文件，通常采用 YAML 格式编写。这些测试会在Helm安装或升级chart之后执行，用于验证chart部署后的Kubernetes资源状态是否符合预期。
│       └── test-connection.yaml
└── values.yaml #这个文件包含了chart模板中使用的默认配置值。使用者可以通过覆盖这些默认值来定制部署的应用配置。例如，可以定义Pod的数量、镜像版本、环境变量等。

3 directories, 10 files
```



### 2.下载chart

添加仓库

```shell

```



```shell
root@ubuntu20:~# cd kafka/
root@ubuntu20:~/kafka# ls
Chart.lock  charts  Chart.yaml  README.md  templates  values.yaml
```


