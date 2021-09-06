# java 内存占用分析

##

>实时查看：jmap -histo <pid> | head -20
>把heap文件dump下来分析，MAT(Memory Analyse Tool)分析整个文件，MAT插件需要下载安装： jmap -dump:format=b,file=filename.bin <pid>

![](./res/mat-jmap-dump.png "")