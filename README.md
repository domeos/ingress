## 说明
domeos/ingress模块是以k8s原生ingress controllers模块为基础，为适应DomeOS中nginx类型的负载均衡需求而设计修改的:

1、由于官方的Ingress controllers不支持自定义nginx的监听端口，为此我们引入环境变量LISTENPORT来配置nginx的监听端口，在启动负载均衡实例时，controller会检测该端口是否被占用，如果被占用，容器会启动失败，而且控制台日志中会有该端口被占用的提示。

2、由于官方的Ingress controllers不支持自定义nginx的负载均衡方式，为此我们引入环境变量LBMETHOD来定义nginx的负载均衡方式。

3、由于nginx类型的负载均衡实例是以Host的网络模式启动，而官方的Ingress controllers启动时，会占用443端口进行tcp的代理处理，默认占用10254端口作为nginx容器的健康检查端口，为此去掉了官方源码中的这部分逻辑，防止端口占用问题的出现。

4、当所关联的部署没有任何实例时，官方默认的nginx-controller会在upstream中添加127.0.0.1:8181作为默认server，此时负载均衡实例所在的node上可能有某个服务正好在监听8181端口，为此我们针对默认server都做了down处理。

5、精简nginx配置文件，去除了nginx.tmpl中有关HTTPS的配置。
## 注意
1、在修改nignx-controller源码之前，请先阅读完以上说明。

2、不要去除有关LISTENPORT和LBMETHOD等的有关处理逻辑。

3、nginx模板配置文件nginx.tmpl位于controllers/nginx/rootfs/etc/nginx/template。

4、尽量优先使用controllers/nginx下面的Makefile进行代码编译、镜像构建等。
## 编译构建镜像

```
git clone https://github.com/domeos/ingress.git
cd ingress/controllers/nginx
编译源码：make build
构建镜像：make container
上传镜像：make push
```
默认仓库地址为pub.domeos.org/domeos/nginx-controller:1.0，可以根据需要自行修改Makefile中的PREFIX和RELEASE
## 参考
Ingress官方仓库：https://github.com/kubernetes/ingress

Ingress资源说明：https://kubernetes.io/docs/concepts/services-networking/ingress

