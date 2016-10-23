# 解析器
http://note.youdao.com/noteshare?id=01aa0f4e3ccc3c9b15775a7ae5d4eda6
## 说明
- 集成zk/redis，获取变更通知（或直接获取最新解析配置文件。）
- 支持解析规则配置
- 支持解析和标准化两类api
    + 解析用于获取数据字段
    + 标准化用于存储中间结果。从原始日志调用标准化api存入中间存储，从中间结果调用解析api获取字段
- 解析流程图
```
graph LR
源日志-->中间结果
中间结果--是否序列化-->标准化存储
中间结果-- 直接返回 -->解析结果:Map
标准化存储--是否反序列化-->解析结果:Map
```
- api：
    + getParser(conf: String): Parser
    + + 指定配置文件，对应使用默认解析配置的root key，返回解析器对象
    + getParser(conf: String, key: String): Parser
    + + 指定配置文件，对应解析配置的root key，返回解析器对象
    + parse(line: String): Map[String, Object]
    + + 传入一行日志，解析返回Map[String, Object]；
    + + 如果line是原始日志，调用解析规则链解析出kv；
    + + 如果line是序列化结果，则反序列为Map对象；
    + serialize(): String
    + + 把解析结果map序列化为字符串，用于存储中间结果

- demo
1. 解析器启动配置
```
{
  "desc": "驱动组件启动的配置",
  "sys_conf_component_name": "sys_parser",
  "sys_zkAddress": "localhost:2181",
  "sys_sessionTimeout": 5000,
  "sys_rootPath": "/titan/parser",
  "sys_version": "0.0.1"
}

```
2. 全局配置
```
{
  "desc": "全局配置",
  "sys_version": "0.0.3",
  "sys_conf_component_keys": ["sys_parser", "sys_compute", "sys_store", "sys_monitor"],
  "sys_conf_cache_version_num": 3,

  "info": "公共参数",
  "sys_zkAddress": "localhost:2181",
  "sys_sessionTimeout": 5000,
  "sys_rootPath": "/titan/parser",

  "sys_parser": {
    "desc": "驱动组件具体工作的配置",
    "sys_version": "0.0.1",
    "sys_conf_component_name": "sys_parser",
    "sys_rule": {
      "sys_plugins": [],
      "sys_rules": [
        "String[] labels = \"name\tage\taddress\tphone\tsex	info\ttitle\".split(\"\t\");",
        "String[] data = \"_src_\".split(\"\t\");",
        "if(name.equals(\"nel\")){title=\"so high\"; age=20;}"
      ]
    }
  },
  "sys_compute": {
    "desc": "驱动组件具体工作的配置",
    "sys_version": "0.0.1"
  },
  "sys_store": {
    "desc": "驱动组件具体工作的配置",
    "sys_version": "0.0.1"
  },
  "sys_monitor": {
    "desc": "驱动组件具体工作的配置",
    "sys_version": "0.0.1"
  }
}
```
3. 解析过程
```
graph LR
组件配置 --驱动--> 解析器组件
全局配置 --发布变更--> 配置中心::zookeeper
配置中心::zookeeper --事件通知--> 解析器组件
解析器组件 --实例化--> 解析器
源日志--> 解析器
解析器组件 --更新内部参数--> 解析器
解析器 --词法解析-->中间结果:Map
中间结果:Map --直接返回--> 解析结果:Map
中间结果:Map --序列化--> 标准化存储:kafka/hdfs/es.etc
```
4. 解析日志
```
[2016-10-23 21:18:42]--[main] [INFO] -com.nel.component.operator.driver$.main(OperatorTrait.scala:416) - jobConf:{  "desc": "驱动组件启动的配置",  "sys_conf_component_name": "sys_parser",  "sys_zkAddress": "localhost:2181",  "sys_sessionTimeout": 5000,  "sys_rootPath": "/titan/parser",  "sys_version": "0.0.1"}
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:host.name=192.168.1.101
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:java.version=1.7.0_80
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:java.vendor=Oracle Corporation
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:java.home=/Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home/jre
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:java.library.path=/Users/nel/Library/Java/Extensions:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java:.
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:java.io.tmpdir=/var/folders/wt/x0k0x9sj4mld5yytm419rv_w0000gn/T/
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:java.compiler=<NA>
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:os.name=Mac OS X
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:os.arch=x86_64
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:os.version=10.10.5
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:user.name=nel
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:user.home=/Users/nel
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.Environment.logEnv(Environment.java:100) - Client environment:user.dir=/Users/nel/workspace/github/smart-tools
[2016-10-23 21:18:42]--[main] [INFO] -org.apache.zookeeper.ZooKeeper.<init>(ZooKeeper.java:438) - Initiating client connection, connectString=localhost:2181 sessionTimeout=5000 watcher=com.nel.zookeeper.event.ZookeeperClient$$anon$2@679cac0d
[2016-10-23 21:18:42]--[main-SendThread(localhost:2181)] [INFO] -org.apache.zookeeper.ClientCnxn$SendThread.logStartConnect(ClientCnxn.java:975) - Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
[2016-10-23 21:18:42]--[main-SendThread(localhost:2181)] [INFO] -org.apache.zookeeper.ClientCnxn$SendThread.primeConnection(ClientCnxn.java:852) - Socket connection established to localhost/127.0.0.1:2181, initiating session
[2016-10-23 21:18:42]--[main-SendThread(localhost:2181)] [INFO] -org.apache.zookeeper.ClientCnxn$SendThread.onConnected(ClientCnxn.java:1235) - Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x157f18de7c90007, negotiated timeout = 5000
[2016-10-23 21:18:42]--[main-EventThread] [WARN] -com.nel.component.ComponentBase.subscribe(ComponentBase.scala:69) - [watch event]
[2016-10-23 21:18:42]--[main-EventThread] [INFO] -com.nel.component.ComponentBase.updateConf(ComponentBase.scala:84) - path:{"/titan/parser":{"data":{"sys_store":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_sessionTimeout":5000,"sys_version":"0.0.3","desc":"全局配置","sys_monitor":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_compute":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_rootPath":"/titan/parser","sys_parser":{"sys_rule":{"sys_plugins":[],"sys_rules":["String[] labels = \"name\tage\taddress\tphone\tsex\tinfo\ttitle\".split(\"\t\");","String[] data = \"_src_\".split(\"\t\");","if(name.equals(\"nel\")){title=\"so high\"; age=20;}"]},"sys_conf_component_name":"sys_parser","sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_zkAddress":"localhost:2181","sys_conf_cache_version_num":3,"sys_conf_component_keys":["sys_parser","sys_compute","sys_store","sys_monitor"],"info":"公共参数"}}}, newConf:{}
[2016-10-23 21:18:42]--[main-EventThread] [INFO] -com.nel.component.ComponentBase.updateConf(ComponentBase.scala:94) - 0
[2016-10-23 21:18:42]--[main-EventThread] [INFO] -com.nel.component.ComponentBase.updateConf(ComponentBase.scala:107) - [start]globalConfQueueMap:Map(/titan/parser -> Queue({"data":{"sys_store":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_sessionTimeout":5000,"sys_version":"0.0.3","desc":"全局配置","sys_monitor":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_compute":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_rootPath":"/titan/parser","sys_parser":{"sys_rule":{"sys_plugins":[],"sys_rules":["String[] labels = \"name\tage\taddress\tphone\tsex\tinfo\ttitle\".split(\"\t\");","String[] data = \"_src_\".split(\"\t\");","if(name.equals(\"nel\")){title=\"so high\"; age=20;}"]},"sys_conf_component_name":"sys_parser","sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_zkAddress":"localhost:2181","sys_conf_cache_version_num":3,"sys_conf_component_keys":["sys_parser","sys_compute","sys_store","sys_monitor"],"info":"公共参数"}}))
[2016-10-23 21:18:42]--[main-EventThread] [INFO] -com.nel.component.ComponentBase.updateConf(ComponentBase.scala:110) - [start] Check confQueue length. max is 3, cur is 1
[2016-10-23 21:18:42]--[main-EventThread] [INFO] -com.nel.component.ComponentBase.updateConf(ComponentBase.scala:117) - [end] Check confQueue length. max is 3, cur is 1
[2016-10-23 21:18:42]--[main-EventThread] [INFO] -com.nel.component.ComponentBase.updateConf(ComponentBase.scala:118) - [end] globalConfQueueMap:Map(/titan/parser -> Queue({"data":{"sys_store":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_sessionTimeout":5000,"sys_version":"0.0.3","desc":"全局配置","sys_monitor":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_compute":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_rootPath":"/titan/parser","sys_parser":{"sys_rule":{"sys_plugins":[],"sys_rules":["String[] labels = \"name\tage\taddress\tphone\tsex\tinfo\ttitle\".split(\"\t\");","String[] data = \"_src_\".split(\"\t\");","if(name.equals(\"nel\")){title=\"so high\"; age=20;}"]},"sys_conf_component_name":"sys_parser","sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_zkAddress":"localhost:2181","sys_conf_cache_version_num":3,"sys_conf_component_keys":["sys_parser","sys_compute","sys_store","sys_monitor"],"info":"公共参数"}}))
[2016-10-23 21:18:42]--[main-EventThread] [INFO] -com.nel.component.ComponentBase.getCompnentConf(ComponentBase.scala:30) - queue:Queue({"data":{"sys_store":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_sessionTimeout":5000,"sys_version":"0.0.3","desc":"全局配置","sys_monitor":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_compute":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_rootPath":"/titan/parser","sys_parser":{"sys_rule":{"sys_plugins":[],"sys_rules":["String[] labels = \"name\tage\taddress\tphone\tsex\tinfo\ttitle\".split(\"\t\");","String[] data = \"_src_\".split(\"\t\");","if(name.equals(\"nel\")){title=\"so high\"; age=20;}"]},"sys_conf_component_name":"sys_parser","sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_zkAddress":"localhost:2181","sys_conf_cache_version_num":3,"sys_conf_component_keys":["sys_parser","sys_compute","sys_store","sys_monitor"],"info":"公共参数"}})
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.ComponentBase.getCompnentConf(ComponentBase.scala:30) - queue:Queue({"data":{"sys_store":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_sessionTimeout":5000,"sys_version":"0.0.3","desc":"全局配置","sys_monitor":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_compute":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_rootPath":"/titan/parser","sys_parser":{"sys_rule":{"sys_plugins":[],"sys_rules":["String[] labels = \"name\tage\taddress\tphone\tsex\tinfo\ttitle\".split(\"\t\");","String[] data = \"_src_\".split(\"\t\");","if(name.equals(\"nel\")){title=\"so high\"; age=20;}"]},"sys_conf_component_name":"sys_parser","sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_zkAddress":"localhost:2181","sys_conf_cache_version_num":3,"sys_conf_component_keys":["sys_parser","sys_compute","sys_store","sys_monitor"],"info":"公共参数"}})
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$class.parse(OperatorTrait.scala:256) - jobConf:{"sys_rule":{"sys_plugins":[],"sys_rules":["String[] labels = \"name\tage\taddress\tphone\tsex\tinfo\ttitle\".split(\"\t\");","String[] data = \"_src_\".split(\"\t\");","if(name.equals(\"nel\")){title=\"so high\"; age=20;}"]},"sys_conf_component_name":"sys_parser","sys_version":"0.0.1","desc":"驱动组件具体工作的配置"}
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$parse$4.apply(OperatorTrait.scala:281) - start rules with:String[] labels = "name	age	address	phone	sex	info	title".split("	");
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$parse$4.apply(OperatorTrait.scala:281) - start rules with:String[] data = "_src_".split("	");
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$parse$4.apply(OperatorTrait.scala:281) - start rules with:if(name.equals("nel")){title="so high"; age=20;}
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$class.parse(OperatorTrait.scala:284) - [String[] labels = "name, age, address, phone, sex, info, title".split(", ");]
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lsplitBody$2.apply(OperatorTrait.scala:183) - name	age	address	phone	sex	info	title
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lsplitBody$2.apply(OperatorTrait.scala:184) - "	");
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lsplitSyntax$4.apply(OperatorTrait.scala:199) - (labels~[Ljava.lang.String;@5c4ddc5)
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lsplitBody$2.apply(OperatorTrait.scala:183) - _src_
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lsplitBody$2.apply(OperatorTrait.scala:184) - "	");
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lsplitSyntax$4.apply(OperatorTrait.scala:199) - (data~[Ljava.lang.String;@3557133c)
[2016-10-23 21:18:47]--[main] [WARN] -com.nel.component.operator.OperatorTrait$class.parse(OperatorTrait.scala:288) - begin.data_source:Map(), Map(labels -> [Ljava.lang.String;@5c4ddc5, data -> [Ljava.lang.String;@3557133c), meta:{}
name
age
address
phone
sex
info
title
(name,nel)
(age,18)
(info,a engineer at china)
(sex,man)
(address,beijing.chaoyang)
(title,())
(phone,10086)
[2016-10-23 21:18:47]--[main] [WARN] -com.nel.component.operator.OperatorTrait$class.parse(OperatorTrait.scala:303) - mid.data_source:Map(name -> nel, age -> 18, info -> a engineer at china, sex -> man, address -> beijing.chaoyang, title -> (), phone -> 10086), Map(labels -> [Ljava.lang.String;@5c4ddc5, data -> [Ljava.lang.String;@3557133c), meta:{}
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$parse$8.apply(OperatorTrait.scala:311) - x:if(name.equals("nel")){title="so high"; age=20;}, index:2
(name,equals,nel)
[2016-10-23 21:18:47]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lifSyntax$2.apply(OperatorTrait.scala:166) - true, Map(title -> "so high", age -> 20)
[2016-10-23 21:18:47]--[main] [WARN] -com.nel.component.operator.OperatorTrait$class.parse(OperatorTrait.scala:316) - end.data_source:Map(name -> nel, age -> 20, info -> a engineer at china, sex -> man, address -> beijing.chaoyang, title -> "so high", phone -> 10086), Map(labels -> [Ljava.lang.String;@5c4ddc5, data -> [Ljava.lang.String;@3557133c), meta:{}
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.ComponentBase.getCompnentConf(ComponentBase.scala:30) - queue:Queue({"data":{"sys_store":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_sessionTimeout":5000,"sys_version":"0.0.3","desc":"全局配置","sys_monitor":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_compute":{"sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_rootPath":"/titan/parser","sys_parser":{"sys_rule":{"sys_plugins":[],"sys_rules":["String[] labels = \"name\tage\taddress\tphone\tsex\tinfo\ttitle\".split(\"\t\");","String[] data = \"_src_\".split(\"\t\");","if(name.equals(\"nel\")){title=\"so high\"; age=20;}"]},"sys_conf_component_name":"sys_parser","sys_version":"0.0.1","desc":"驱动组件具体工作的配置"},"sys_zkAddress":"localhost:2181","sys_conf_cache_version_num":3,"sys_conf_component_keys":["sys_parser","sys_compute","sys_store","sys_monitor"],"info":"公共参数"}})
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$class.parse(OperatorTrait.scala:256) - jobConf:{"sys_rule":{"sys_plugins":[],"sys_rules":["String[] labels = \"name\tage\taddress\tphone\tsex\tinfo\ttitle\".split(\"\t\");","String[] data = \"_src_\".split(\"\t\");","if(name.equals(\"nel\")){title=\"so high\"; age=20;}"]},"sys_conf_component_name":"sys_parser","sys_version":"0.0.1","desc":"驱动组件具体工作的配置"}
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$parse$4.apply(OperatorTrait.scala:281) - start rules with:String[] labels = "name	age	address	phone	sex	info	title".split("	");
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$parse$4.apply(OperatorTrait.scala:281) - start rules with:String[] data = "_src_".split("	");
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$parse$4.apply(OperatorTrait.scala:281) - start rules with:if(name.equals("nel")){title="so high"; age=20;}
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$class.parse(OperatorTrait.scala:284) - [String[] labels = "name, age, address, phone, sex, info, title".split(", ");]
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lsplitBody$2.apply(OperatorTrait.scala:183) - name	age	address	phone	sex	info	title
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lsplitBody$2.apply(OperatorTrait.scala:184) - "	");
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lsplitSyntax$4.apply(OperatorTrait.scala:199) - (labels~[Ljava.lang.String;@3764253e)
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lsplitBody$2.apply(OperatorTrait.scala:183) - _src_
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lsplitBody$2.apply(OperatorTrait.scala:184) - "	");
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lsplitSyntax$4.apply(OperatorTrait.scala:199) - (data~[Ljava.lang.String;@fc925db)
[2016-10-23 21:18:57]--[main] [WARN] -com.nel.component.operator.OperatorTrait$class.parse(OperatorTrait.scala:288) - begin.data_source:Map(), Map(labels -> [Ljava.lang.String;@3764253e, data -> [Ljava.lang.String;@fc925db), meta:{}
name
age
address
phone
sex
info
title
(name,nel)
(age,11)
(info,a engineer at china)
(sex,man)
(address,beijing.chaoyang)
(title,())
(phone,10086)
[2016-10-23 21:18:57]--[main] [WARN] -com.nel.component.operator.OperatorTrait$class.parse(OperatorTrait.scala:303) - mid.data_source:Map(name -> nel, age -> 11, info -> a engineer at china, sex -> man, address -> beijing.chaoyang, title -> (), phone -> 10086), Map(labels -> [Ljava.lang.String;@3764253e, data -> [Ljava.lang.String;@fc925db), meta:{}
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$parse$8.apply(OperatorTrait.scala:311) - x:if(name.equals("nel")){title="so high"; age=20;}, index:2
(name,equals,nel)
[2016-10-23 21:18:57]--[main] [INFO] -com.nel.component.operator.OperatorTrait$$anonfun$lifSyntax$2.apply(OperatorTrait.scala:166) - true, Map(title -> "so high", age -> 20)
[2016-10-23 21:18:57]--[main] [WARN] -com.nel.component.operator.OperatorTrait$class.parse(OperatorTrait.scala:316) - end.data_source:Map(name -> nel, age -> 20, info -> a engineer at china, sex -> man, address -> beijing.chaoyang, title -> "so high", phone -> 10086), Map(labels -> [Ljava.lang.String;@3764253e, data -> [Ljava.lang.String;@fc925db), meta:{}



```


| 时间 | 修改 | 修改人 |
| :------:| ------: | :------:  | 
| 2016-10-18 | 新增 | 李奇      |
| 2016-10-18 |      |           |
| 2016-10-18 |      |           |




