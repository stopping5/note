# 设置内容

### 1、tomcat8+在IDEA乱码？

在setting中设置fileEncoding = utf-8

![image-20200611141821358](img\image-20200611141821358.png)

设置tomcat中文：-Dfile.encoding=UTF-8

![image-20200611141917487](img\image-20200611141917487.png)

【实在不行】IDEA安装目录bin/idea64.exe.vmoptions 打开 添加一行配置 -Dfile.encoding=UTF-8

![image-20200611142033125](img\image-20200611142033125.png)



### 2、jar引用不到问题

在project settings libraries重新导入

![image-20200612093415117](img\image-20200612093415117.png)

![image-20200612093100879](img\image-20200612093100879.png)

导入路径

H:\ideaWorkspace\weifuwu\ectrip-market\web\src\main\webapp\WEB-INF\lib

将项目中lib的jar导入。



### 3 、Tomcat热部署

一、配置tomcat

![image-20200617111219044](file://F:/app/%E6%96%87%E6%9C%AC%E5%B7%A5%E5%85%B7/Typora/TyporaImage/learning/tomcat%E7%83%AD%E9%83%A8%E7%BD%B2/image-20200617111219044.png?lastModify=1592788137)

二、添加依赖

![image-20200617111248139](file://F:/app/%E6%96%87%E6%9C%AC%E5%B7%A5%E5%85%B7/Typora/TyporaImage/learning/tomcat%E7%83%AD%E9%83%A8%E7%BD%B2/image-20200617111248139.png?lastModify=1592788137)

3、删除第一个war包

4、返回server



### 4、IEAD切换数据源

（1）maven切换数据源

![image-20200630115217412](img/image-20200630115217412.png)

（2）在Web模块下查看数据源配置是否正确

![image-20200630115320727](img/image-20200630115320727.png)

![image-20200630115333116](img/image-20200630115333116.png)

（3）重启服务器（不行则刷新maven）





# 代码管理

## 1、修改代码之前一定要先pull避免冲突

## 2、新建的文件需要add之后再commit

## 3、静态页面存放在src下面

路径：H:\ideaWorkspace\weifuwu\ectrip-market\web\src\main\webapp\WEB-INF\templates\wechat\malls

## 4、不用提交的代码 右键->new channel

![image-20200716233356604](img/image-20200716233356604.png)