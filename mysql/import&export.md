# mysql 数据导入导出

## mysqldump导出
>mysqldump -u root -p database_name table_name > dump.txt
>mysqldump -u web -p'!web@123#@AIENGLISH' -P12046 -Dop_resource2 word_video > ./word_video.sql   --去掉表名，就是导出库

## mysql 导入
>mysql -u web -p'password' -P12046 -Dop_resource2 < ./word_video.sql
>mysql -u web -p'!web@123#@AIENGLISH' -P12046 -Dop_resource2 < ./word_video.sql