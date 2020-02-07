---
date: 2019-09-19 11:11
layout: post
status: public
title: SpringBoot笔记
---

# 实战笔记
## 静态资源
优先级顺序/META-INF/resources>resources>static>public

# SpringBoot精要
1. 自动配置： 会根据ClassPath里的库自动生成一些常用模板Bean，比如JDBC H2
2. 起步依赖：自动引入所需库
3. 命令行： SpringBOOTCLI 甚至单个文件构造web应用
4. Actutor：窥探运行时的内部情况 调试和运维都会很方便

# 启动
start.spring.io 获得初始化项目
@SpringBootApplication开启了Spring的组件扫描和Spring Boot的自动配置功能。