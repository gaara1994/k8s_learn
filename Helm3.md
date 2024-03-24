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
root@master:~# helm version
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

Helm 安装 *charts* 到 Kubernetes 集群中，每次安装都会创建一个新的 *release*。你可以在 Helm 的 chart *repositories* 中寻找新的 chart。



## 3.

创建一个chart

```shell
root@master:~# helm create my_chart
Creating my_chart
```

查看chart包含的文件

```shell
root@master:~# tree my_chart/
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











