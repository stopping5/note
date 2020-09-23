# 自动注入

[博客链接](https://blog.csdn.net/October_zhang/article/details/105050017)

## 1、情景：

最近在写MQ时发现在使用了@Component同时使用@Autowired自动注入service的时候发现并未注入成功，得到的对象是null

## 2、**原因**：
在使用@Component注解将bean实例化到spring容器内的时候，@Autowired是在这个bean之中的，@Autowired还未完成自动装载，所以导致service为null

