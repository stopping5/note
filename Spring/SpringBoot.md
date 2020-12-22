# SpringBoot 整理

## 1、监听器

- applicationContext域
- session域
- request域
- 自定义事件监听
  - 设置ApplicationEvent
  - 实现implements ApplicationListener<MyEvent> 在此处进行逻辑操作
    - applicationContext.publishEvent(event);发送事件监听
  - 一旦被监听