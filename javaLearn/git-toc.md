# 生成git目录

>1，基于markdown语法，用[toc]一般也可以生成目录，但是github不认，所以需要借助工具github-markdown-toc.go

>2,下载了一个window版本的 [gh-md-toc 工具](https://github.com/ekalinin/github-markdown-toc.go/releases/download/1.0.0/gh-md-toc.windows.386.tgz)

>3,用7zip解压
>4，在git的bash下执行  /d/markdown-toc/gh-md-toc hashmap.md 会得到目录，将内容拷贝到hashmap.md文件首部，用作目录，再提交git，就可以在github上看到目录结构了

* [git目录生成工具](https://github.com/ekalinin/github-markdown-toc.go)
* [gh-md-toc git目录生成工具releases下载](https://github.com/ekalinin/github-markdown-toc.go/releases)
* [github官方文档，介绍了git相关的使用](https://docs.github.com/cn/free-pro-team@latest/github/writing-on-github/basic-writing-and-formatting-syntax#%E6%A0%B7%E5%BC%8F%E6%96%87%E6%9C%AC)