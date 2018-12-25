# 微信推文朗读小插件
## 一 产品需求文档  
> 有更多关于PRD部分，具体请点击[产品需求文档](/PRD.md)     


### 小结
- **加值宣言**：通过调用百度语音合成API，将微信文章转换成音频朗读出来，以按钮的形式放置在文章标题下，提升读者阅读体验。除此之外还增加了推荐文章及最近阅读文章查看的功能。
- **核心价值**：最小可用性产品是增加微信文章朗读功能，让用户能够用耳朵听文章。
- **用户痛点**
	- 用户处在难以使用眼睛阅读的环境，需要借助耳朵来阅读
	- 微信公众号文章缺少朗读功能，有公众号自己录音，但成本较高。
	- 微信读书可以读公众号文章，但下载一个app对用户来说成本高，且动作繁杂，粘度不高
	- 因此本插件直接以按钮的形式放置在文章标题下朗读文章，解决了用户用耳朵听文章的需求
- **人工智能概率性**：文章中有多音字时，会朗读错误的几率。但开发者可以实现标注多音字的发音，如：重(chong2)报集团。网络上没有提及到关于百度语音合成准确度的概率，于是我自己做了实验，阅读10篇不同的文章，其中有3篇遇到多音字读音不准确，正确率为70%。
	
- **需求列表**

| Title  | User story |Importance |Notes|API|
| ------------- | ------------- |------------- |------------- |------------- |
| 用耳朵听推文 | 用户在公交车等人多晃荡的地方也能用耳朵获取文章内容  |重要|核心功能|[百度语音合成](http://ai.baidu.com/docs#/TTS-API/top)|
| 推荐文章  | 根据用户的阅读内容推荐相关或热门的文章 |次要|推荐系统|[WechatSogou](https://github.com/Chyroc/WechatSogou)|

------

## 二.原型部分
### 1.交互及界面设计 
- **微信文章页面增加朗读文章按钮及推荐文章按钮**   

![朗读文章与推荐](/img/朗读文章与推荐.png)   

---
- **推荐文章页面与最近阅读页面**   
最近阅读页面展示用户近一周朗读过的文章   


![推荐文章与最近阅读页](/img/推荐文章与最近阅读页.png)   

------

- **锁屏页面**    


![锁屏页面](/img/锁屏界面.png)   

-------

### 2.信息设计 
- **产品功能结构**   

![产品功能结构](/img/功能结构.png)   

----

- **产品信息结构**   

![产品信息结构](/img/信息结构图.png)

----


### 3.原型文档 
**原型文档请参见**：[最小可行性产品的可交互原型文档](https://koujii.github.io/wechat_read_article/#g=1&p=%E6%9C%97%E8%AF%BB%E6%96%87%E7%AB%A0-%E7%95%8C%E9%9D%A2%E4%B8%8E%E4%BA%A4%E4%BA%92)     

下载请点击[此处](/MVP可交互原型文档.rp)



-------

## 三.API部分
### API使用
**1.核心功能：朗读文章，见[read_article](/read_article.ipynb)，代码如下：**

```
from bs4 import BeautifulSoup
from urllib import request
from aip import AipSpeech
import IPython
APP_ID = '14979803'
API_KEY = '3yZGOA0MF7vIaVSkU5tSuqH5'
SECRET_KEY = '7bYMj9pV41TbW2curcNKFG6dOwszOQV4'
client = AipSpeech(APP_ID, API_KEY, SECRET_KEY)

#抓取网页内容
def get_url(url):
    with request.urlopen(url) as f:
        data = f.read()
    Data=data.decode('utf-8')
    soup = BeautifulSoup(Data)
    return soup
	
#清理数据
def save_article(soup):
    article_title=soup.title.get_text()
    article_name=soup.find(id="js_name").get_text()
    article_content=soup.find(id="js_content").get_text()
    with open("article.txt","w",encoding='utf-8') as f:
        f.write(article_title)
        f.write(article_name)
        f.write(article_content)
		
#调用百度语音合成api以及IPython模块读文本
def read_article():
    with open ("article.txt",'r',encoding='utf-8') as f:
        p=f.read()
    result  = client.synthesis(p, 'zh', 1, {
    'vol': 5,})
    if not isinstance(result, dict):
        with open('auido.mp3', 'wb') as f:
            f.write(result)
    return IPython.display.Audio('auido.mp3')

#串起三个函数
def run(url):
    save_article(get_url(url))
    return read_article()`

```
- **最终效果：调用run()函数，即可输入网址输出音频**

![效果.png](/img/效果.png)



### 使用比较分析 

| API（产品文档）  | 文本长度  |  调用量  |价格|
| --------   | -----:  | :----:  | :----:  |
| [百度语音合成](http://ai.baidu.com/docs#/TTS-API/top)    | 小于2048个中文字或者英文数字   |  调用量无限制（QPS限制100）    |免费|
| [腾讯语音合成](https://cloud.tencent.com/document/api/441/18086)       |   字符长度不超过500   |  100次/每秒   |内测期间免费使用|
| [讯飞语音合成](https://doc.xfyun.cn/rest_api/%E8%AF%AD%E9%9F%B3%E5%90%88%E6%88%90.html)       |   长度小于1000字节    |  每日500次限额 |免费|

**小结**：从文本长度、调用量及性价比来看，百度语音合成API更适合我本次的项目。


### 使用后风险报告

- 目前语音交互处在热潮，我使用的语音合成则属于产品对人的输出的环节，即应用开口说话。使用场景广，语音市场竞争激烈，未来发展性强。（文：[中国智能语音行业格局与未来发展趋势](http://www.woshipm.com/it/590207.html) 、[2017-2018年我国智能语音行业规模及市场竞争情况分析](http://www.chyxx.com/industry/201809/675473.html)）
- 我使用[百度语音合成API](http://ai.baidu.com/tech/speech/tts)后:
	- 调用时，会自动下载整个音频文件到本地，会占用到用户的存储空间。
	- 当文章过长时，需要分段多次请求。
	- 支持中英混读，但不支持方言或其他语言，
	- Access Token有效期为30 天，一旦过期需要开发者重新申请。
	- 发音人的类型少，只有四种
	- 在多音字及断句方面有待加强，机器的声音不够自然。

  但百度语音合成的调用免费，调用无限制，性价比高；尽管有小瑕疵，却是当前最合适的选择。

------
## 清单
- [PRD.md](/PRD.md)
- [PRD-原型.md](/PRD-原型.md)
- [PRD-API.md](/PRD-API.md)
- [代码-朗读功能](/read_article.ipynb)
- [原型文档](https://koujii.github.io/wechat_read_article/#g=1&p=%E6%9C%97%E8%AF%BB%E6%96%87%E7%AB%A0-%E7%95%8C%E9%9D%A2%E4%B8%8E%E4%BA%A4%E4%BA%92)
