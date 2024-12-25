# AI&行为树ProfilerTrace

[TOC]

---

## 1. Editor数据

![EditorServer](..\..\资源引用\图片\AI&行为树ProfilerTrace\EditorServer.png)

## 2. Test数据

![TestServer](..\..\资源引用\图片\AI&行为树ProfilerTrace\TestServer.png)

## 3. 数据对比

1. Test与Editor对比有大量的数据不统一
2. 一部分是应对Test测试新加的Trace Editor下没有
3. 一部分是Test测试排除了一部分的Trace

## 4. 补充打点

1. BTComponentTick
2. BTTask
3. AITask

## 5. 重点性能检测

1. AI感知
2. 寻路

## 6. 查漏补缺

1. 服务器跑ABP 宝箱&能量核心
![ServerABP1](..\..\资源引用\图片\AI&行为树ProfilerTrace\ServerABP1.png)

![ServerABP1](..\..\资源引用\图片\AI&行为树ProfilerTrace\ServerABP2.png)

2. 释放技能有大量的OnSkillActive

![image-20241225215529996](C:\Users\peili.dong\AppData\Roaming\Typora\typora-user-images\image-20241225215529996.png)
