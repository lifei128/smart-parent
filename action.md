**新浪股票数据统计分析平台**
# demo目的
- - 核心框架实现，主流程跑通
## 1.接收方式
- 爬虫抓取api（http://blog.csdn.net/simon803/article/details/7784682）落数据文件
- flume上报kafka
## 解析器
- 支持和配置中心、埋点中心的集成
- 支持配置解析该类型日志
- 支持版本维护
## 计算器
- 使用spark平台落地hive
- 支持配置计算规则
