# k8s_ui

k8s搭建kuboard 可视化视图

下载需要的镜像
docker pull eipwork/kuboard:latest

在主节点写入如下的yaml文件---------------------------------------注意在第二十六行写你要在那个节点上面创建kuborad

运行yaml文件
  
‘kubectl apply -f kuboard.yaml’

浏览器访问http://ip:32567/

获取token

‘echo $(kubectl -n kube-system get secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}') -o go-template='{{.data.token}}' | base64 -d)‘
