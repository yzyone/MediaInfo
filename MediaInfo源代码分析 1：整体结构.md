# MediaInfo源代码分析 1：整体结构

注：此前已经写了一系列分析MediaInfo源代码的文章，列表如下：  
[MediaInfo源代码分析 1：整体结构](http://blog.csdn.net/leixiaohua1020/article/details/12016231)  
[MediaInfo源代码分析 2：API函数](http://blog.csdn.net/leixiaohua1020/article/details/12449277)  
[MediaInfo源代码分析 3：Open()函数](http://blog.csdn.net/leixiaohua1020/article/details/12449803)  
[MediaInfo源代码分析 4：Inform()函数](http://blog.csdn.net/leixiaohua1020/article/details/12451561)  
[MediaInfo源代码分析 5：JPEG解析代码分析](http://blog.csdn.net/leixiaohua1020/article/details/12452991)  

===================

MediaInfo 用来分析视频和音频文件的编码和内容信息，是一款是自由软件 (免费使用、免费获得源代码）。之前编程的时候，都是直接调用它提供的Dll，这次突然来了兴趣，想研究一下它内部究竟是怎么实现的。

MediaInfo的源文件可以从Sourceforge上面下载，地址：http://sourceforge.net/projects/mediainfo/

在这里我使用的是 Media Player Classic (MPC-HC)源代码自带的MediaInfo库，内容应该都是一样的。

MPC-HC把MediaInfo整合到了它的“属性”选项卡中。

使用VC2010打开MPC-HC之后，可以看到MediaInfo的库的源代码如下图所示：

![](http://img.blog.csdn.net/20130925155328140?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

展开Source Files（文件太多，截图竟然截不下来= =）：

![](http://img.blog.csdn.net/20130925155440453?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

在此首先介绍几个我已知的几个文件夹中的源代码的功能：

Archive：支持的各种压缩文档，由图可见包括7z，rar，zip，tar等格式

![](http://img.blog.csdn.net/20130925155951093?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

Audio：支持的各种音频编码方式，由图可见包括aac，ac3，ape等等

![](http://img.blog.csdn.net/20130925160040171?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

Duplicate：不知道干啥的

Export：设置导出的格式，由图可见可以导出为MPEG7格式

![](http://img.blog.csdn.net/20130925160331296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

Image：支持的图片压缩编码方式，由图可见包括bmp，jpeg，等格式

![](http://img.blog.csdn.net/20130925160554296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

Muitiple：支持的文件封转格式。由图可见包括flv，mp4，mkv等格式

![](http://img.blog.csdn.net/20130925160647218?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

Reader：支持的输入方式。我一直以为MediaInfo只支持文件输入，后来发现还支持MMS这样的流媒体输入

![](http://img.blog.csdn.net/20130925160734609?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

Tag：支持的标签，包括idv3等等

![](http://img.blog.csdn.net/20130925160936734?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

Text：支持的文本。这个用的比较少

![](http://img.blog.csdn.net/20130925161033625?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

Video：支持的视频编码。由图可见包括H.264，H.263等。令人瞩目的是，也支持HEVC。

![](http://img.blog.csdn.net/20130925161229750?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

先分析这么多吧，以后有空再写。
