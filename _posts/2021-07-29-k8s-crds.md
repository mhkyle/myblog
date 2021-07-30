# Kubernetes crds
![](/assets/img/k8s-logo.jpeg)
## 序
crd 即用于记录某个资源的资源。类似于一个类，你用它可以创建很多实例。你在api-server 那里注册了crd之后，api-server就知道有了这种资源的配置文件，然后你就可以通过yaml文件来创建这个“类”的实例了。

注意: 这些实例只是用来创建各个变量来记录某个资源，他没有任何动作。如果想要有动作的话就需要创建写controller来对这个资源的各种状态来进行监控，一旦有什么变化就做出什么样的动作等等。因此crd难的不是技术性，他难的是设计层面上的，怎么用它很好的设计一个模型。

## 创建crd
根据[crd官方documents](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)需要以下步骤：
### 1. 创建一个cluster
这个是必须的前提。有master节点和node节点，把关键组件特别是api-server全部都创建好。可以通过[简单的实验](https://labs.play-with-k8s.com/)来创建一个小型的cluster.

### 2. 创建一个crd yaml文件
在测试的时候我们可以随意的定义一个简单的crd,在实际生产中，这个一定要设计好blueprint之后才能着手做，因为一旦没有设计好想要从后面的再进行修改的话，那么会让人痛不欲生而且各种问题。

现在我们简单创建一个简易版的crd，我们可以类比kubernetes本身带的资源类型来创建。pod是一个或者若干个容器模拟出来的类似于openstack阶段的vm；node是pod载体的模拟；job是定时任务的模拟；那我们假如想对若干个pod的健康状况进行监控，我们可以创建一个NodeMonitor的资源类型。我们想要NodeMonitor的如下要求：

- 检查Node是否健康，如果pod对外暴露22端口，我们可以通过ping方法简单模拟
- Node是否调度pod
- Node容量还剩多少
- Node内存还有多少
- Node健康指数
- 。。。

我们可以随便定义任意我们想定义的，至于怎么来获取这些数据，就是controller要做的事情，我们稍后讨论。

然后我们就可以像写代码定义结构体一样定义crd的yaml.根据[官方给的spec](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema)的要求我们写下一下的spec:
```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: nodemonitors.compute.company.com
spec:
  # 我们把他归类为monitor类型，v1版本
  group: compute.company.com
  conversion:
    strategy: None
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                nodeName:
                  type: string
                pingDuration:
                  type: integer
                replicas:
                  type: integer
                image:
                  type: string
  names:
    plural: nodemonitors
    singular: nodemonitor
    kind: NodeMonitor
    shortNames:
    - nm
```
保存为文件crds-demo.yaml，然后
```
kubectl apply -f crds-demo.yaml
```
报错：
```
error: error validating "crds-demo.yaml": error validating data: ValidationError(CustomResourceDefinition.spec): missing required field "versions" in io.k8s.apiextensions-apiserver.pkg.apis.apiextensions.v1.CustomResourceDefinitionSpec; if you choose to ignore these errors, turn validation off with --validate=false
```
我们加上--validate=false就可以了
```
$ kubectl apply -f crds-demo.yaml --validate=false
customresourcedefinition.apiextensions.k8s.io/nodemonitors.compute.company.com created
```
现在这个crd已经创建成功了。
回顾我们创建的crd有三个字段：
- nodeName(string)
- pingDuration(integer)
- replicas(integer)
- image(string)

创建一个nodemonitor的实例object spec:
```
apiVersion: compute.company.com/v1
kind: NodeMonitor
metadata:
  name: myfirst-nodemonitor-object
spec:
  nodeName: "nodeName1"
  pingDuration: 5
  replicas: 1
  image : "Node Image"
```
保存为：nodemonitor.yaml
```
$ kubectl apply -f nodemonitor.yaml
nodemonitor.compute.company.com/myfirst-nodemonitor-object created

$ k get nodemonitor
NAME                         AGE
myfirst-nodemonitor-object   56s

$ k get nodemonitor myfirst-nodemonitor-object -oyaml
apiVersion: compute.company.com/v1
kind: NodeMonitor
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"compute.company.com/v1","kind":"NodeMonitor","metadata":{"annotations":{},"name":"myfirst-nodemonitor-object","namespace":"default"},"spec":{"image":"Node Image","nodeName":"nodeName1","pingDuration":5,"replicas":1}}
  creationTimestamp: "2021-07-29T08:11:37Z"
  generation: 1
  name: myfirst-nodemonitor-object
  namespace: default
  resourceVersion: "656187"
  selfLink: /apis/compute.company.com/v1/namespaces/default/nodemonitors/myfirst-nodemonitor-object
  uid: 865941dc-d3f4-4509-90a5-f4332c1a9194
spec:
  image: Node Image
  nodeName: nodeName1
  pingDuration: 5
  replicas: 1
```
ok, crd 已经对应的object已经定义完毕。接下来就是创建这个操作这个crd 对象的client.

现在基本的做法就是使用client-gen,让client-gen自动生成client已经informer，lister等等。我们在下一届介绍。

鉴于大家对文章内容有些地方英翻中的观点有一点不太一样，所以接下来我将全部使用英文来介绍关于k8s的内容，这样所有的固定名词都是原滋原味更容易理解。

