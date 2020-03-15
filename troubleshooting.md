# 4. 常见问题

## 4.1 github的问题

Atom不支持自动注册远程github，需要通过命令行去添加remote repository。 比如：

git remote add origin https://github.com/weihj1999/redmine.git
git push -u origin master

然后点击右下角的github标签，会提示登录 github.atom.io/login, 需要注册一个atom的账号。

获取一个token后可以正常登录。

它会自动识别你的githug账号，授权登录即可。

然后在git标签进行stage，commit和推送即可了。
