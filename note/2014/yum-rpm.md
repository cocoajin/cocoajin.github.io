#YUM&RPM

##yum

- `yum search + 关键字` 搜索相关的安装包，如果联网会在软件源中联网搜索；
- `yum info + 关键字` 查看指定关键字安装包信息，显示软件是否已经安装，及包信息；
- `yum install +关键字` 安装指定软件包，会从附近的镜像下载资源安装；指定 -y 就会忽略安装提醒进行安装, yum -y install php；
- `yum update + 关键字` 更新指定软件包；
- `yum erase +关键字` 从系统中移除一个软件包；
- `yum deplist +关键字` 列出指定软件包的依赖关系；
- `yum list +关键字` 列出一个软件的信息;
- `yum reinstall +关键字`重新安装软件包；
- `yum repolist` 查看安装源信息；
- `yum remove +关键字` 移除指定软件包；
- 更多信息 `man yum`


## rpm

- `rpm -q +关键字` 查询相关软件包的安装情况；
- `rpm -V+关键字`验证指定的软件包；
- `rpm -i +关键字` 安装指定软件包；
- `rpm -U+关键字`更新指定软件包；
- `rpm -e+关键字` 删除指定软件包；
- 更多信息 `man rpm`