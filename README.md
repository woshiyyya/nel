nel
======================

__nel__ is an fast, accurate and highly modular framework for linking entities in documents.

### Getting Started

Checkout the up-to-date example notebook: [train.ipynb](notebooks/train.ipynb)

Documentation for the previous release is available online via: [nel.readthedocs.org](http://nel.readthedocs.org/en/latest/).

### Citation

__nel__ is based on work described in *Entity Disambiguation with Web Links* ([pdf](http://aclweb.org/anthology/Q15-1011)).

----------------
NEL is open-source software released under an MIT licence.

想用别人的轮子真的是困难，在gpu18上新开账号，记录配置过程
# 从头开始配置nel
OS：Ubuntu 16.04LTS
## 1.安装anaconda3
下载anaconda3 （https://www.anaconda.com/download/#linux）
```
wget https://repo.anaconda.com/archive/Anaconda3-5.2.0-Linux-x86_64.sh
bash Anaconda3-5.2.0-Linux-x86_64.sh
```
新建python2环境
```
conda create -n py2 python=2.7
```

## 2.安装配置JAVA
下载jdk8_xxx.tar.gz
```
tar -xzvf jdk8_xxx.tar.gz
```
添加PATH到/etc/profile
```
sudo vi /etc/profile
```
将下列内容添加到最后
```
export JAVA_HOME=/[usr]/jdk1.8.0_181
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```
立即激活并检查
```
source /etc/profile
java -version
```

## 3.安装配置TOMCAT
```
sudo apt-get install tomcat8 tomcat8-docs tomcat8-examples tomcat8-admin
```
安装目录为 **/usr/share/tomcat8**
```
sudo chmod 777 -R tomcat8
cd tomcat8/bin
bash startup.sh
```
若出现cannot touch /usr/share/tomcat8/log/catalina.out, 手动建立logs文件夹并修改权限  
启动成功！

### 更改JVM最大堆内存参数 ~~**（有问题）**~~ 已解决

在catalina.sh 添加,(这里设置了4G) 
```
JAVA_OPTS="-Xms1024m -Xmx4090m"
```
还是报错GC overhead limited！！ 这个大坑  
最终发现是SPARK的问题，而且并不是worker的内存不够，而是driver

## 解决方法
在$SPARK_HOME/conf/目录中，将spark-defaults.conf.template模板文件拷贝一份到/spark_home/conf目录下，命名为spark-defaults.conf，然后在里面设置spark.driver.memory  memSize属性来改变driver内存大小。
```
spark.driver.memory                5g
```


## 4. 安装scala
从 https://www.scala-lang.org/download/ 找到压缩包
```
sudo mkdir /sur/lib/scala
sudo tar -zxvf scala-2.12.6.tgz -C /usr/lib/scala
```

打开/etc/profile, 在最后添加配置
```
export SCALA_HOME=/usr/lib/scala/scala-2.12.6
export PATH=${SCALA_HOME}/bin:$PATH
```
激活profile，输入scala测试是否安装成功。

## 5.安装配置Spark
官网下载 http://spark.apache.org/downloads.html   
解压到 /usr/lib/spark 目录  
修改 /etc/profile
```
export SPARK_HOME=/usr/lib/spark/spark-2.3.1-bin-hadoop2.7
export PATH=${SPARK_HOME}/bin:$PATH
```
### SPARK默认参数
进入$SPARK_HOME/conf文件夹，拷贝一份spark-default.conf,添加
```
spark.driver.memory  10g
spark.executor.memory 2g
spark.maxResultSize  10g
```


### python 中使用pyspark 
参见 https://blog.csdn.net/u010171031/article/details/51849562

## 6. 安装nel及sift库
安装sift
```
pip install git+http://git@github.com/wikilinks/sift.git
pip install git+http://git@github.com/wikilinks/nel.git
```
出错： EnvironmentError: mysql_config not found，原因是没有安装 libmysqlclient-dev
```
sudo apt-get install libmysqlclient-dev
```
又报GCC的错
```
sudo apt-get install gcc
sudo apt-get install build-essential
```

## 7.解决库错误
```python
import nel
>>> No module named en
```
是没有安装英文包，终端输入
```
python -m spacy download en
```
再次import仍然报错 No module named en  

手动修改 /home/yunxuanxiao/anaconda3/envs/py2/lib/python2.7/site-packages/nel/features/recognition.py
```python
form spacy.en import English
改为
from spacy.lang.en import English
```
修改 /home/yunxuanxiao/anaconda3/envs/py2/lib/python2.7/site-packages/nel/process/tag.py 第47行
```python
self.spacy_model = spacy_model or 'en_default'
改为
self.spacy_model = spacy_model or 'en'
```



