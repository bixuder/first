<h1><center>持续集成-交付-发布-VUE</center></h1>

------

作者：行癫（盗版必究）

## 一：Jenkins服务器部署Node.js

#### 1.下载Node.js安装包

```shell
https://nodejs.org/dist/v18.15.0/node-v18.15.0-linux-x64.tar.gz
```

#### 2.安装

```shell
[root@jenkins ~]# tar xf node-v14.19.3-linux-x64.tar.gz  -C /usr/local/
[root@jenkins ~]# mv node-v14.19.3-linux-x64/ node
[root@jenkins ~]# yum -y install gcc-c++ make cmake
```

#### 3.设置环境变量

```shell
[root@jenkins ~]# cat /etc/profile
JAVA_HOME=/usr/local/java
MAVEN_HOME=/usr/local/maven
NODE_HOME=/usr/local/node
PATH=$JAVA_HOME/bin:$PATH
PATH=$MAVEN_HOME/bin:$PATH
PATH=$NODE_HOME/bin:$PATH
export JAVA_HOME MAVEN_HOME  NODE_HOME PATH
```

#### 4.验证

```shell
[root@jenkins local]# npm version
{
  npm: '6.14.17',
  ares: '1.18.1',
  brotli: '1.0.9',
  cldr: '40.0',
  icu: '70.1',
  llhttp: '2.1.4',
  modules: '83',
  napi: '8',
  nghttp2: '1.42.0',
  node: '14.19.3',
  openssl: '1.1.1o',
  tz: '2021a3',
  unicode: '14.0',
  uv: '1.42.0',
  v8: '8.4.371.23-node.87',
  zlib: '1.2.11'
}
```

## 二：Jenkins配置Node.js

#### 1.安装node插件

![image-20230321200315169](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230321200315169.png)

注意：安装完成后重启Jenkins使其生效

#### 2.系统管理配置Node

![image-20230321200438675](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230321200438675.png)

#### 3.新建自由风格项目

![image-20230321200557255](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230321200557255.png)

#### 4.配置项目

General

![image-20230321200725408](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230321200725408.png)

![image-20230321200758057](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230321200758057.png)

源码管理

​		注意：需要提前将项目上传到gitlab/github仓库

![image-20230321200912765](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230321200912765.png)

构建环境

![image-20230321201218621](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230321201218621.png)

构建

![image-20230322102736415](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230322102736415.png)

shell内容

```
cd /root/.jenkins/workspace/vue #进入项目目录;这里是war包安装
npm config set registry https://registry.npm.taobao.org #npm源设置为淘宝源
npm config get registry #检测npm是否切换成功
#npm i --unsafe-perm
#npm install -g npm
#npm i node-sass -D
npm update
#npm install yarn
npm run-script build #打包
rm -rf dist.tar.gz #删除上次打包生成的压缩文件
tar -zcvf dist.tar.gz dist/ #打包
```

注意：因为node环境构建出现问题的可能性较大，#号部分根据构建报错逐渐打开

构建后：Send build artifacts over SSH

![image-20230322102935384](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230322102935384.png)

脚本

```shell
[root@localhost opt]# cat jenkins-nginx.sh
#!/usr/bin/bash
#该项目是构建nginx的vue项目,并实现持续构建

#使用过程中注意全局变量的修改
web_path="/dist" #定义网站发布目录变量(可以修改)
job_path="/root/upload"
files_dir=dist  #构建后的目录名字
files=dist.tar.gz   #构建后压缩包的名字
UPDATE="/update/`date +%F-%T`/"
BACKUP="/backup/`date +%F-%T`/"

mkdir -p $UPDATE
mkdir -p $BACKUP

#将项目移动到更新目录下

mv ${job_path}/${files} $UPDATE


if [ -d /dist ];then

        tar cvf $BACKUP/`date +%F-%T`-$files $web_path
        if [ $? -ne 0 ];then
                echo "打包失败"
                exit 1
        else
                rm -rf $web_path

                tar xf $UPDATE/$files -C /
                chmod 777 /dist/ -R
        fi
else
        tar xf $UPDATE/$files -C /
        chmod 777 /dist/ -R
fi
```

## 三：WebHook持续构建发布

#### 1.Gitlab配置

开启出站请求

![image-20230322182537947](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230322182537947.png)

![image-20230322182556301](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230322182556301.png)

#### 2.jenkin配置

安装webhook插件（略）

项目开启webhook

![image-20230322182720390](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230322182720390.png)

#### 3.Gitlab关联jenkins的url

![image-20230322182808998](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230322182808998.png)

![image-20230322182903014](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230322182903014.png)

![image-20230322182920485](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230322182920485.png)

![image-20230322182938211](https://xingdian-image.oss-cn-beijing.aliyuncs.com/xingdian-image/image-20230322182938211.png)

5.总结

​		修改项目源码，提交后，会自动进行项目构建和发布
