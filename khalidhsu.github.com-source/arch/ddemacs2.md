Title: 用emacs写点点博客(续)
Date: 2012-10-19 00:21
Tags: emacs,diandian
Category: 2012-10
Slug:  ddemacs2 
Author: Khalid Hsu





终于做完了多到变态又毫无意义的数据库作业。  
好了，贴些代码，忘记所有的不快。  
上次我用emacs写点点博客时候用两个bash脚本，然后干脆修改一下，合并成一个，代码如下

```bash
#!/bin/bash
self="$0"
pushd "$(dirname -- $self)">/dev/null

ddlog()
{
    #username pwds out_cookie_file
    curl -c $3 http://www.diandian.com/login -d "account=$1&password=$2&persistent=1"
}

ddgetfrmkey()
{
    #cookiefile bgaddr out_put_ddfrmkey
    curl http://www.diandian.com/dianlog/$2/new/text  -b $1 |grep DDformKey|cut -d"'" -f 2 >$3
}

ddpost()
{
    #cookiefile bgaddr ddfrmkey  postfile
    cookiefile=$1;  bgaddr=$2; ddfrmkey=$3; postfile=$4
    ddcfg="$postfile.dd"
    tags=`cat $ddcfg|grep -v \#|grep tags` #has = !
    uri=`cat $ddcfg|grep -v \#|grep uri` # has = !
    formKey=`cat $ddfrmkey`
    markdownbody=$(python -c "import urllib; print urllib.quote(open('$postfile').read())")
    curl -b $cookiefile  http://www.diandian.com/dianlog/$bgaddr/new/text -d "formKey=$formKey&title=&markdownContent=$markdownbody&$uri&$tags&privacy=0&setTop=false&creativeCommonsEnable=true&creativeCommonsType=by_nc_sa&syncToWeibo=false&syncToQqWeibo=false&syncToDouban=false&syncToQzone=false&syncToRenren=false&syncToFacebook=false&syncToTwitter=false&syncToFlickr=false"
    
}

function showerr()
{
    echo err!
}

case $1 in  
    -l | --log-in)  
        ddlog $2 $3 $4
        ;;  
    -g | --get-fk)  
        ddgetfrmkey $2 $3 $4
        ;;
    -p | --post)  
        ddpost $2 $3 $4 $5
        ;; 
    *)  
        showerr
        ;;  
esac  
```

再贴一段调用该bash脚本的elisp，代码如下
```cl
;;ddemacs
;;by Khalid Hsu
;;2012.10.17

(defvar dd-sh "~/bash/dd-foo.sh");;path of the bash script
(defvar dd-dir "~/.diandian");;dir for saving cookie and formkey
(defvar dd-acc "khalidhsu@gmail.com")
(defvar dd-pwd nil);;just leave it nil
(defvar dd-blog-addrs '("khalidhsu"  "anotherblog"))
(defvar dd-cfg-ext ".dd")
(defvar dd-cookie-file nil);;just leave it nil

(defun dd-select-blog(blog-list)
  "Before U post."
  (let ((blog-list-in-list
         (mapcon 'list blog-list))
        (ret nil))
    (while (or (not ret)
               (= 0 (length ret)))
      (setq ret (completing-read
                 "Select a blog:"
                 blog-list-in-list nil t (car blog-list))))
    ret))

(defun dd-check-dir()
  "Before U login"
  (if (not (file-readable-p
            (directory-file-name dd-dir)))
      (make-directory dd-dir)))

(defun dd-save-cookie()
  ;;#username pwds out_cookie_file
  (shell-command (format "bash %s -l %s %s %s" dd-sh
                         dd-acc dd-pwd dd-cookie-file)))

(defun dd-save-formkey()
  ;;#cookiefile bgaddr out_put_ddfrmkey
  (mapc '(lambda (x)
           (shell-command
            (format "bash %s -g %s %s %s" dd-sh
                    dd-cookie-file x (expand-file-name
                                      x dd-dir))))
        dd-blog-addrs))

(defun dd-login()
  (interactive)
  (dd-init)
  (dd-check-dir)
  (setq dd-pwd (read-passwd "Password:"))
  (dd-save-cookie)
  (dd-save-formkey)
  (message "Done."))

(defun dd-post()
  "To post the dd article."
  (interactive)
  (dd-init)
  (save-buffer t)
  (let ((ddbuf (buffer-file-name (current-buffer)))
        target-bg
        target-bg-frmk
        ddcfgf ;;.dd
        post-cmd)
    (if  (string-suffix-p  dd-cfg-ext ddbuf)
        (setq ddbuf (substring ddbuf 0 -3)));;in case the user got it wrong
    (setq ddcfgf (concat ddbuf dd-cfg-ext))
    (if (not (file-readable-p ddcfgf) )
        (progn (dd-cfg)
               (message "Finish the dd cfg first!"))
      ;;else part: all-fine
      (setq target-bg (dd-select-blog dd-blog-addrs))
      (setq target-bg-frmk (expand-file-name target-bg dd-dir))
      ;;#cookiefile bgaddr ddfrmkey  postfile
      (setq post-cmd
            (format "bash %s -p %s %s %s %s" dd-sh
                    dd-cookie-file  target-bg target-bg-frmk ddbuf ))
      (m-eshell)
      (insert post-cmd))))

(defun dd-cfg()
  "To write the dd cfg file."
  (interactive)
  (save-buffer t)
  (let ((postname (i-get-buf-base-name (current-buffer))))
    (find-file (concat (buffer-file-name
                        (current-buffer)) dd-cfg-ext))
    (if (> (+ (point-min) 10) (point-max))
        (progn
          (delete-region (point-min) (point-max))
          (insert "[ddcfg]\n")
          (insert "tags=\n")
          (insert (concat "#uri=post%2F"
                          (format-time-string "%Y-%m-%d")
                          "%2F" postname "\n"))
          (save-buffer t)))))

(defun dd-init()
  "Exec-ed by the func of 'login' and 'post'."
  (interactive)
  (setq dd-cookie-file
        (expand-file-name "ddcookie"
                          dd-dir)))

(defalias 'ddp 'dd-post)
(defalias 'ddc 'dd-cfg)
(defalias 'ddl 'dd-login)

(provide 'dd-emacs)
```

关键的是bash代码，而elisp写了这么多，只是为了我自己用起来更舒服。
这里比较特别的地方是，由于我喜欢给文章加上标签和发布链接等信息，而又不想污染markdown文档，所以，对于每篇要发表的文章X，我都用额外的一个文件X.dd来记录附加信息。
该文件格式如。

<pre config="brush:mybox;toolbar:false;">
[ddcfg]
tags=emacs,diandian,折腾
#uri=post%2F2012-10-17%2Fddemacs2
</pre>


tags是要加入的标签，uri是自定义发布的链接，加了#号在前就是注释掉。
所以每次发布文章前，执行

<pre config="brush:mybox;toolbar:false;">
M－x ddc RET  
</pre>

来填写附加的信息。然后

<pre config="brush:mybox;toolbar:false;">
M－x ddp RET
</pre>


就发布到博客中。
当然，如果是第一次使用或者是cookie失效了，就先

<pre config="brush:mybox;toolbar:false;">
M－x ddl RET
</pre>

来获取cookie和formkey等。

