# docker-image-syncer

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们在国内部署kubernetes时,第一步安装k8难倒了各大英雄好汉。原因是k8s 各种组件镜像在谷歌服务器上(k8s.gcr.io)，<br />而我们有墙的存在，所以会经常性的下载失败。解决办法是搭梯子，或者是使用其它国内镜像仓库。当然我们也可以自己制作自己的镜像仓库，下面看操作步骤：

# 同步原理
&nbsp;本仓库使用阿里开源的镜像同步工具（[aliyun image-syncer](https://github.com/AliyunContainerService/image-syncer)），<br />同时结合GitHub的Action功能，来同步k8s组件镜像(k8s.gcr.io)  到 DockerHub上自己的账号中去或自己的搭建的私服harbor中去，这样后期安装k8s就可以将镜像源仓库，改为自己的DockerHub地址了或自己的私服地址。
    
    
# docker-image-syncer 运行原理

1. `docker pull` 下拉所需镜像

     由于GitHub Action 运行在国外的github服务器上，不会被墙，就可以轻松pull到google的镜像。

2. `docker tag` 修改镜像tag

3. `docker push` 推送镜像到相应docker register


# 具体操作步骤

1、fork 这个仓库, 创建你自己的docker register 账号密码:


操作路径： Settings --》Secrets --》New Repository Secrets --》添加两个变量名称为<br />（名称必须为下面两个，因为在workflow的yaml文件引用了这两个变量）：<br />
DOCKER_USERNAME<br />
DOCKER_PASSWORD<br />

![image-20221213102110118](https://img-blog.csdnimg.cn/img_convert/de478aaf77041569c82f17cd34834926.png#pic_center)

2、修改`images.json` 文件，修改为你dockHub地址，例如这里的shihaiyan是我自己DockerHub的账户，修改为你自己的就可以了

```
{
  "quay.io/coreos/kube-rbac-proxy": "shihaiyan/kube-rbac-proxy",
  "k8s.gcr.io/metrics-server/metrics-server": "shihaiyan/metrics-server",
  "k8s.gcr.io/ingress-nginx/controller": "shihaiyan/ingress-nginx-controller",
  "k8s.gcr.io/git-sync/git-sync": "shihaiyan/git-sync",
  "gcr.io/kaniko-project/executor:debug,latest": "shihaiyan/kaniko-executor",
  "k8s.gcr.io/kube-state-metrics/kube-state-metrics": "shihaiyan/kube-state-metrics",
  "k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner": "shihaiyan/nfs-subdir-external-provisioner",
  "k8s.gcr.io/prometheus-adapter/prometheus-adapter": "shihaiyan/prometheus-adapter",
  "k8s.gcr.io/kube-apiserver": "shihaiyan/kube-apiserver",
  "k8s.gcr.io/kube-controller-manager": "shihaiyan/kube-controller-manager",
  "k8s.gcr.io/kube-scheduler": "shihaiyan/kube-scheduler",
  "k8s.gcr.io/kube-proxy": "shihaiyan/kube-proxy",
  "k8s.gcr.io/pause": "shihaiyan/pause",
  "k8s.gcr.io/etcd": "shihaiyan/etcd",
  "k8s.gcr.io/coredns/coredns": "shihaiyan/coredns"
}
```

3、通过`auth.json`可以添加其它仓库认证信息<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里变量${USERNAME}和${PASSWORD}为DockerHub账号密码，其值在workflows中被赋值:<br />
（当然也可以直接写明文了，虽然我们不曾这么干！）
   USERNAME: ${{ secrets.DOCKER_USERNAME }}<br />
   PASSWORD: ${{ secrets.DOCKER_PASSWORD }}<br />
   其实就是我们在第一步setting中添加的两个变量！<br />
    这个文件中主要提供镜像仓库的登录验证信息，所以依次可以添加多个镜像仓库！

```yaml
{
  "registry.hub.docker.com": {
    "username": "${USERNAME}",
    "password": "${PASSWORD}"
  }
}
```



4、检查 action logs

![](https://img-blog.csdnimg.cn/img_convert/b72068c934fbc4394675e66e0b28e8c8.png#pic_center)

# Action 执行完后,检查成果

[dockerhub](https://hub.docker.com/repositories/admin4j)

![image-20221213102633683](https://img-blog.csdnimg.cn/img_convert/7d01e06938461c646c1354b2bdc3f383.png#pic_center)

5、检测同步结果
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;去自己的DockerHub上看看是否有已经同步的镜像

6、补充GitHub的workflow使用
参考：
[workflows使用](https://zhuanlan.zhihu.com/p/377731593)
