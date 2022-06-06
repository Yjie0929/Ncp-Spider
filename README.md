# Ncp-Spider
---
# 无目录，直接上总结
---
## 读前必阅

 - **口水话讲解**
 - **基于scrapy框架**
 - **数据来自百度、腾讯**
 - **数据存储在csv、mysql中**
 - **爬虫定时调度方法很多，不介绍**

---
## 爬虫框架
该爬虫基于**scrapy框架**实现，因为对代码的管理比较方便，可以在各个特定的.py文件下实现不同需求，如在spider下专心写爬虫，数据可以临时存储在Item类中，持久化存储又能够在Pipeline下实现，且scrapy的中间件功能真的很好用。

---
## 数据来源
**数据分别来自：度娘、企鹅**。因为度娘把各省份的数据都丢到了js脚本中，且爬出来的数据异常杂乱，而企鹅把各省份数据封装为json，所以决定**在度娘爬取总体数据**，**在企鹅爬取各省份的数据**。

---
## 爬取思路
度娘与企鹅并没有对数据进行加密，数据均为动态加载，可以使用浏览器抓包工具发现接口分别为**from=osari_aladin_banner**(度娘)、**eiyan.htm**(企鹅)，企鹅的数据是个json包，可以在scrapy中变成dict后通过逐层找key提取value；度娘因为将数据封装在js脚本中了，需要正则提取需要的数据：
```python
以百度的response：'"new_diagnosis":"8888","new_local":"9999"'为举例

ex = r'"summaryDataIn":{(.*)},"summaryDataOut"'
data = re.findall(ex, response.text, re.S)[0]  # 取关键Json，但返回的数据类型为str
dic = dict()
for item in data.split(','):  # 分割各元素，得到：('"new_diagnosis":"8888"')
    item = item.replace('"', '')  # 去除多余符号，得到：('new_diagnosis:8888')
    key = item.split(':')[0]  # 分割，取左边为key，得到：'newdiagnossi'
    value = item.split(':')[1]  # 分割，取右边为value，得到：'8888'
    dic[key] = int(value)
```
---

## 数据存储思路
假设数据存储方式分为**临时存储**、**持久化存储**，这两种功能可以在scrapy中的**Item类**(items.py)、**管道**(pipelines.py)中实现，Item类是为管道的持久化存储提供了一个临时存储区。
为了防止数据遭到恶意破坏，我**设置了两个管道**，管道一负责**将数据写入本地的csv**，管道二**将数据写入数据库**，必要时可以从本地CSV中将数据重写入Mysql。
因为该爬虫是为了后期做数据可视化图表准备的数据收集，所以在爬取数据前我**以日期为名创建了不同的目录、数据库的table**。
```python
记得启用多管道
ITEM_PIPELINES = {
    'ncpPro.pipelines.NcpproPipeline': 300,  # 数据存储在本地CSV
    'ncpPro.pipelines.NcpproPipelineToMySQL': 350,  # 数据存在在Mysql
}
```
---
## （总结）必须要吐槽的话
因为scrapy是基于异步协程实现的爬取机制，在Pipeline中存储数据时总会出现一些并发问题，为了解决该问题花了我不少时间才处理，管道中有一半的代码是为了解决这一问题，如果**后期确定不会再需要爬取其它数据的话尽量不使用scrapy，直接在.py中配置多线程或协程爬取数据**。# Ncp-Spider
---
