
在 _post 目下创建自己的文章

然后

jekyll build

重新打包文件，生成的文档放在 _site 目录下

将 _site 拷贝的httpd对应的服务目录下

cp -r ~/code/github/kingqh.github.io/_site/* .
