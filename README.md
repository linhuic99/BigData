# Hadoop小记
### Eclipse Haddop插件安装
1.在Apache官网中搜索与服务器版本对应的Ecplise插件jar包(这里使用:hadoop-eclipse-plugin-2.7.2.jar)[下载地址:http://download.csdn.net/download/tondayong1981/9432425]<br/>
2.将下载好的hadoop-eclipse-plugin-2.7.2.jar放到Eclipse的plugins目录下<br/>
3.重启Ecplise<br/>
4.在Window->Show View中找到(MapReduce Tools)说明安装成功


### Hadoop单机部署(Standalone mode)
<b>注:这种模式,仅1个节点运行1个java进程,主要用于调式。</b><br/>
1.mkdir /home/hadoop/input(在Hadoop的安装目录下，创建input目录)<br/>
2.cp /home/hadoop/hadoop-2.7.3/etc/hadoop/\*.xml /home/hadoop/input
(拷贝input文件到input目录下)<br/>
3./home/hadoop/hadoop-2.7.3/bin/hadoop jar /home/hadoop/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.0.jar grep /home/hadoop/input /home/hadoop/output 'dfs[a-z.]+'(执行Hadoop job)<br/>
4.上面的job是使用hadoop自带的样例，在input中统计含有dfs的字符串。<br/>
5.确认执行结果 cat /home/hadoop/out/\*<br/>
&nbsp;&nbsp;&nbsp;问题点:<br/>
&nbsp;&nbsp;&nbsp;WARN io.ReadaheadPool: Failed readahead on ifile<br/>
&nbsp;&nbsp;&nbsp;EBADF: Bad file descriptor<br/>
&nbsp;&nbsp;&nbsp;如果出现上面的警告，是因为快速读取文件的时候，文件被关闭引起，也可能是其他bug导致，此处忽略。<br/>

### Hadoop伪分布式部署(Pseudo-Distributed mode)
<b>注:这种模式是,1个节点上运行,HDFS daemon的 NameNode 和 DataNode、YARN daemon的 ResourceManger 和 NodeManager，分别启动单独的java进程，主要用于调试。</b><br/><br/>
1.修改如下设置文件<br/>
\# vi etc/hadoop/core-site.xml<br/>

\<configuration\><br/>
    \<property\><br/>
        \<name\>fs.defaultFS\</name\><br/>
        \<value\>hdfs://localhost:9000\</value\><br/>
    \</property\><br/>
\</configuration\><br/>

\# vi etc/hadoop/hdfs-site.xml<br/>

\<configuration\><br/>
    \<property\><br/>
        \<name\>dfs.replication\</name\><br/>
        \<value\>1\</value\><br/>
    \</property\><br/>
\</configuration\><br/>

2.设置本机的无密码ssh登陆(命令如下)<br/>
\# ssh-keygen -t rsa <br/>
\# cat ~/.ssh/id\_rsa.pub >> ~/.ssh/authorized\_keys

3.MapReduce V2叫做YARN,下面分别操作一下这两种job<br/>
4.执行MapReduce job<br/>
5.格式化文件系统(如下命令内容)<br/>
\# hdfs namenode -format

6.启动namenode和datanode后台进程<br/>
\# sbin/start-dfs.sh<br/>

7.确认启动情况<br/>
\# jps<br/>

8.访问namenode的web界面<br/>
\# curl http://localhost:50070/<br/>

9.创建HDFS
\# ./hdfs dfs -mkdir /user/<br/>
\# ./hdfs dfs -mkdir /user/test<br/>

10.拷贝input文件到HDFS目录下<br/>
\# ./hdfs dfs -put /home/hadoop/hadoop-2.7.3/etc/hadoop/ /user/test/input<br/>
\# ./hadoop fs -ls /user/test/input<br/>

11.执行Hadoop job<br/>
\# ./hadoop jar ../share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.0.jar grep /user/test/input output 'dfs[a-z.]+'<br/>

12.确认执行结果<br/>
\# ./hadoop dfs -cat output/*<br/>

13.停止daemon<br/>
\# sbin/stop-dfs.sh<br/>

### Hadoop伪分布式部署(Pseudo-Distributed mode)--YARN
<b>注:这种模式是,1个节点上运行,HDFS daemon的 NameNode 和 DataNode、YARN daemon的 ResourceManger 和 NodeManager，分别启动单独的java进程，主要用于调试。</b>
MapReduce V2框架叫YARN<br/>

1.修改YARN 设定文件<br/>
\# cp /home/hadoop/hadoop-2.7.3/etc/hadoop/mapred-site.xml.template /home/hadoop/hadoop-2.7.3/etc/hadoop/mapred-site.xml<br/>
\# vi /home/hadoop/hadoop-2.7.3/etc/hadoop/mapred-site.xml<br/>
\<configuration\><br/>
    \<property\><br/>
        \<name\>mapreduce.framework.name\</name\></br>
        \<value\>yarn\</value\><br/>
    \</property\><br/>
\</configuration\><br/>

\# vi /home/hadoop/hadoop-2.7.3/etc/hadoop/yarn-site.xml<br/>
\<configuration\><br/>
    \<property\><br/>
        \<name\>yarn.nodemanager.aux-services\</name\><br/>
        \<value\>mapreduce_shuffle\</value\><br/>
    \</property\><br/>
\</configuration\><br/>

2.启动ResourceManger和NodeManager后台进程
\# sbin/start-yarn.sh

3.确认
\# jps

4.访问ResourceManger的web界面
\# curl http://localhost:8088/

5.创建HDFS
\# ./hdfs dfs -mkdir /user/<br/>
\# ./hdfs dfs -mkdir /user/test<br/>

6.拷贝input文件到HDFS目录下<br/>
\# ./hdfs dfs -put /home/hadoop/hadoop-2.7.3/etc/hadoop/ /user/test/input<br/>
\# ./hadoop fs -ls /user/test/input<br/>

7.执行Hadoop job<br/>
\# ./hadoop jar ../share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.0.jar grep /user/test/input output 'dfs[a-z.]+'<br/>

8.确认执行结果<br/>
\# ./hadoop dfs -cat output/*<br/>

13.停止daemon<br/>
\# sbin/stop-yarn.sh<br/>

### Hadoop 常用命令
1../hadoop-daemon.sh --script hdfs start datanode(执行后只能启动当前节点)<br/>
2../hadoop-demand.sh --script hdfs stop  datanode(执行后只能停止当前节点)<br/>
3../start-dfs.sh(启动hadoop)<br/>
4../stop-dfs.sh (停止hadoop)<br/>
5../start-yarn.sh(启动hadoop yarn)<br/>
6../stop-yarn.sh(停止hadoop yarn)<br/>
7../mr-jobhistory-daemon.sh start historyserver(启动job历史服务器)</br>
8../mr-jobhistory-daemon.sh stop historyserver(停止job历史服务器)</br>
9../hdfs dfs -mkdir /user(创建HDFS文件夹)<br/>
10../hdfs dfs -put /home/hadoop/hadoop2.7.3/etc/hadoop/ /user/test/input(拷贝input文件到HDFS目录下)</br>
11../hadoop fs -ls /user/test/input(查看hadoop指定目录的文件信息)<br/>
12../hdfs dfs -get output output(从HDFS复制文件到本地)
13../hdfs dfs -cat output/*(查看指定目录的执行结果)
14../hadoop jar ../share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.0.jar grep input output 'dfs[a-z.]+'(执行Standalone mode 模式的Hadoop job<在input中统计含有dfs的字符串>)<br/>

### Hadoop常用工具
1.当hadoop启动成功后可以进行访问(http://{ip}:50070)NameNode管理界面内容<br/>
2.当hadoop yarn启动成功后可以进行访问(http://{ip}:8088)ResourceManager管理界面内容<br/>
3.当jobhistory daemon启动成功后可以进行访问(http://{ip}:19888)Job History Server管理界面内容<br/>


### 常用配置
1.hdfs-site.xml 中的dfs.permissions(HDFS文件的权限是否开启[value=true or false])