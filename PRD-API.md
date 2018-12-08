## API使用
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



## 使用比较分析 

| API（产品文档）  | 文本长度  |  调用量  |价格|
| --------   | -----:  | :----:  | :----:  |
| [百度语音合成](http://ai.baidu.com/docs#/TTS-API/top)    | 小于2048个中文字或者英文数字   |  调用量无限制（QPS限制100）    |免费|
| [腾讯语音合成](https://cloud.tencent.com/document/api/441/18086)       |   字符长度不超过500   |  100次/每秒   |内测期间免费使用|
| [讯飞语音合成](https://doc.xfyun.cn/rest_api/%E8%AF%AD%E9%9F%B3%E5%90%88%E6%88%90.html)       |   长度小于1000字节    |  每日500次限额 |免费|

**小结**：从文本长度、调用量及性价比来看，百度语音合成API更适合我本次的项目。


## 使用后风险报告

- 目前语音交互处在热潮，我使用的语音合成则属于产品对人的输出的环节，即应用开口说话。使用场景广，语音市场竞争激烈，未来发展性强。（文：[中国智能语音行业格局与未来发展趋势](http://www.woshipm.com/it/590207.html) 、[2017-2018年我国智能语音行业规模及市场竞争情况分析](http://www.chyxx.com/industry/201809/675473.html)）
- 我使用[百度语音合成API](http://ai.baidu.com/tech/speech/tts)后:
	- 调用时，会自动下载整个音频文件到本地，会占用到用户的存储空间。
	- 当文章过长时，需要分段多次请求。
	- 支持中英混读，但不支持方言或其他语言，
	- Access Token有效期为30 天，一旦过期需要开发者重新申请。
	- 发音人的类型少，只有四种
	- 在多音字及断句方面有待加强，机器的声音不够自然。

  但百度语音合成的调用免费，调用无限制，性价比高；尽管有小瑕疵，却是当前最合适的选择。