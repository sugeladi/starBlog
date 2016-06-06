title: ELK 学习笔记(从零开始)
tags:
  - ELK
categories: ELK
date: 2016-04-22 17:56:08
---
- 一句话介绍
    - Logstash是一个接收，处理，转发日志的工具
    - elasticsearch是基于lucene的开源搜索引擎(实时的全文索引),一个实时分布式搜索和分析引擎
    - kibana 是根据elastic search 数据可视化，丰富功能，支持点选
- 版本号

    - elasticsearch 版本 2.3.1
    - logstash 版本 2.3.1
    - kibana 版本 4.5.0
- 文档资料
    - 官网 https://www.elastic.co
![](http://7xk2rp.com1.z0.glb.clouddn.com/Snip20160422_1.png)
<!--more-->
    - Logstash 最佳实践 http://udn.yyuap.com/doc/logstash-best-practice-cn/index.html
    - Elasticsearch 权威指南  http://www.learnes.net/index.html
    - ELKstack 中文指南 http://kibana.logstash.es/content/
- 先用apt-get安装jdk logstash需要
    - sudo apt-get update
    - sudo apt-get install openjdk-7-jdk
    - 设置java环境变量 sudo vi .bashrc 添加以下
        - export JAVA_HOME=/usr/local/java/jdk1.8.0_45
        - export PATH=$PATH:$JAVA_HOME/bin
        - exportCLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$CLASSPATH
        - source .bashrc
- 下载elasticsearch 版本 2.3.1

    - wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.3.1/elasticsearch-2.3.1.tar.gz
    - 解压 tar zxf elasticsearch-2.3.1.tar.gz -C /usr/local
    - 修改配置文件 /config/elasticsearch.yml  Network下 network.host 192.168.0.240 （测试服务器为 0.0.0.0） 也可以不设置，设置的原因是想在外网通过es的插件查看一些信息
    - bin/elasticsearch -Des.insecure.allow.root=true (解决root不能启动)
    -  启动 elasticsearch 进入 elastic search 目录 ./bin/elasticsearch -d （-d 参数 在后台运行）
    - 可以用 curl 'http://localhost:9200/_search?pretty' 来查看
- 下载logstash 版本 2.3.1

    - wget https://download.elastic.co/logstash/logstash/logstash-2.3.1.tar.gz
    - 解压 tar zxf logstash-2.3.1.tar.gz -C /usr/local
    - 执行 /usr/local/logstash-2.3.1/bin/logstash -e 'input { stdin { } } output { stdout {} }’  这是你输入什么，控制台就会有格式的显示什么
    - 使用agent -f , 编写文件 logstash-simple.conf  内容为 input { stdin { } } output { stdout {} }
    - /usr/local/logstash-2.3.1/bin/logstash agent -f logstash-simple.conf
    -  nohup ./bin/logstash -f logstash-nginx-file.conf & 在后台启动
- 下载kibana 版本 4.5.0
    - wget https://download.elastic.co/kibana/kibana/kibana-4.5.0-linux-x64.tar.gz
    - 解压 tar zxf kibana-4.5.0-linux-x64.tar.gz -C /usr/local
    - 修改配置文件 /config/kibana.yml 指定 elasticsearch.url
    - 启动 进入 kibana-4.5.0-linux-x64.tar.gz目录 ./bin/kibana 在后台运行  用 honup ./bin/kibana &
    - kibana是nodejs开发的，本身并没有任何安全限制，直接浏览url就能访问，如果公网环境非常不安全，可以通过nginx请求转发增加认证
    - kibana没有重启命令，要重启，只能ps -ef|grep node 查找nodejs进程，干掉重来。
    - 添加auth认证
        - 配置nginx配置文件 vi /etc/nginx/sites-available/default 添加 auth_basic auth_basic_user_file
            - upstream kibana {
            -     server 127.0.0.1:5601;
            - }
            - server{
            -     listen    80;
            -     server_name kibanabeta.occall.com;
            -     auth_basic "Restricted Access";
            -     auth_basic_user_file /etc/nginx/htpasswd.users;
            -     location / {
            -         proxy_pass_header Server;
            -         proxy_set_header Host $http_host;
            -         proxy_redirect off;
            -         proxy_set_header X-Real-IP $remote_addr;
            -         proxy_set_header X-Scheme $scheme;
            -         proxy_pass http://kibana;
            -     }
            - }
        - 使用apt-get install apache2-utils
        - htpasswd -c /etc/nginx/htpasswd.users kadmin 输入后，会让你输入密码
        - 重启nginx service nginx restart   
-----------
 安装结束，开始配置！
- 结合 logstash 和 elasticsearch   监控文件
    - 先启动elasticsearch
    - 编写logstash文件logstash-es-simple.conf   input { stdin { } } output { elasticsearch {hosts=> "localhost"} stdout {} }
    - 启动logstash /usr/local/logstash-2.3.1/bin/logstash agent -f logstash-es-simple.conf
    - 这是你在控制台输入的东西 会 有规则的显示在控制台 同时会写入到 elasticsearch中
    - 查看elasticsearch， curl 'http://localhost:9200/_search?pretty' 会发现刚写入的内容会在response中
    - logstash 跟踪文件 mvoice的日志 input { file { path => "/path/to/logstash-tutorial.log"  start_position => beginning   ignore_older => 0 }}
    - 跟踪mvoice的日志文件 input {  file { path => "/home/gin/nohup.out" start_position => beginning ignore_older =>0 } } output { elasticsearch {hosts=> "localhost"} stdout { codec=> rubydebug} }
    - start_position 仅在该文件从未被监听过的时候起作用。如果 sincedb 文件中已经有这个文件的 inode 记录了，那么 logstash 依然会从记录过的 pos 开始读取数据。所以重复测试的时候每回需要删除 sincedb 文件(官方博客上提供了另一个巧妙的思路：将 sincedb_path 定义为 /dev/null，则每次重启自动从头开始读)。
    - logstash 读取nginx的access.log 需要nginx的日志文件 相匹配 (这里的date解析的时候，遇到一个坑（低级错误），HH我写成了hh，所以会一直报一个警告 {:timestamp=>"2016-04-22T16:54:57.527000+0800", :message=>"Failed parsing date from field", :field=>"time_local", :value=>"22/Apr/2016:16:54:56 +0800", :exception=>"Cannot parse \"22/Apr/2016:16:54:56 +0800\": Value 16 for clockhourOfHalfday must be in the range [1,12]", :config_parsers=>"dd/MMM/yyyy:hh:mm:ss Z", :config_locale=>"en", :level=>:warn})       直接看到信息 16 for clockhourOfHalfday must be in the range [1,12]
         ```shell
         filter{
                 ruby {
                 init => "@kname = ['remote_addr','remote_user','time_local','request','status','body_bytes_sent','request_time','upstream_response_time','http_referer','http_user_agent','http_x_forwarded_for']"
                 code => "event.append(Hash[@kname.zip(event['message'].split(' | '))])"
             }
             if [request] {
                 ruby {
                     init => "@kname = ['method','uri','verb']"
                     code => "event.append(Hash[@kname.zip(event['request'].split(' '))])"
                 }
              }
              mutate {
                 convert => [
                     "body_bytes_sent" , "integer",
                     "upstream_response_time", "float",
                     "request_time", "float"
                 ]
             }
             date {
                 match => [ "time_local", "dd/MMM/yyyy:HH:mm:ss Z" ]
                 locale => "en"
             }
             geoip {
                    source => "remote_addr"
              }
         }
         ```
    - grok http://grokdebug.herokuapp.com  在线调试网址
    - gsup ：替换 支持正则，为了保证uri为统一，即 可统计数据，将bsonId 和 uId 去除，需要在gsup中添加响应的正则。
        ```shell
         mutate {
                 convert => [
                     "body_bytes_sent" , "integer",
                     "upstream_response_time", "float",
                     "request_time", "float"
                 ]
                 gsub => ["uri","[0-9a-z]{24}”,"bsonid"]
                 gsub => ["uri","\d{6,}”,"uid"]
                 gsub => ["uri","\d+x\d+","wh"]
                 gsub => ["uri","\?.+",""]
             }
         ```
- logstash 创建多个conf 并保存到多个类型index
    - logstash 还提供一个方便我们规划和书写配置的小功能。你可以直接用 bin/logstash -f /etc/logstash.d/ 来运行。logstash 会自动读取 /etc/logstash.d/ 目录下所有 *.conf 的文本文件，然后在自己内存里拼接成一个完整的大配置文件，再去执行。
    - Logstash 在有多个 conf 文件的情况下，进入 ES 的数据会重复，几个 conf 数据就会重复几次。其实问题原因在之前启动参数章节有提过，output 段顺序执行，没有对日志 type 进行判断的各插件配置都会全部执行一次。在 output 段对 type 进行判断的语法如下所示:
        ```shell
         output {
            if [type] == "nginxaccess" {
              elasticsearch { }
            }
        }
    ```

- 安装elasticsearch 插件 Elasticsearch-kopf 插件可以查询 Elasticsearch 中的数据

    - 在elasticsearch目录下 ./bin/plugin install lmenezes/elasticsearch-kopf
    - 安装完成后在 plugins 目录下可以看到 kopf
    - 访问 http://localhost:9200/_plugin/kopf 需要开启端口号
    - 修改配置文件 /config/elasticsearch.yml 其中 network.host 启动的IP(参考 4.3)
    - 修改配置文件 /config/elasticsearch.yml 开启script 脚本(这样就可以愉快的使用 update += 了 ) 即 8.11

- elastic 通过 HTTP 向 RESTful API 传送 json
    - 相应的 HTTP 请求方法 或者 变量 : GET, POST, PUT, HEAD 或者 DELETE
    - 创建一个index  curl -XPUT 'localhost:9200/customer?pretty'
    - 查看所有index  curl 'localhost:9200/_cat/indices?v'
    - 向type中插入一条数据 curl -XPUT elk.occall:9200/hxindex/user/2 -d '{"name":"hx02","age":26}'
    - 向type中插入一条数据 不指定id  curl -XPOST elk.occall:9200/hxindex/user -d '{"name":"hx03","age":26}'
    - 查询数据 curl -XGET elk.occall:9200/hxindex/user/1?pretty
    - 删除文档一条数据 curl -XDELETE elk.occall:9200/hxindex/user/1?pretty
    - 修改数据 同插入，如果有就是修改，没有就是插入
    - 修改某个字段 curl -XPOST elk.occall:9200/hxindex/user/2/_update?pretty -d '{"doc":{"name":"hxx02"}}'
    - 添加一个字段 curl -XPOST elk.occall:9200/hxindex/user/2/_update?pretty -d '{"doc":{"name":”hxx02”,”address":"BJ"}}'
    - 使用脚本 curl -XPOST elk.occall:9200/hxindex/user/2/_update?pretty -d '{"script”:{"age":"cx._source.age+=5"}}'
- 额外的话

    - LogStash 的性能，是最让新人迷惑的地方。因为 LogStash 本身并不维护队列，所以整个日志流转中任意环节的问题，都可能看起来像是 LogStash 的问题。这里，需要熟练使用本节说的测试方法，针对自己的每一段配置，都确定其性能。另一方面，就是本书之前提到过的，LogStash 给自己的线程都设置了单独的线程名称，你可以在 top -H 结果中查看具体线程的负载情况。

    - Elasticsearch 的性能。这里最需要强调的是：Elasticsearch 是一个分布式系统。从来没有分布式系统要跟人比较单机处理能力的说法。所以，更需要关注的是：在确定的单机处理能力的前提下，性能是否能做到线性扩展。当然，这不意味着说提高处理能力只能靠加机器了——有效利用 mapping API 是非常重要的。不过暂时就在这里讲述了。

    - Kibana 的性能。通常来说，Kibana 只是一个单页 Web 应用，只需要 nginx 发布静态文件即可，没什么性能问题。页面加载缓慢，基本上是因为 Elasticsearch 的请求响应时间本身不够快导致的。不过一定要细究的话，也能找出点 Kibana 本身性能相关的话题：因为 Kibana3 默认是连接固定的一个 ES 节点的 IP 端口的，所以这里会涉及一个浏览器的同一 IP 并发连接数的限制。其次，就是 Kibana 用的 AngularJS 使用了 Promise.then 的方式来处理 HTTP 请求响应。这是异步的。