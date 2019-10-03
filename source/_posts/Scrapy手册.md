---
title: Scrapy手册
date: 2019-09-24 16:17:56
tags:
categories:
- 手册
---

### 基本命令

- genspider

  使用预定义的模板生成一个spider类

  ```
  Usage
  =====
    scrapy genspider [options] <name> <domain>
  
  Generate new spider using pre-defined templates
  
  Options
  =======
  --help, -h              show this help message and exit 
  --list, -l              List available templates  列出可用的模板
  --edit, -e              Edit spider after creating it	创建spider后编辑它
  --dump=TEMPLATE, -d TEMPLATE
                          Dump template to standard output		输出模板到标准输出
  --template=TEMPLATE, -t TEMPLATE
                          Uses a custom template.
  --force                 If the spider already exists, overwrite it with the	
                          template 如果spider存在,就强制使用模板覆盖
  
  Global Options	全局选项A
  --------------
  --logfile=FILE          log file. if omitted stderr will be used	日志文件
  --loglevel=LEVEL, -L LEVEL
                          log level (default: DEBUG)				日志登记
  --nolog                 disable logging completely
  --profile=FILE          write python cProfile stats to FILE	
  --pidfile=FILE          write process ID to FILE
  --set=NAME=VALUE, -s NAME=VALUE
                          set/override setting (may be repeated)		覆盖setting的值
  --pdb                   enable pdb on failure		失败时开启pdb
  ```

  内置的模板有

    basic
    crawl
    csvfeed
    xmlfeed

  这几个

- 

### 常用操作

- 在IDE里设置response的类型,来代码提示

  回调函数里的ide识别不了类型，手动指定类型即可

  ```python
  scrapy.http.response.html.HtmlResponse
  from scrapy.http.response.html import HtmlResponse
  
  ```

  

- 设置运行日志输出等级

  调试完后正式运行，不想输出太多日志

### 使用示例

1. 创建工程

2. 关闭机器人协议

3. 生成spider,生成一个叫wallpaper的spider,域名为`wall.alphacoders.com`

   ``