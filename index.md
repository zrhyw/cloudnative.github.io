# 一.promtheus简介

## 1.简介

```
    Prometheus是一个开源的系统监控和报警系统，现在已经加入到CNCF基金会，成为继k8s之后第二个在CNCF托管的项目，在kubernetes容器管理系统中，通常会搭配prometheus进行监控，同时也支持多种exporter采集数据，还支持pushgateway进行数据上报，Prometheus性能足够支撑上万台规模的集群。
```

## 2.promtheus特点

```
(1)多维度数据模型
每一个时间序列数据都由metric度量指标名称和它的标签labels键值对集合唯一确定：这个metric度量指标名称指定监控目标系统的测量特征（如：http_requests_total- 接收http请求的总计数）。labels开启了Prometheus的多维数据模型：对于相同的度量名称，通过不同标签列表的结合, 会形成特定的度量维度实例。(例如：所有包含度量名称为/api/tracks的http请求，打上method=POST的标签，则形成了具体的http请求)。这个查询语言在这些度量和标签列表的基础上进行过滤和聚合。改变任何度量上的任何标签值，则会形成新的时间序列图。
(2)灵活的查询语言（PromQL）：可以对采集的metrics指标进行加法，乘法，连接等操作；
(3)可以直接在本地部署，不依赖其他分布式存储；
(4)通过基于HTTP的pull方式采集时序数据；
(5)可以通过中间网关pushgateway的方式把时间序列数据推送到prometheus server端；
(6)可通过服务发现或者静态配置来发现目标服务对象（targets）。
(7)有多种可视化图像界面，如Grafana等。
(8)高效的存储，每个采样数据占3.5 bytes左右，300万的时间序列，30s间隔，保留60天，消耗磁盘大概200G。
(9)做高可用，可以对数据做异地备份，联邦集群，部署多套prometheus，pushgateway上报数据
```

## 3.promtheus组件

### 3.1promtheus工作组件

```
(1)Prometheus Server: 用于收集和存储时间序列数据。
(2)Exporters: prometheus支持多种exporter，通过exporter可以采集metrics数据，然后发送到prometheus server端
(3)Alertmanager: 从 Prometheus server 端接收到告警后，会进行去重，分组，并路由到相应的接收方，发出报警，常见的接收方式有：邮件，企业微信，钉钉
(4)Grafana：监控仪表盘，可视化监控数据
(5)pushgateway: 各个目标主机可上报数据到pushgateway，然后prometheus server统一从pushgateway拉取数据。
```

![8956d4f652de60cdbc6dc5f0153553f4](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/8956d4f652de60cdbc6dc5f0153553f4.png)

### 3.2prometheus组件

```
(1)Retrieval负责在活跃的target主机上抓取监控指标数据
(2)Storage存储主要是把采集到的数据存储到磁盘中
(3)PromQL是Prometheus提供的查询语言模块。
```

## 4.prometheus工作流程

```
(1)Prometheus server可定期从活跃的（up）目标主机上（target）拉取监控指标数据，目标主机的监控数据可通过配置静态job或者服务发现的方式被prometheus server采集到，这种方式默认的pull方式拉取指标；也可通过pushgateway把采集的数据上报到prometheus server中
(2)Prometheus server把采集到的监控指标数据保存到本地磁盘或者数据库；
(3)Prometheus采集的监控指标数据按时间序列存储，通过配置报警规则，把触发的报警发送到alertmanager
(4)Alertmanager通过配置报警接收方，发送报警到邮件，微信或者钉钉等
(5)Prometheus 自带的web ui界面提供PromQL查询语言，可查询监控数据
(6)Grafana可接入prometheus数据源，把监控数据以图形化形式展示出
```



# 二.promtheus语句

## 1.数据类型

### 1.1计数器（Counter）

```
从数据量为0开始累积计算，在理想状态下只能永远增长，不会降低，例如对用户访问量的采样数据；
Counter数据类型的特点：
    Counter 用于累计值，例如 记录 请求次数、任务完成数、错误发生次数。
    一直增加，不会减少。
    重启进程后，会被重置。
    
案例:
prometheus_http_requests_total    #http请求量
```

![image-20230605151320587](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/image-20230605151320587.png)

### 1.2仪表（Gauge）

```
最简单的度量指标，只有一个简单的返回值（瞬时状态），例如监控磁盘或者内存的使用量，那么就应该使用gauges格式来度量；
Gauge数据类型的特点：
    Gauge 常规数值，例如 温度变化、内存使用变化。
    可变大，可变小。
    重启进程后，会被重置
    
案例:
node_memory_Active_bytes /1024/1024/1024     #node剩余内存
```

![image-20230605150423719](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/image-20230605150423719.png)

### 1.3直方图（Histogram）

```
histogram是柱状图
    Histogram:累积直方图，会在一段时间范围内对数据进行采样(通常是请求持续时间或响应大小等)
```

### 1.4汇总（Summary）

```
Summary直方图
     通过计算分位数(quantile)显示指标结果，可用于统计一段时间内数据采样结果 
```

## 2.匹配器

```
    =: 精确的匹配指定的标签值
    !=: 不匹配指定的标签值，取反
    =~: 正则表达式匹配指定的标签值
    !~: 不匹配正则表格式指定的标签值

node_memory_MemTotal_bytes{instance="192.168.3.157:9100"}  #精准匹配
node_memory_MemTotal_bytes{instance!="192.168.3.157:9100"}  #匹配取反
node_memory_MemTotal_bytes{instance=~"192.168.3.*:9100"}  #正则匹配
node_memory_MemTotal_bytes{instance!~"192.168.3.*:9100"}   #正则取反
```

## 3.算数运算

```
    +：加法；
    -：减法；
    *：乘法；
    /：除法；
    ^：幂运算；

node_memory_MemTotal_bytes/1024/1024/1024    #将内存单位从字节转换为G
(node_memory_MemTotal_bytes-node_memory_MemFree_bytes)/1024/1024/1024     #所有内存-剩余内存=已使用内存
```

## 4.时间格式

```
ms(毫秒)、s(秒)、m(分钟)、h(小时)、d(天)、w(周)、y(年)

d 表示一天总是24小时
w 表示一周总是7天
y 表示一年总是365天

node_memory_MemFree_bytes[5m]     #查看5分钟内的数据
node_memory_MemFree_bytes[1d]      #查看1天内的数据
```

## 5.聚合运算

### 5.1max，min，avg

```
max()   最大值
min()   最小值
avg()   平均值

max(prometheus_http_requests_total)  #http请求量最大值
min(prometheus_http_requests_total)  #http请求量最小值
avg(prometheus_http_requests_total)  #http请求量平均值
```

### 5.2sum，count

```
sum()       求数据值相加的总和
sum(prometheus_http_requests_total)   #http所有请求的和

count()      统计返回值的条数
count(probe_http_version)
一条返回的数据，统计所有的http_version条数
```

### 5.3abs,absent

```
abs    返回指标数据的值
abs(node_memory_MemTotal_bytes/1024/1024/1024)

absent       #监控指标有数据就返回空，没有数据就返回1，可用于对监控项设置告警通知
absent(node_memory_MemTotal_bytes/1024/1024/1024)
```

### 5.4stddev，stdvar

```
stddev   标准差,统计系统波动的大小，不稳定
stddev(node_memory_MemTotal_bytes/1024/1024/1024)

stdvar  求方差
stdvar(node_memory_MemTotal_bytes/1024/1024/1024)
```

### 5.5topk，bottomk

```
topk       //样本值最大的n个数据
#取剩余内存从大到小的前5个
topk(5,prometheus_http_requests_total)

bottomk   //取样本值排名最小的n个数据
#取剩余内存从小到大的前5个
bottomk(5,prometheus_http_requests_total)
```

### 5.6rate

```
rate   函数专门搭配counter数据类型使用函数，功能是取conunter数据类型这个时间段中平均每秒的增量平均数据
rate(node_network_receive_bytes_total[5m])
```



### 5.7by，without

```
by        保留by指定的标签的值，移除其他值
max(prometheus_http_requests_total) by (instance)

without   移除列举的标签的值，保留其他的标签
max(prometheus_http_requests_total/1024/1024/1024) without (instance)
```



# 三.安装promtheus-server

```
[root@prometheus ~]# mkdir /data/prometheus
[root@prometheus ~]# vim /data/prometheus/prometheus.yml
global:                           #全局配置
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:                       #告警规则，自定义告警规则
scrape_configs:                   #采集配置，配置数据源
  - job_name: "prometheus"        #定义job名称
    static_configs:               #静态配置job
      - targets: ["192.168.3.157:9090"]     #配置promtheus监控地址
[root@prometheus ~]# chmod -R 777  /data/prometheus/*

docker run -d --name prometheus --restart=always -p 9090:9090 -v /data/prometheus/:/etc/prometheus/  prom/prometheus  --config.file=/etc/prometheus/prometheus.yml --web.enable-lifecycle


--config.file=/etc/prometheus/prometheus.yml   #指定prometheus的配置文件
--web.enable-lifecycle                         #启动prometheus热加载
```

![image-20230605115443934](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/image-20230605115443934.png)



# 四.promtheus监控

## 1.node_exporter

### 1.1安装node_exporter

```
# 启动node-exporter
docker run -d --name node-exporter --restart=always -p 9100:9100 -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro" prom/node-exporter
```

### 1.2配置prometheus

```
[root@prometheus ~]# vim /data/prometheus/prometheus.yml
  - job_name: "node_exporter157"
    static_configs:
      - targets: ["192.168.3.157:9100"]
[root@prometheus ~]# curl -X POST http://192.168.3.157:9090/-/reload
```

![image-20230605135739018](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/image-20230605135739018.png)



## 2.mysql_exporter

### 2.1安装mysql

```
docker run -d --name mysql-server -t \
 -v /etc/localtime:/etc/localtime \
 --restart=always \
 -e MYSQL_ROOT_PASSWORD="123456" \
 -p 3306:3306 \
 mysql:5.7
 
CREATE USER 'exporter'@'%' IDENTIFIED BY 'exporter';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%';
flush privileges;
```

### 2.2安装mysql_exporter

```
docker run -d --name mysqld_exporter \
-p 9104:9104 \
-e DATA_SOURCE_NAME="root:123456@(192.168.3.157:3306)/" \
prom/mysqld-exporter
```

### 2.3配置promtheus

```
[root@prometheus ~]# vim /data/prometheus/prometheus.yml
  - job_name: "mysqld_exporter"
    static_configs:
      - targets: ["192.168.3.157:9104"]
[root@prometheus ~]# curl -X POST http://192.168.3.157:9090/-/reload
```

![image-20230605140530605](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/image-20230605140530605.png)



## 3.blackbox_exporter

```
Blackbox Exporter是Prometheus社区提供的官方黑盒监控解决方案，其允许用户通过：HTTP、HTTPS、DNS、TCP以及ICMP的方式对网络进行探测。
```

### 3.1安装blackbox_exporter

```
docker run -d --name blackbox_exporter \
--restart=always \
-p 9115:9115 \
prom/blackbox-exporter
```

### 3.2配置prometheus

```
[root@prometheus ~]# vim /data/prometheus/prometheus.yml
rule_files:
  - /etc/prometheus/rule/*.yml
  
  - job_name: "blackbox_exporter"
    static_configs:
      - targets: ["192.168.3.157:9115"]
[root@prometheus ~]# curl -X POST http://192.168.3.157:9090/-/reload
```

![image-20230605141451828](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/image-20230605141451828.png)



### 3.2监控http

```
docker run -d nginx -p 8888:80
[root@prometheus ~]# vim /data/prometheus/prometheus.yml
  - job_name: "http_status"           #定义job名称
    scrape_interval: 30s              #每次数据采集的时间间隔为30s
    scrape_timeout: 15s               #采集请求超时时间为15秒
    metrics_path: /probe              #从targets获取meitric的HTTP资源路径，默认是/metrics
    params:                           #数据采集访问时HTTP URL设定的参数
      modelus: [http_2xx]             #配置http检测      
    static_configs:                   #静态配置job      
    - targets:                        #配置监控目标访问地址
      - www.baidu.com
    relabel_configs:                   #重新打标签
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: url
    - target_label: __address__
      replacement: 192.168.3.157:9115  #配置blackbox访问地址
[root@prometheus ~]# curl -X POST http://192.168.3.157:9090/-/reload
```



### 3.3监控icmp

```
[root@prometheus ~]# vim /data/prometheus/prometheus.yml
  - job_name: "ping_status"
    metrics_path: /probe
    params:
      modelus: [icmp]
    static_configs:
    - targets:
      - 114.114.114.114
    relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: ip
    - target_label: __address__
      replacement: 192.168.3.157:9115
[root@prometheus ~]# curl -X POST http://192.168.3.157:9090/-/reload
```



### 3.4监控tcp

```
[root@prometheus ~]# vim /data/prometheus/prometheus.yml
  - job_name: "port_status"
    metrics_path: /probe
    params:
      modelus: [tcp_connect]
    static_configs:
    - targets:
      - 192.168.3.157:9090
    relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: tcp
    - target_label: __address__
      replacement: 192.168.3.157:9115
[root@prometheus ~]# curl -X POST http://192.168.3.157:9090/-/reload
```

![image-20230605143224319](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/image-20230605143224319.png)

## 4.alertmanager

### 4.1简介

```
    Prometheus 包含一个报警模块，就是我们的 AlertManager，Alertmanager 主要用于接收 Prometheus 发送的告警信息，它支持丰富的告警通知渠道，而且很容易做到告警信息进行去重，降噪，分组等，是一款前卫的告警通知系统。
```

### 4.2告警过程

```
Prometheus触发一条告警的过程如下：
prometheus server —>触发阈值—>超出指定时间—>alertmanager—>分组|抑制|静默—>媒体类型—>邮件|钉钉|微信等等。
```

![image-20230605145458460](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/image-20230605145458460.png)

```
分组(group):将类似性质的警报合并为单个通知，比如pod通知，node通知
静默(silences):是一种简单的特定时间的静音机制，例如:服务器要升级维护的时候可以设置某个时间段告警静默
抑制(inhibition):当警报发出后，在这个时间段内，将当前警报和其他警报合并然后告警，可以消除冗余警告
```



### 4.3安装

```
mkdir -p /data/alertmanager/tmpl
chmod -R 777 /data/alertmanager/

docker run -d \
  -p 9093:9093 \
  --name alertmanager \
  --restart=always \
  -v /etc/localtime:/etc/localtime \
  -v /data/alertmanager:/etc/alertmanager \
  prom/alertmanager
```

```
[root@prometheus alertmanager]# cat alertmanager.yml
global:
# 经过此时间后，如果尚未更新告警，则将告警声明为已恢复。(即prometheus没有向alertmanager发送告警了)
  resolve_timeout: 5m

templates:                  #配置告警的模板
  - '/etc/alertmanager/tmpl/cluster.tmpl'
  - '/etc/alertmanager/tmpl/domain.tmpl'

route:                      # route用来设置报警的分发策略
  group_by: [alertname]     # 采用哪个标签来作为分组依据
  group_wait: 30s           # 组告警等待时间。也就是告警产生后等待30s，如果有同组告警一起发出
  group_interval: 30s       # 两组告警的间隔时间
  repeat_interval: 1m       # 重复告警的间隔时间，减少相同邮件的发送频率
  receiver: cluster         # 设置默认接收人

  routes:                   # 配置告警路由
  - receiver: domain        # 配置告警接收人
    group_by: [nodeExt2]
    matchers:
    - domain = check        # 匹配告警规则domain=check
    group_interval: 30s     # 两组告警的间隔时间
    group_wait: 30s         # 组告警等待时间。也就是告警产生后等待30s，如果有同组告警一起发出
    repeat_interval: 1m     # 重复告警的间隔时间，减少相同邮件的发送频率

receivers:                  #配置告警接收人
  - name: 'cluster'         #告警接收人为cluster
    wechat_configs:         #配置企业告警
      - corp_id: 'wwbb7bad0a26f72565'
        agent_id: '1000004'
        to_party:  '1'
        api_secret: 'UUFeBkfElEXDx3itLvE_Fs_14UfPU-dsGAsFtzVm5BA'
        to_user: '@all'
        message: '{{ template "cluster.default.message" . }}'     # 配置告警模板
        send_resolved: true          # 发送告警恢复通知

  - name: 'domain'
    wechat_configs:
      - corp_id: 'wwbb7bad0a26f72565'
        agent_id: '1000002'
        to_party:  '2'
        api_secret: 'ievn5h5_cbqdtA9uGlAjkMirKDDL6y3uSEmhRqcg8Jg'
        to_user: '@all'
        message: '{{ template "domain.default.message" . }}'
        send_resolved: true


inhibit_rules:            #抑制规则，当出现critical告警时 忽略warning
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
# 必须在源告警和目标告警中具有相等值的标签才能使抑制生效。(即源告警和目标告警中这三个标签的值相等'alertname', 'cluster', 'service')
```

```
[root@prometheus tmpl]# cat cluster.tmpl
{{ define "cluster.default.message" }}
{{- if gt (len .Alerts.Firing) 0 -}}
{{- range $index, $alert := .Alerts -}}
======== 异常告警 ========
告警名称：{{ $alert.Labels.alertname }}
告警级别：{{ $alert.Labels.severity }}
告警机器：{{ $alert.Labels.instance }}
告警详情：{{ $alert.Annotations.summary }}
告警时间：{{ ($alert.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}
========== END ==========
{{- end }}
{{- end }}
{{- if gt (len .Alerts.Resolved) 0 -}}
{{- range $index, $alert := .Alerts -}}
======== 告警恢复 ========
告警名称：{{ $alert.Labels.alertname }}
告警级别：{{ $alert.Labels.severity }}
告警机器：{{ $alert.Labels.instance }}
告警详情：{{ $alert.Annotations.summary }}
告警时间：{{ ($alert.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}
恢复时间：{{ ($alert.EndsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}
========== END ==========
{{- end }}
{{- end }}
{{- end }}

[root@prometheus tmpl]# cat domain.tmpl
{{ define "domain.default.message" }}
{{- if gt (len .Alerts.Firing) 0 -}}
{{- range $index, $alert := .Alerts -}}
======== 异常告警 ========
告警名称：{{ $alert.Labels.alertname }}
告警级别：{{ $alert.Labels.severity }}
告警机器：{{ $alert.Labels.instance }}
告警域名：{{ $alert.Annotations.summary }}
告警状态码：{{ $alert.Annotations.description }}
告警时间：{{ ($alert.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}
========== END ==========
{{- end }}
{{- end }}
{{- if gt (len .Alerts.Resolved) 0 -}}
{{- range $index, $alert := .Alerts -}}
======== 告警恢复 ========
告警名称：{{ $alert.Labels.alertname }}
告警级别：{{ $alert.Labels.severity }}
告警机器：{{ $alert.Labels.instance }}
告警域名：{{ $alert.Annotations.summary }}
告警状态码：{{ $alert.Annotations.description }}
告警时间：{{ ($alert.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}
恢复时间：{{ ($alert.EndsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}
========== END ==========
{{- end }}
{{- end }}
{{- end }}

```

### 4.4配置promtheus

```
vim /data/prometheus/prometheus.yml
alerting:
  alertmanagers:
  - static_configs:
    - targets: ["192.168.3.157:9093"]
```

```
mkdir /data/prometheus/rule/
[root@prometheus rule]# cat domain-rule.yml
groups:
- name: 网站状态码检测                  #报警规则组名称
  rules:                              #定义规则
  - alert: view网站状态码检测错误        #告警规则的名称    
    #基于PromQL表达式告警触发条件，用于计算是否有时间序列满足该条件。
    expr: probe_http_status_code{job="view_status"} <= 199 OR probe_http_status_code{job="view_status"} >= 400
    for: 30s             #评估等待时间，可选参数。用于表示只有当触发条件持续一段时间后才发送告警。在等待期间新产生告警的状态为pending
    labels:             #自定义标签，允许用户指定要附加到告警上的一组附加标签。
      severity: critical    #指定告警级别。有三种等级，分别为warning、critical和emergency。严重等级依次递增。
      domain: check         #自定义告警的标签，domain=check
    annotations:            #附加信息，比如用于描述告警详细信息的文字等
      summary: "{{ $labels.url }}"       #描述告警的概要信息
      description: "{{ $value }}"        #用于描述告警的详细信息。

  - alert: server网站状态码检测错误
    expr: probe_http_status_code{job="server_status"} != 404
    for: 30s
    labels:
      severity: critical
      domain: check
    annotations:
      summary: "{{ $labels.url }}"
      description: "{{ $value }}"

  - alert: https证书过期时间检测
    expr: probe_ssl_earliest_cert_expiry{url="https://ismarthealth.com"} - time() < 86400 * 30
    for: 1d
    labels:
      severity: critical
      domain: check
    annotations:
      summary: "{{ $labels.url }}域名证书还有30天过期"
      description: "{{ $value }}"


[root@prometheus rule]# cat node-rule.yml
groups:
- name: noded主机告警
  rules:
  - alert: 服务存活告警
    expr: up == 0
    for: 1m
    labels:
      severity: critical
      node: check
    annotations:
      summary: "{{ $labels.instance }} 宕机！"

  - alert: 主机内存使用率告警
    expr: (1 - (node_memory_MemAvailable_bytes / (node_memory_MemTotal_bytes))) * 100 > 85
    for: 15m
    labels:
      severity: warning
      node: check
    annotations:
      summary: "内存利用率大于85%, 实例: {{ $labels.instance }}，当前值：{{ $value }}%"

  - alert: 主机CPU使用率告警
    expr: 100 - (avg by (instance)(irate(node_cpu_seconds_total{mode="idle"}[1m]) )) * 100 > 85
    for: 15m
    labels:
      severity: warning
      node: check
    annotations:
      summary: "CPU使用率大于85%, 实例: {{ $labels.instance }}，当前值：{{ $value }}%"

  - alert: 主机磁盘使用率告警
    expr: 100 - node_filesystem_free_bytes{fstype=~"xfs|ext4"} / node_filesystem_size_bytes{fstype=~"xfs|ext4"} * 100 > 85
    for: 15m
    labels:
      severity: warning
      node: check
    annotations:
      summary: "磁盘使用率大于85%, 实例: {{ $labels.instance }}，当前值：{{ $value }}%"

```



## 5.安装grafana

### 5.1安装grafana

```
docker run -d -p 3000:3000 --name=grafana grafana/grafana
```

### 5.2添加数据源

![image-20230605165526954](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/image-20230605165526954.png)



### 5.3监控node图

11074

![image-20230605165513148](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/image-20230605165513148.png)



### 5.4监控mysql

7362

![image-20230605170057854](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/image-20230605170057854.png)



### 5.5监控http

9965

![image-20230605171600457](https://note-picture-zrh.oss-cn-beijing.aliyuncs.com/data/image-20230605171600457.png)























