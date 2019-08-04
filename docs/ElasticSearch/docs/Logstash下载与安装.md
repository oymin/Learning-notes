# Logstash 下载与安装

Logstash 是 ES 下的一款开源软件，它能够同时 从多个来源采集数据、转换数据，然后将数据发送到 Eleasticsearch 中创建索引。

本项目使用 Logstash 将 MySQL 中的数据采用到 ES 索引中。

## 1. 下载 Logstash

下载 Logstash6.4.1 版本，和本项目使用的 Elasticsearch6.4.1 版本一致。

下载地址：[https://artifacts.elastic.co/downloads/logstash/logstash-6.4.1.zip](https://artifacts.elastic.co/downloads/logstash/logstash-6.4.1.zip)

## 2. 安装 logstash-input-jdbc 插件

### 2.1 下载 ruby 并安装

下载地址：[https://github.com/oneclick/rubyinstaller2/releases/download/RubyInstaller-2.5.5-1/rubyinstaller-devkit-2.5.5-1-x64.exe](https://github.com/oneclick/rubyinstaller2/releases/download/RubyInstaller-2.5.5-1/rubyinstaller-devkit-2.5.5-1-x64.exe)

安装完成查看是否安装成功：

```bash
ruby -v

ruby 2.5.5p157 (2019-03-15 revision 67260) [x64-mingw32]
```

安装完成后可在 logstash 目录下查看对应的插件版本：

`logstash-6.4.1\vendor\bundle\jruby\2.3.0\gems\logstash-input-jdbc-4.3.13`

## 3. 创建模板文件

Logstash 的工作是从 MySQL 中读取数据，向 ES 中创建索引，这里需要提前创建 mapping 的模板文件以便 logstash 使用。

在 logstach 的 config 目录创建 xc_course_template.json，内容如下：

本教程的 xc_course_template.json 目录是：`D:/ElasticSearch/logstash-6.4.1/config/xc_course_template.json`

```json
{
	"mappings": {

		"doc": {
			"properties": {
				"qq": {
					"index": false,
					"type": "keyword"
				},
				"price_old": {
					"type": "double"
				},
				"st": {
					"type": "keyword"
				},
				"expires": {
					"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis",
					"type": "date"
				},
				"charge": {
					"type": "keyword"
				},
				"mt": {
					"type": "keyword"
				},
				"pub_time": {
					"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis",
					"type": "date"
				},
				"studymodel": {
					"type": "keyword"
				},
				"end_time": {
					"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis",
					"type": "date"
				},
				"description": {
					"search_analyzer": "ik_smart",
					"analyzer": "ik_max_word",
					"type": "text"
				},
				"teachmode": {
					"type": "keyword"
				},
				"pic": {
					"index": false,
					"type": "keyword"
				},
				"teachplan": {
					"search_analyzer": "ik_smart",
					"analyzer": "ik_max_word",
					"type": "text"
				},
				"users": {
					"index": false,
					"type": "text"
				},
				"valid": {
					"type": "keyword"
				},
				"start_time": {
					"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis",
					"type": "date"
				},
				"price": {
					"type": "double"
				},
				"grade": {
					"type": "keyword"
				},
				"name": {
					"search_analyzer": "ik_smart",
					"analyzer": "ik_max_word",
					"type": "text"
				},
				"id": {
					"type": "keyword"
				},
				"status": {
					"type": "keyword"
				}
			}
		}
	},
	"template" : "xc_course"
}
```

## 4. 配置 mysql.conf 文件

在 logstash 的 config 目录下配置 mysql.conf 文件供 logstash 使用，logstash 会根据 mysql.conf 文件的配置的地址从 MySQL 中读取数据向 ES 中写入索引。

参考[https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html)

配置输入数据源和输出数据源：

```yml
input {
  stdin {
  }
  jdbc {
	  jdbc_connection_string => "jdbc:mysql://localhost:3306/xc_course?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC"

	  # the user we wish to excute our statement as
	  jdbc_user => "root"
	  jdbc_password => root

	  # the path to our downloaded jdbc driver  
	  jdbc_driver_library => "D:/Maven3/repository/mysql/mysql-connector-java/5.1.37/mysql-connector-java-5.1.37.jar"

	  # the name of the driver class for mysql
	  jdbc_driver_class => "com.mysql.jdbc.Driver"
	  jdbc_paging_enabled => "true"
	  jdbc_page_size => "50000"

	  #要执行的sql文件
	  #statement_filepath => "/conf/course.sql"
	  statement => "select * from course_pub where timestamp > date_add(:sql_last_value,INTERVAL 8 HOUR)"

	  #定时配置
	  schedule => "* * * * *"
	  record_last_run => true
	  last_run_metadata_path => "D:/Develop2/logstash-6.4.1/config/logstash_metadata"
  }
}


output {
  elasticsearch {
	  #ES的ip地址和端口
	  hosts => "localhost:9200"
	  #hosts => ["localhost:9200","localhost:9202","localhost:9203"]

	  #ES索引库名称
	  index => "xc_course"
	  document_id => "%{id}"
	  document_type => "doc"
	  template =>"D:/Develop2/logstash-6.4.1/config/xc_course_template.json"
	  template_name =>"xc_course"
	  template_overwrite =>"true"
  }
  stdout {
	#日志输出
	codec => json_lines
  }
}
```

说明：

1、ES 采用 UTC 时区问题

ES 采用 UTC 时区，比北京时间早8小时，所以 ES 读取数据时让最后更新时间加8小时

```sql
where timestamp > date_add(:sql_last_value,INTERVAL 8 HOUR)
```

2、logstash 每个执行完成会在 `D:/ElasticSearch/logstash-6.2.1/config/logstash_metadata` 记录执行时间下次以此时间为基准进行增量同步数据到索引库。

## 5. 启动

在 logstash 根目录下执行：

```bash
.\logstash.bat ‐f ..\config\mysql.conf
```
