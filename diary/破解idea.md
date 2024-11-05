# 破解idea

## 工具
/Users/woshare/develop/devDoc/idea-active-tools

## 要求
好像是2021～2022版本的，我安装2024版本的没用

## 破解步骤
1，就按照工具的指示步骤操作，比较简单
2，第一次是版本2022的，基本很快就破解可用了
3，第二次是版本2024的，操作之后，破解没生效
4，之后卸载idea 2024，再安装2022
5，此时2022版本，打不开
6，解决打不开2022版本:（好像是不干净卸载，尽管是正操的卸载操作，还是会有点问题，需要额外再去删除一些文件）
    1) 卸载idea 2022
    2）到/Users/woshare/Library/Application Support/JetBrains，把所有idea的版本都删除
    3）重新安装idea 2022
    4）可以打开，
7，再按照工具不走操作，还是有问题
    1）因为下载的版本是idea2022.1.4版本，cd到/Users/woshare/Library/Application Support/JetBrains/IntelliJIdea2022.1
    2）cat  idea.vmoptions发现这里有地址是不存在的：javaagent:/Users/liujiaxin/configfile/yz/active-agt-idea.jar
    3）找到对应的文件地址：/Users/woshare/configfile/yz/active-agt-idea.jar，是有文件的
    4）修改idea.vmoptions中的javaagent
    5）打开idea，查看注册信息，到2099年
