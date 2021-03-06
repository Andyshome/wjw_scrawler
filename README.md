# wjw_scraler

## 总述

本程序用于爬取各省(或许会增加到市)卫健委关于疫情的新闻报道。

## 工作方法

程序规定了一个Manger-Crawler的管理架构，在一个Crawler实例完成对一个url的爬取，Manger统一管理所有crawler，用协程的方法定时爬取（目前采用一分钟一次的爬取频率），每次爬取最新的新闻，如果与记录中的最新新闻不同且标题有关键词，则进行汇报。

## 爬虫配置

为了便于编写对不同的url的爬取，我采用外部json配置的方法进行管理，在`config`文件夹中有目前已经完成的`<地名>.json`文件，格式样例如下：

```json
{
    "name": "shanxi",
    "Chinese_name": "山西",
    "url": "http://wjw.shanxi.gov.cn/",
    "parser_name": "title_in_attr_href_in_attr",
    "search_path": [
        {
            "name": "div",
            "class_": "news_box"
        },
        {
            "name": "li"
        },
        {
            "name": "a"
        }
    ]
}
```

其中`name`是地名的官方英文(实际上大多是拼音，目前用于log输出)，`Chinese_name`是中文地名(用于发邮件)，`url`是爬取目标的网址，`parser_name`是从目标结点解析出新闻标题与链接的函数名(该部分函数统一编写、管理)，`search_path`是一个从html解析树根部向下找到最新新闻标题的路径(实际路径往往可以压缩到2～3段，设置路径时请自行用BeautifulSoup进行验证)。

目前需要将各个网站添加到解析范围中，主要是添加本部分内容。

## 解析函数

`parser`模块中给出了若干解析函数，由于此时已经解析到一个具体的结点，实际上获取数据应该只需要从该结点标签、文本中获取，目前给出了三个函数，如果遇到不能cover的情况再行添加(如专门添加的`chongqing_parser`)，万一有需要的话，对于某些特殊网站可以考虑直接把所有数据传到专用解析函数里进行针对性解析。

## 汇报方法

可以自行实现汇报方法，继承`reporter.BasicReporter`，实现`process(self, messages)`函数即可。

目前提供了`PrintReporter`和`EmailReporter`两个子类，前者会讲更新信息打印到标准输出，后者会根据配置文件(默认为`email.json`)登录163邮箱并发给列表中的目标。

## 当前进度

目前正在陆续添加各个省份的json配置以完成爬取，具体进度见issue。

以下网站暂且搁置：

- 西藏：官方网站未知

以下网站虽然初步实现了爬取，但是写法可能需要考虑优化：

- 河南：借助BeautifulSoup和正则在疫情新闻页找时间逆序的第一条
- 云南：URL中包含一些尚未理解的query string
-  湖北：使用了比上海复杂的反爬，但黄冈卫健委也有湖北疫情消息，因此从黄冈市卫健委爬取，或许有延时

以下网站存在一定的反爬设定，不确定解决办法是否可以更好：

- 贵州：涉及JavaScript动态生成脚本，但是所需要的字符串均在名称固定的变量中，使用正则暴力匹配
- 上海：涉及JavaScript生成加密的cookie，使用正则匹配出函数与参数后借用js2py执行得出cookie
- 江苏：获取主页以绕过新闻页`![CDATA[`结构
- 天津：获取主页以绕过新闻页`![CDATA[`结构

## \*注意\*

上海网站需执行一段javascript脚本才能获取cookie以正常访问，这与本程序框架不兼容，故建议直接将Cookie写到headers中以绕开此问题。我在`crawler.py`的`HEADERS`中添加了一个我从浏览器复制过来的Cookie，但是它应该很快会失效，因此建议每次运行前手动更改一下cookie。

## 作者相关

如有问题可以发送邮件至rcycyzh@163.com联系作者。