Title: 用emacs写点点博客
Date: 2012-10-14
Tags: emacs,diandian
Category: 2012-10
Slug:  ddemacs 
Author: Khalid Hsu




用emacs写点点博客
发现点点还是挺不错的，清新无广告，还支持高度的自定义。不过话说回来，在网页上那个小小的文本窗口写blog绝非一件很cool的事，特别是对于emacser。   
如果要用emacs写blog的话，可以通过邮件发送的方式。但我发现用邮件发布的话，如果你要加入标签，标签还是会存在于文本的正文中。（不知道点点的攻城狮现在有没有改进）     
当然，我喜欢另外一种办法，那就是通过模拟http请求来写blog。  

用firebug监视一下点点发布文章的HTTP请求，可以发现，发布文章的post请求大致为


```bash
"formKey=$formKey&title=&markdownContent=$markdownbody&$uri&$tags&privacy=0&setTop=false&creativeCommonsEnable=false&creativeCommonsType=by_nc_sa&syncToWeibo=false&syncToQqWeibo=false&syncToDouban=false&syncToQzone=false&syncToRenren=false&syncToFacebook=false&syncToTwitter=false&syncToFlickr=false"
```

其中$formkey为对应每个博客的一个数字ID。$markdownbody为发布的内容（我用的是markdown编辑模式发布的）
其他的参数也没什么好解释的，就是字面的意思。

获取formKey的办法

```bash
curl http://www.diandian.com/dianlog/khalidhsu/new/text  -b ~/ddcookie |grep DDformKey|cut -d"'" -f 2 >~/ddformkey
```

其中khalidhsu为我的博客地址，获取的ddformkey保存为~/ddformkey。   
等等，你还没拿到cookie哦，要先把这块不能吃的cookie拿到手才能干活的。代码如下：


```bash
curl -c ~/ddcookie http://www.diandian.com/login -d "account=khalidhsu%40gmail.com&password=youpassword&persistent=1"
```

将上面的账户名和密码改为你的即可。执行一下，得到的cookie就保存在~/ddcookie。  
以上的几步就是发表文章的关键几步了。  
实际上，我把这写代码写成两个bash脚本，然后再用elisp调用的。  
听起来很麻烦？当然不会，我在emacs里面写好文章，只需要简单的两步就可以发表了^_^  
好了，先写到这，有空再补充一下。 


