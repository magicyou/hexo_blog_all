---
title: scrapy爬虫之初次尝试
date: 2018-10-10 21:34:23
updated: 2018-10-10 21:34:23
tags: [Python,scrapy]
categories: ['Python']
---
本文主要记录scrapy安装、配置，到第一个爬虫实例实现的过程。

<!--more-->

### 准备工作

#### 环境
* 环境：macOS
* python版本：python 3.7
* Scrapy版本：1.5.1

#### 安装scrapy
```
pip3 install scrapy
```

#### 创建Scrapy项目
创建python目录，切换到python目录,创建爬虫项目
```
magicyou@magicYoudeMacBook-Pro ~$ mkdir ~/python
magicyou@magicYoudeMacBook-Pro ~$ cd ~/python
magicyou@magicYoudeMacBook-Pro python$ scrapy startproject articleSpider
New Scrapy project 'articleSpider', using template directory '/usr/local/lib/python3.7/site-packages/scrapy/templates/project', created in:
    /Users/magicyou/Desktop/python/articleSpider

You can start your first spider with:
    cd articleSpider
    scrapy genspider example example.com

```
创建一个Spider爬虫程序。本文进行抓取的模板网站 'blog.jobbole.com'，

```
magicyou@magicYoudeMacBook-Pro python$ cd articleSpider
magicyou@magicYoudeMacBook-Pro articleSpider$ scrapy genspider jobbole blog.jobbole.com
Created spider 'jobbole' using template 'basic' in module:
  articleSpider.spiders.jobbole
```

#### 创建一个mysql表，用来存储文章信息
直接上sql
```sql
DROP TABLE IF EXISTS `article`;
CREATE TABLE `article` (
  `url_object_id` varchar(255) COLLATE utf8_unicode_ci NOT NULL COMMENT 'url的md5',
  `url` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT 'url',
  `title` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '文章名字',
  `create_date` date DEFAULT NULL COMMENT '创建日期',
  `front_image_url` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '封面路原始url',
  `front_image_path` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '封面图本地路径',
  `comment_nums` int(11) DEFAULT NULL COMMENT '评论数',
  `fav_nums` int(11) DEFAULT NULL COMMENT '点赞数',
  `praise_nums` int(11) DEFAULT NULL,
  `content` longtext COLLATE utf8_unicode_ci COMMENT '文章内容',
  PRIMARY KEY (`url_object_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

SET FOREIGN_KEY_CHECKS = 1;
```
至此准备工作基本完成
此时的目录结构应该是如下展示：
```
|-- articleSpider
    |--- articleSpider
        |-- spiders    # Spiders类
        	|-- jobbole.py
        	|-- __init__.py
        	|-- __pycache__
            	|-- __init__.cpython-37.pyc
        |-- __pycache__
            |-- settings.cpython-37.pyc
            |-- __init__.cpython-37.pyc
        |-- middlewares.
        |-- pipelines.py
        |-- items.py
        |-- settings.py
        |-- __init__.py
    |-- scrapy.cfg

```

### 简单的了解

#### Spiders
Spider类定义了如何爬取某个(或某些)网站。包括了爬取的动作(例如:是否跟进链接)以及如何从网页的内容中提取结构化数据(爬取item)。 换句话说，Spider就是您定义爬取的动作及分析某个网页(或者是有些网页)的地方。

对spider来说，爬取的循环类似下文:

* 以初始的URL初始化Request，并设置回调函数。 当该request下载完毕并返回时，将生成response，并作为参数传给该回调函数。

  spider中初始的request是通过调用 start_requests() 来获取的。 start_requests() 读取 start_urls 中的URL， 并以 parse 为回调函数生成 Request 。

* 在回调函数内分析返回的(网页)内容，返回 Item 对象或者 Request 或者一个包括二者的可迭代容器。 返回的Request对象之后会经过Scrapy处理，下载相应的内容，并调用设置的callback函数(函数可相同)。

* 在回调函数内，您可以使用 选择器(Selectors) (您也可以使用BeautifulSoup, lxml 或者您想用的任何解析器) 来分析网页内容，并根据分析的数据生成item。

* 最后，由spider返回的item将被存到数据库(由某些 Item Pipeline 处理)或使用 Feed exports 

#### Items
爬取的主要目标就是从非结构性的数据源提取结构性数据，例如网页。 Scrapy提供 Item 类来满足这样的需求。
Item 对象是种简单的容器，保存了爬取到得数据。 其提供了 类似于词典(dictionary-like) 的API以及用于声明可用字段的简单语法。

#### Item Pipeline
当Item在Spider中被收集之后，它将会被传递到Item Pipeline，一些组件会按照一定的顺序执行对Item的处理。

每个item pipeline组件(有时称之为“Item Pipeline”)是实现了简单方法的Python类。他们接收到Item并通过它执行一些行为，同时也决定此Item是否继续通过pipeline，或是被丢弃而不再进行处理。

以下是item pipeline的一些典型应用：

* 清理HTML数据
* 验证爬取的数据(检查item包含某些字段)
* 查重(并丢弃)
* 将爬取结果保存到数据库中

### 爬虫逻辑编写

入口文件 main.py
编辑器是PyCharm，为了调试方便，添加main.py作为程序的入口，切换在articleSpider操作(当前路径 ~/python/articleSpider/articleSpider)

```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
__author__ = 'magicYou'

from scrapy.cmdline import execute
import sys
import os

print(os.path.dirname(os.path.abspath(__file__)))
sys.path.append(os.path.dirname(os.path.abspath(__file__)))
execute(["scrapy", "crawl", "jobbole"])
```

Spider爬虫使用parse(self,response)方法来解析所下载的页面。此方法返回一个包含新的URL资源网址的迭代对象，这些新的URL网址将被添加到下载队列中以供将来进行爬取数据和解析。
spider主要代码（python/articleSpider/articleSpider/jobbole.py）：
```python
# -*- coding: utf-8 -*-
from urllib import parse
from datetime import datetime
from scrapy.http import Request
from articleSpider.items import JobboleArticlespiderItem
from .utils.common import get_md5

class JobboleSpider(scrapy.Spider):
    name = 'jobbole'
    allowed_domains = ['blog.jobbole.com']
    start_urls = ['http://blog.jobbole.com/all-posts/']

    def parse(self, response):
        '''
        1.获取文章列表中的文章url并交给scrapy下载并进行解析
        2.获取下一页的url并交给scary进行下载，下载完成后交给parse
        '''
        post_nodes = response.css("#archive .floated-thumb .post-thumb a")
        for post_node in post_nodes:
            image_url = post_node.css("img::attr(src)").extract_first("")
            post_url = post_node.css("::attr(href)").extract_first()
            yield Request(url=parse.urljoin(response.url, post_url), meta={"front_image_url":image_url}, callback=self.parse_detail)

        # 提取下一页并交给scrapy进行下载
        next_urls = response.css(".next.page-numbers::attr(href)").extract_first("")
        if next_urls:
            yield Request(url=parse.urljoin(response.url, next_urls), callback=self.parse)

    def parse_detail(self, response):
        '''
        详情页信息提取
        '''
        article = JobboleArticlespiderItem()
        url = response.url
        title = response.xpath("//div[@class='entry-header']/h1/text()").extract()[0]
        create_date = response.xpath("//div[@class='entry-meta']/p[@class='entry-meta-hide-on-mobile']/text()").extract()[0].strip()
        create_date = datetime.strptime(create_date.split()[0], "%Y/%m/%d")
        praise_nums = response.xpath("//div[@class='post-adds']/span[contains(@class, 'vote-post-up')]/h10/text()").extract()[0]
        fav_nums = response.xpath("//div[@class='post-adds']/span[contains(@class, 'bookmark-btn')]/text()").extract()[0]
        fav_nums = fav_nums.replace('收藏', '') if fav_nums.replace('收藏', '').strip() else 0
        comment_nums = response.xpath("//div[@class='post-adds']/a[@href='#article-comment']/span/text()").extract()[0]
        comment_nums = comment_nums.replace('评论', '') if comment_nums.replace('评论', '').strip() else 0
        content = response.xpath("//div[@class='entry']").extract()[0]

        # 获取图片
        front_image_url = response.meta.get("front_image_url", "")
        article_item = {}
        article_item["url"] = url
        article_item["url_object_id"] = get_md5(url)
        article_item["title"] = title
        article_item["create_date"] = create_date
        article_item["praise_nums"] = int(praise_nums)
        article_item["fav_nums"] = int(fav_nums)
        article_item["comment_nums"] = int(comment_nums)
        article_item["content"] = content
        article_item["front_image_url"] = [front_image_url]
        
        yield article_item
```
过程简单描述
1. parse函数从start_urls为起始页，获取列表里的文章url，和文章的封面图url；
2. 讲步骤1中的获取的url交给parse_detail，进行详情处理，处理结果交付给item；
3. 获取下一页的url，作为参数再次交给parse函数，直到没有下一页为止。


Item使用简单的class定义语法以及 Field 对象来声明
items主要代码（python/articleSpider/items.py）：
```
import scrapy

class ArticleItem(scrapy.Item):
    title = scrapy.Field()
    url = scrapy.Field()
    url_object_id = scrapy.Field()
    crate_date = scrapy.Field()
    praise_nums = scrapy.Field()
    fav_nums = scrapy.Field()
    comment_nums = scrapy.Field()
    content = scrapy.Field()
    front_image_url = scrapy.Field()
```
pipeline中接收item传过来的数据，并且存入数据库

```python
from scrapy.pipelines.images import ImagesPipeline
import pymysql

from twisted.enterprise import adbapi

class ArticleImagePipeline(ImagesPipeline):
    '''
    存储图片，并返回存储的路径
    '''
    def item_completed(self, results, item, info):
        for ok, value in results:
            image_file_path = value['path']
        # 给item添加图片本地存储路径
        item["front_image_path"] = image_file_path
        return item

class MysqlPipeline():
    '''
    同步存储mysql，比较慢
    '''
    def __init__(self, params):
        self.conn = pymysql.connect( host = params['host'],
                                     user = params['user'],
                                     password = params['password'],
                                     db = params['db'],
                                     charset = params['charset'],
                                     cursorclass = pymysql.cursors.DictCursor)
        self.cursor = self.conn.cursor()

    @classmethod
    def from_crawler(cls, crawler):
        '''
        获取settings文件中的配置
        '''
        params = crawler.settings.get('MYSQL')
        return cls(params)

    def process_item(self, item, spider):
        insert_sql = '''
            insert into article 
               (title, create_date, url, url_object_id, front_image_url, front_image_path, comment_nums, fav_nums, praise_nums, content)
            values 
                ('%s', '%s','%s', '%s', '%s', '%s', %d, %d, %d,'%s')
        ''' % (item['title'], item['create_date'], item['url'], item['url_object_id'], ','.join(item['front_image_url']), item["front_image_path"], item['comment_nums'], item['fav_nums'], item['praise_nums'], item['content'])
        self.cursor.execute(insert_sql)
        self.conn.commit()
        return item

class MysqlTwistedPipline(object):
    '''
    异步存储mysql，相对较快
    '''
    def __init__(self, params):
        # 使用Twisted中的adbapi获取数据库连接池对象
        self.dbpool = adbapi.ConnectionPool("pymysql", **params)

    @classmethod
    def from_crawler(cls,crawler):
        '''
        获取settings文件中的配置
        '''
        param = crawler.settings.get('MYSQL')
        params = dict(
            host = param['host'],
            user = param['user'],
            password = param['password'],
            db = param['db'],
            charset = param['charset'],
            cursorclass = pymysql.cursors.DictCursor,
        )
        return cls(params)

    def process_item(self, item, spider):
        # 使用数据库连接池对象进行数据库操作,自动传递cursor对象到第一个参数
        query = self.dbpool.runInteraction(self.do_insert, item)
        # 设置出错时的回调方法,自动传递出错消息对象failure到第一个参数
        query.addErrback(self.handle_error)

    def handle_error(self,failure):
        print(failure)

    def do_insert(self, cursor, item):
        insert_sql = '''
            insert into article 
               (title, create_date, url, url_object_id, front_image_url, front_image_path, comment_nums, fav_nums, praise_nums, content)
            values 
                ('%s', '%s','%s', '%s', '%s', '%s', %d, %d, %d,'%s')
        ''' % (item['title'], item['create_date'], item['url'], item['url_object_id'], ','.join(item['front_image_url']), item["front_image_path"], item['comment_nums'], item['fav_nums'], item['praise_nums'], item['content'])
        cursor.execute(insert_sql)
```
最重要的一步就是settings需要配置
关键的settings配置：
```python
import os
# Pipeline自定义一个，就要在这里添加一个，后面的数字表示执行的先后顺序，数字越大越靠后
ITEM_PIPELINES = {
   # 'articleSpider.pipelines.ArticlespiderPipeline': 300,
    'scrapy.pipelines.images.ImagesPipeline': 2,
    'articleSpider.pipelines.ArticleImagePipeline': 1,
    'articleSpider.pipelines.MysqlPipeline': 3,
   #  'articleSpider.pipelines.MysqlTwistedPipline': 3,
}

# 图片下载
# item中要下载的图片路径，数值类型必须为list
IMAGES_URLS_FIELD = "front_image_url"
# 获取当前绝对路径
project_dir = os.path.abspath(os.path.dirname(__file__))
# 图片存储到本地路径
IMAGES_STORE = os.path.join(project_dir, 'images')

# mysql配置参数
MYSQL = {'host':'localhost','user':'root','password':'root','db':'jobbole','charset':'utf8'}

```
### 注意
1. spiders目录下添加utils目录，存放常用的自定义公共方法common.py
```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
__author__ = 'magicYou'

import hashlib

def get_md5(url):
    if isinstance(url, str):
        url = url.encode("utf-8")
    m = hashlib.md5()
    m.update(url)
    return m.hexdigest()
```
2. articleSpider目录下创建images，用来存储下载的图片

### 知识补充
xpath基本语法

|表达式 | 说明 |
| - | - |
|/body |  选出当前选择器的根元素body |
|/body/div |  选取当前选择器文档的根元素body的所有div子元素 |
|/body/div[1] |   选取body根元素下面第一个div子元素 |
|/body/div[last()]  | 选取body根元素下面最后一个div子元素 |
|/body/div[last()-1] | 选取body根元素下面倒数第二个div子元素 |
|//div  | 选取所有div子元素(不论出现在文档任何地方) |
|body//div |  选取所有属于body元素的后代的div元素(不论出现在body下的任何地方) |
|/body/@id  | 选取当前选择器文档的根元素body的id属性 |
|//@class   | 选取所有元素的class属性 |
|//div[@class]  | 选取所有拥有class属性的div元素 |
|//div[@class='bold']  |  选取所有class属性等于bold的div元素 |
|//div[contains(@class,'bold')] | 选取所有class属性包含bold的div元素 |
|/div/* | 选取当前文档根元素div的所有子元素 |
|//* | 选取文档所有节点 |
|//div[@*] |  获取所有带属性的div元素 |
|//div/a \| //div/p  | 选取所有div元素下面的子元素a和子元素p(并集) |
|//p[@id='content']/text() |  选取id为content的p标签的内容(子元素的标签和内容都不会获取到) |

一个用法实例：
```
txt_header = response.xpath("//div[@id='wrapper_header']/h3/text()").extract()[0] # 选取wrapper_header下h3标签的内容
```

CSS选择器

|表达式  | 说明 |
| - | - |
| *  | 选择所有节点 |
| #container | 选择Id为container的节点 |
| .container   | 选取所有包含container类的节点 |
| li a     | 选取所有li下的所有后代a元素(子和孙等所有的都会选中) |
| ul + p   | 选取ul后面的第一个相邻兄弟p元素 |
| div#container > ul  |  选取id为container的div的所有ul子元素 |
| ul ~ p   | 选取与ul元素后面的所有兄弟p元素 |
| a[title]   |   选取所有有title属性的a元素 |
| a[href='http://taobao.com']  | 选取所有href属性等于http://taobao.com的a元素 |
| a[href*='taobao']   |  选取所有href属性包含taobao的a元素 |
| a[href^='http']  | 选取所有href属性开头为http的a元素 |
| a[href$='.com']  | 选取所有href属性结尾为.com的a元素 |
| input[type=radio]:checked    | 选取选中的radio的input元素 |
| div:not(#container)  | 选取所有id非container的div元素 |
| li:nth-child(3)  | 选取第三个li元素 |
| tr:nth-child(2n)    |  选取偶数位的tr元素 |
| a::attr(href)   |  获取所有a元素的href属性值 |

一个用法实例：
```
txt_header = response.css("#wrapper_header h3::text").extract_first()  # 选取wrapper_header下h3标签的内容
```

### 参考资料、视频
1. [scrapy中文文档](https://scrapy-chs.readthedocs.io/zh_CN/0.24/)
2. [Python爬虫框架Scrapy学习笔记原创（来自CSDN）](https://blog.csdn.net/wxystyle/article/details/81128026)
3. [Python分布式爬虫打造搜索引擎（慕课视频）](https://coding.imooc.com/class/92.html)

