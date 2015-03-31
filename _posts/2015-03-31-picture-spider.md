---
layout: post
title: "爬取pixabay.com图片工具"
tagline: "python"
description: "用github搭建个人博客 "
tags: ["爬虫", "python"]
---
{% include JB/setup %}


由于项目需要，需要爬取一些图片素材。由于国内大部分图片网站都有免责申明，最后选定免版权网站[pixabay.com](http://www.pixabay.com)为爬取目标。由于此前只是大概了解过python，并没有实际运用过，因此快速查阅资料后开始了爬虫的制作。首先经过Firefox分析网站页面发现请求地址为：
http://pixabay.com/zh/photos/?q=eye&image_type=&cat=&order=
q参数代表搜索内容，image_type参数代表图片类型，cat参数不使用，order为排序方式。
接下来解析请求发出后返回来的页面内容，经过firebug调试，发现图片地址为：
"<img class="preview_img" data-cut="50" data-url="http://thumb9.shutterstock.com/photos/display_pic_with_logo/1072085/154754276.jpg" data-width="450" data-height="329" src="http://thumb9.shutterstock.com/photos/thumb_large/1072085/154754276.jpg">"
src即为图片地址。根据此地址解析出正则表达式
<div id="photo_grid" class="flex_grid">(.*?)<span id="paginator_clone"
准备工作做完后，正式开始撸代码，麻烦也正式开始。起初从网上搜索到的爬虫代码都是简单的实现了爬取页面，但是每次大量爬取一些图片后，程序就开始自动中断。因为项目需要的图片有几万张，如果总是出错，下载速度会很漫长。因此开始改进代码，增加了异常处理和多线程，使下载进度趋于稳定，不再进行人为干预，最后放到服务器上稳定运行。虽然寥寥数语，但是由于对python的不熟悉，中间也算历经坎坷，版本一直升级，给进行人工分布式的小伙伴造成了一定困扰，也对小伙伴表示歉意。其中代码解决了大量发出请求时候网站的反爬虫机制，和errno 10054错误引起的困扰。[代码下载](https://github.com/zhaoliguang/pictureSpider/archive/master.zip)
多线程下载：
class getImage(threading.Thread):
    def __init__(self,filename):
        threading.Thread.__init__(self)
        self.filename = filename
    def run(self):
        print u'开始下载图片...'

        while(True):
            global imageUrlList
            global imageDownloadCount
            global imageGetCount
            print u'目前捕获图片数量:',imageGetCount
            print u'已下载图片数量:',imageDownloadCount
            imageURL = imageUrlList.get()
            print u'该下载图片地址',imageURL
            try:
                time.sleep(2)
                request = urllib2.urlopen(imageURL)
                data = request.read()
                request.close()
                f = open(self.filename, 'wb')
                f.write(data)
                f.close()

                imageDownloadCount = imageDownloadCount + 1
                if(imageUrlList.empty()):
                    break

            except UnicodeDecodeError as e:
                print '-----UnicodeDecodeErrorurl:',imageURL
            except urllib2.HTTPError, e:
                time.sleep(20)
                print 'The server couldn\'t fulfill the request.',imageURL
                print 'Error code: ', e.code

            except urllib2.URLError, e:
                time.sleep(20)
                print 'We failed to reach a server.',imageURL
                print 'Reason: ', e.reason

            except socket.timeout as e:
                print "-----socket timout:",imageURL
            except Exception,e:
                print u'保存图片错误',e

参考资料：
* [Python爬虫实战四之抓取淘宝MM照片](http://cuiqingcai.com/1001.html)
* [python socket 超时设置 errno10054的解决方法](http://www.51ou.com/browse/python/43948.html)


