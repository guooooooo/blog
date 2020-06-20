---
title: webpack运行流程
date: 2020-02-18
tags:
 - webpack
categories:
 - webpack
---

## webpack主要工作流程

webpack的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

- 初始化参数

  从配置文件和`shell`语句中读取与合并参数，得到最终的配置参数

- 开始编译

  用合并后的参数初始化`Compiler`对象，加载所有配置的插件，执行对象的`run`方法开始执行编译；确定入口，根据配置中的`entry`找出所有的入口文件

- 编译模块

  从入口文件出发，根据配置的`Loader`匹配规则调用对应的`loader`对模块进行编译。然后根据模块的依赖关系递归进行`loader`匹配和处理。直到所有入口的依赖模块都经过了处理

- 模块编译完成

  使用对应`loader`编译过所有依赖模块后，得到了每个模块被编译后的最终内容以及模块间的依赖关系

- 输出资源

  根据入口和模块之间的依赖关系，组装成一个个包含多个模块的`chunk`，再通过模板把每个`chunk`转换成一个单独的文件加入到输出列表，这一步是可以修改输出内容的最后机会

- 输出完成

  在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统

> 在以上过程中，webpack会在特定的时间点广播出特定的事件，插件在监听的感兴趣的事件后会执行特定的逻辑，并且插件可以调用webpack提供的API改变webpack的运行结果

![k782ST.png](https://t1.picb.cc/uploads/2020/02/18/k782ST.png)

### 初始化阶段

![k78kQF.jpg](https://t1.picb.cc/uploads/2020/02/18/k78kQF.jpg)

### 编译阶段

![k78tDr.jpg](https://t1.picb.cc/uploads/2020/02/18/k78tDr.jpg)

![k78wnJ.jpg](https://t1.picb.cc/uploads/2020/02/18/k78wnJ.jpg)

### 输出阶段

![k78fq0.jpg](https://t1.picb.cc/uploads/2020/02/18/k78fq0.jpg)

