---
layout: post
title: 使用cPanel的cronjob制作新浪微博机器人
tags:
- cPanel
- cronjob
- Microblog
- PHP
- Sina
- 创作
status: publish
type: post
published: true
meta:
  tagazine-media: a:7:{s:7:"primary";s:64:"http://astralstorm.files.wordpress.com/2011/06/weibo_app_key.png";s:6:"images";a:6:{s:69:"http://astralstorm.files.wordpress.com/2011/06/data_in_phpmyadmin.png";a:6:{s:8:"file_url";s:69:"http://astralstorm.files.wordpress.com/2011/06/data_in_phpmyadmin.png";s:5:"width";s:3:"600";s:6:"height";s:3:"372";s:4:"type";s:5:"image";s:4:"area";s:6:"223200";s:9:"file_path";s:0:"";}s:65:"http://astralstorm.files.wordpress.com/2011/06/weibo_app_user.png";a:6:{s:8:"file_url";s:65:"http://astralstorm.files.wordpress.com/2011/06/weibo_app_user.png";s:5:"width";s:3:"600";s:6:"height";s:3:"368";s:4:"type";s:5:"image";s:4:"area";s:6:"220800";s:9:"file_path";s:0:"";}s:64:"http://astralstorm.files.wordpress.com/2011/06/weibo_app_key.png";a:6:{s:8:"file_url";s:64:"http://astralstorm.files.wordpress.com/2011/06/weibo_app_key.png";s:5:"width";s:3:"600";s:6:"height";s:3:"497";s:4:"type";s:5:"image";s:4:"area";s:6:"298200";s:9:"file_path";s:0:"";}s:61:"http://astralstorm.files.wordpress.com/2011/06/config_php.png";a:6:{s:8:"file_url";s:61:"http://astralstorm.files.wordpress.com/2011/06/config_php.png";s:5:"width";s:3:"600";s:6:"height";s:3:"397";s:4:"type";s:5:"image";s:4:"area";s:6:"238200";s:9:"file_path";s:0:"";}s:61:"http://astralstorm.files.wordpress.com/2011/06/oauth_done.png";a:6:{s:8:"file_url";s:61:"http://astralstorm.files.wordpress.com/2011/06/oauth_done.png";s:5:"width";s:3:"600";s:6:"height";s:3:"492";s:4:"type";s:5:"image";s:4:"area";s:6:"295200";s:9:"file_path";s:0:"";}s:58:"http://astralstorm.files.wordpress.com/2011/06/cronjob.png";a:6:{s:8:"file_url";s:58:"http://astralstorm.files.wordpress.com/2011/06/cronjob.png";s:5:"width";s:3:"600";s:6:"height";s:3:"336";s:4:"type";s:5:"image";s:4:"area";s:6:"201600";s:9:"file_path";s:0:"";}}s:6:"videos";a:0:{}s:11:"image_count";s:1:"6";s:6:"author";s:8:"24007445";s:7:"blog_id";s:8:"23873706";s:9:"mod_stamp";s:19:"2011-06-30
    11:39:12";}
  _wp_old_slug: '%e4%bd%bf%e7%94%a8cpanel%e7%9a%84cronjob%e5%88%b6%e4%bd%9c%e6%96%b0%e6%b5%aa%e5%be%ae%e5%8d%9a%e6%9c%ba%e5%99%a8%e4%ba%ba'
  _oembed_b8370118af5895dde2842b535de95877: '{{unknown}}'
  _oembed_27a2c86aa56c01c7e1593206c5de8d8b: '{{unknown}}'
---
本文研究如何利用cPanel的cronjob（时钟守护程序）制作自动发送条目的新浪微博机器人。这里的"微博机器人"指按时发送条目的那种微博程序，比如我自己做的<a href="http://weibo.com/1913633425">@奥修的金块</a>。暂不讨论像<a href="http://twitter.com/#!/rtmeme">@rtmeme</a>那么高级的。

具体流程是，在新浪开放平台注册一个应用，把微博帐号添加为应用的测试者，完成一系列授权后，用cPanel主机的PHP脚本就可以向微博帐号发送条目了。最后用cronjob把这个过程设为自动执行。

准备工作包括以下几个方面：
<ul>
	<li>支持cronjob的cPanel空间</li>
	<li>储存微博条目的MySQL数据库</li>
	<li>注册新浪微博应用</li>
	<li>处理微博OAuth的PHP库</li>
	<li>看看新浪微博开发者协议</li>
</ul>
此外最好对PHP和MySQL的语法，以及OAuth基本原理有一些粗浅的了解。之后就可以开搞了。
<h2>准备数据</h2>
微博机器人我见过最多的是报时、发英文或日文单词的，还有各种树洞机器人。大家可以发挥自己的创意～建议文字尽量短小（我发现140字都有人嫌长不看），条目之间相对独立，更新不要太快。测试数据有10多条就行了，成功后再整理正式的。
<!--more-->
在cPanel主机当然用PHP+MySQL，在向导中建好数据库，把你的数据导入进去。每一条除了content字段以外，还需要id和count字段，以后管理发送次数时要用到。其余字段看自己需求。

<img src="http://astralstorm.files.wordpress.com/2011/06/data_in_phpmyadmin.png" alt="data-in-phpmyadmin.png" width="600" height="372" />

数据导入MySQL，最省心的办法是导入整理好的EXCEL表格（第一行写字段名称），折腾了那么久，我只有导入xls时从没乱码过……
<h2>注册新浪微博应用</h2>
<a href="http://open.weibo.com/">新浪微博开放平台注册页面</a>，点"我是开发者"进入，创建新的"普通应用"，表单随便填一下，这个应用以后就自己使唤，没法过审核的。完后在"编辑应用属性-&gt;测试用户"，把微博帐号的UID填进去。最后，记下App Key和App Secret备用。

<img src="http://astralstorm.files.wordpress.com/2011/06/weibo_app_user.png" alt="weibo-app-user.png" width="600" height="368" />

<img src="http://astralstorm.files.wordpress.com/2011/06/weibo_app_key.png" alt="weibo-app-key.png" width="600" height="497" />
<h2>新浪微博OAuth授权</h2>
接下来解决OAuth的问题，让我们的应用能够操作自己的微博。首先下载<a href="http://code.google.com/p/libweibo/">PHP版的新浪微博OAuth库</a>，解压后把刚才记下的App Key和App Secret填在config.php中。

<img src="http://astralstorm.files.wordpress.com/2011/06/config_php.png" alt="config-php.png" width="600" height="397" />

把将要上传的路径填在index.php中替换掉第22行的$callback，完后上传到主机空间。（这些都可以上传完了再改。）
<blockquote>$aurl = $o-&gt;getAuthorizeURL( $keys['oauth_token'] ,false , <span style="color:#ff0000;">$callback</span> ); //第22行</blockquote>
在浏览器访问刚才上传的地址，比如<a href="http://xxxxx.com/weibo-oauth/index.php">http://xxxxx.com/weibo-oauth/index.php</a> ，按照提示登录微博并授权给这个应用，之后自动转入<a href="http://xxxxx.com/weibo-oauth/weibolist.php">http://xxxxx.com/weibo-oauth/weibolist.php</a> 页面，测试一下，可以发送微博就没问题了。

<img src="http://astralstorm.files.wordpress.com/2011/06/oauth_done.png" alt="oauth-done.png" width="600" height="492" />
<h2>用PHP发送微博条目</h2>
现在需要改造weibolist.php使其按我们的要求发送微博。看一下weibolist.php就会发现提交微博是在
<blockquote>$rr = $c-&gt;update( $_REQUEST['text'] ); //第43行</blockquote>
实现的，那么我们只需要从MySQL中取出数据放在这里就可以。在weibolist.php后面加入以下内容
<blockquote>$conn = mysql_connect('localhost', '数据库管理员名', '管理员密码');
if (!$conn) { die('Could not connect: ' . mysql_error());}
mysql_select_db('数据库名');
mysql_query("set names utf8");

$query = "SELECT id,count,content FROM quot WHERE count=(SELECT MIN(count) FROM quot) ORDER BY RAND() LIMIT 1;";
$result=mysql_query($query,$conn);
$row=mysql_fetch_array($result);

$query2 = "UPDATE quot set count=count+1 WHERE id = ". $row['id'] . ";" ;
$result2 = mysql_query($query2,$conn);

$c-&gt;update( $row['content'] );
echo "index=".$row['id']." / count=".$row['count']."&lt;br/&gt;";
echo $row['content'];
echo '----send done';</blockquote>
大体上就是这样，上文提到过记录发送次数的count字段（开始都是0），查询语句就是永远选择当前count最小的数据中随机的一条，并将其count+1。当然纯随机发布或按顺序发布就更简单了，大家可以按自己的需要折腾之。

改好后，每在浏览器刷新这个页面应该会提交一条微博。

之后把weibolist.php中跟浏览器会话（Session）相关的内容替换掉，因为以后用cronjob执行这个PHP文件，那时可没有$_SESSION这玩意。
<blockquote>$c = new WeiboClient( WB_AKEY , WB_SKEY , <span style="color:#ff0000;">$_SESSION['last_key']['oauth_token']</span> , <span style="color:#ff0000;">$_SESSION['last_key']['oauth_token_secret']</span> ); //第8行</blockquote>
怎样知道这两个值呢？有好多办法，比如直接把它写在PHP里，那么从前台就看到了……
<blockquote>&lt;p&gt;token = &lt;?=$_SESSION['last_key']['oauth_token']?&gt;&lt;/p&gt;
&lt;p&gt;token_secret = &lt;?=$_SESSION['last_key']['oauth_token_secret']?&gt;&lt;/p&gt;</blockquote>
把这段也放在weibolist.php里面，刷新一下就知道token和token_secret，注意这跟应用的App Key/App Secret是两回事。
<h2>用cronjob设置自动任务</h2>
最后要把这个PHP设成自动运行的，需要用到cPanel的cronjob（时钟守护作业），在cPanel主面板找到这个工具，命令行的语法是
<blockquote>php /home/<span style="color:#ff0000;">cPanel帐号名</span>/public_html/<span style="color:#ff0000;">刚才页面的地址.php</span></blockquote>
<img src="http://astralstorm.files.wordpress.com/2011/06/cronjob.png" alt="cronjob.png" width="600" height="336" />

跟时间相关的设置按它的提示去选择即可。到这里就全部OK了。最后提醒大家去看看新浪微博的<a href="http://open.weibo.com/wiki/index.php/应用开发者协议">开发者协议</a>，尤其是对机器人式应用的限制。低调行事啊。

（完）
