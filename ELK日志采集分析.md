# ELK日志监控

EC2实例选项

- Amazon系统映像：Ubuntu Server 14.04 LTS 64位
- 实例类型：t2.medium (变量 ECU, 2 vCPU, 2.5 GHz, Intel Xeon Family, 4 GiB 内存, 仅限于 EBS)

## 系统配置

系统更新与升级
```
sudo apt-get update
sudo apt-get upgrade
```

时间同步服务
```
sudo apt-get install ntp # 安装 ntp
sudo service ntp status
```

如果与外部时间相差太大，可能同步不能正常进行。先停止服务，手动同步。
```
sudo service ntp stop
sudo ntpdate ntp.ubuntu.com
sudo service ntp start
```

添加部署用户
```
sudo adduser elk # password elk263
sudo gpasswd -a elk admin # 添加为管理员
```

然后以root身份继续

```
ssh-keygen
```

设置时区
```
# 添加加到.profile
TZ='Asia/Shanghai'; export TZ
```

设置语言  
```
安装中文包  
sudo apt-get -y install language-pack-zh-hans  

# 添加到.profile
export LANG=zh_CN.utf8

export LANGUAGE=zh_CN.utf8:zh_CN.utf8

export LC_ALL=zh_CN.utf8
```

## JDK安装配置
ES依赖与JDK要先安装JDK，推荐Debian安装
```
sudo add-apt-repository ppa:webupd8team/java

sudo apt-get update

sudo apt-get install oracle-java8-installer

java -version
```

## Elasticsearch安装及配置
提供Tar包和DEB等安装方式，我们选择DEB安装，安装版本2.3.3
```
# 下载
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/deb/elasticsearch/2.3.3/elasticsearch-2.3.3.deb

# 安装
sudo dpkg -i elasticsearch-2.3.3.deb

# 使用
sudo service elasticsearch [start|stop|restart|force-reload|status]

# 安装后目录
安装包: /usr/share/elasticsearch
配置: /etc/elasticsearch/

# 配置
1.远程访问：
ES默认开启9200和9300端口，9200提供api访问，但默认只能本地打开，要远程访问需要修改配置
打开 /etc/elasticsearch/elasticsearch.yml
network.host: 0.0.0.0

# ES监控插件
官方使用的是Marvel但是只能试用，开源的使用较多的是Kopf(https://github.com/lmenezes/elasticsearch-kopf)
cd /usr/share/elasticsearch/bin/
sudo ./plugin install lmenezes/elasticsearch-kopf
访问 http://112.124.46.102/:9200/_plugin/kopf/
```
参考文件
https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-elasticsearch-on-ubuntu-14-04

## Kibana安装及配置
安装版本4.5.1
```
# 下载
wget https://download.elastic.co/kibana/kibana/kibana_4.5.1_amd64.deb

# 安装
sudo dpkg -i kibana_4.5.1_amd64.deb
安装后默认在/opt/kibana目录下

# 使用
sudo service kibana [start|force-start|stop|force-start|force-stop|status|restart]
Kibana默认使用5601端口，访问 http://112.124.46.102/:5601/ 即可查看
```

## Nginx安装与配置  
Kibana没有认证，需要通过增加nginx转发来认证
```
sudo add-apt-repository ppa:nginx/stable
sudo apt-get update
sudo apt-get install nginx

# 安装密码生成htpasswd，htpassws是apache自带的小工具
sudo apt-get install apache2-utils

# Kibana配置
sudo htpasswd -c /etc/nginx/kibana_passwd.db kibana
kibana是用户名，执行后系统会让输入密码
Auth: kibana/kibana263

sudo touch /etc/nginx/sites-enabled/kibana
server {
  listen       80;
  server_name  112.124.46.102;
  location / {
     auth_basic "secret";
     auth_basic_user_file /etc/nginx/kibana_passwd.db;
     proxy_pass http://localhost:5601;
     proxy_set_header Host $host:5601;
     proxy_set_header X-Real-IP $remote_addr;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header Via "nginx";
  }
  access_log off;
}
Kibana访问: http://112.124.46.102/  Auth: kibana/kibana263

# ES配置
sudo htpasswd -c /etc/nginx/es_passwd.db es
es是用户名，执行后系统会让输入密码
Auth: es/es263

sudo touch /etc/nginx/sites-enabled/es
server {
  listen       9292;
  server_name  112.124.46.102;
  location / {
     auth_basic "secret";
     auth_basic_user_file /etc/nginx/es_passwd.db;
     proxy_pass http://localhost:9200;
     proxy_set_header Host $host:9200;
     proxy_set_header X-Real-IP $remote_addr;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header Via "nginx";
  }
  access_log off;
}
ES访问: http://112.124.46.102:9292/_plugin/kopf/  Auth: es/es263

# 重启nginx
sudo service nginx restart
```

## Logstash安装及配置
logstash也需要安装jdk安装方法同上

```
# 安装
logstash是日志的collector，安装在需要收集日志的服务器上，选择DEB包安装，版本2.3.2
wget https://download.elastic.co/logstash/logstash/packages/debian/logstash_2.3.2-1_all.deb
sudo dpkg -i logstash_2.3.2-1_all.deb
安装包在/opt/logstash
默认配置文件在/etc/logstash/conf.d

# 使用
sudo service logstash start|stop|force-stop|status|reload|restart|configtest
```

####  Logstash配置
```
input {
  file {
    type => "nginx_access"
    path => ["/var/log/nginx/access.log"]
    add_field => {"server" => "98"}
    close_older => 10
  }
  file {
    type => "iad_event"
    path => ["/opt/rails-apps/blog/shared/log/events.log"]
    add_field => {"server" => "blog98"}
    close_older => 10
    codec => multiline {
      pattern => "^D,"
      negate => true
      what => "previous"
    }
  }
  file {
    type => "event_process"
    add_field => {"server" => "blog98"}
    path => ["/opt/rails-apps/blog/shared/log/events-process.log"]
    close_older => 10
    codec => 'json'
  }
}
filter {
  if [type] == "nginx_access" {
    grok {
      match => [ "message" , "%{COMBINEDAPACHELOG}+%{GREEDYDATA:extra_fields}"]
      overwrite => [ "message" ]
    }

    mutate {
      convert => ["response", "integer"]
      convert => ["bytes", "integer"]
      convert => ["responsetime", "float"]
    }

    date {
      match => [ "timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
      remove_field => [ "timestamp" ]
    }

    geoip {
      source => "clientip"
      target => "geoip"
      add_tag => [ "nginx-geoip" ]
    }
  }
  if [type] == "iad_event" {
    mutate {
      gsub => ["message", "\n|\s", " "]
    }
    grok {
      match => ["message", "., \[%{TIMESTAMP_ISO8601:timestamp} #%{POSINT}\] %{LOGLEVEL} -- : From %{IP:remote}:%{SPACE}<%{DATA}>%{SPACE}<%{DATA:category} (id|attribute)=\"(%{DATA:event})\">%{SPACE}<MAC>%{MAC:mac}</MAC>"]
    }

    mutate {
      gsub => ["mac", ":", '']
    }

    date {
      match => ["timestamp", "yyyy-MM-dd HH:mm:ssZ"]
      remove_field => ["timestamp"]
    }
  }
  if [type] == "event_process" {
    date {
      match => [ "timestamp" , "ISO8601" ]
      remove_field => [ "timestamp" ]
    }
  }
}
output {
  elasticsearch {
    hosts => ["172.31.0.233:9292"]
    user => "es"
    password => "es263"
  }
}
```

