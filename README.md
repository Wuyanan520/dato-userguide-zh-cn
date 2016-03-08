# GraphLab Create 用户指南中文版

英文版本：2016-03-08 commit 91c04027e37c5f4f6b415add546f19c6e29b71c6

**翻译有不到位的地方，欢迎批评指正！**

[目录](SUMMARY.md)

Dato的使命是构建最强大和易用的数据科学工具，让你能快速将灵感转化为产品。

[GraphLab Create](https://dato.com/products/create/) 是一个允许程序员执行端到端
大规模数据分析和数据产品开发的Python包。

- **使用SFrames进行数据提取和清洗**。SFrame是一个高效的基于磁盘的表数据结构，
  它可以摆脱内存的限制。它甚至可以让你在笔记本上将分析和数据处理扩展到TB级数据。

- **使用GraphLab Canvas进行数据探索和可视化**。GraphLab Canvas是一个基于浏览器的
  交互式GUI，允许你探究表格数据，概要图和统计信息。

- **使用SGraph进行网络分析**。SGraph是基于磁盘的图数据结构，它在SFrame中存储顶点和边。

- **使用机器学习工具集进行预测模型开发**。GraphLab Create包含多个使用快速、可扩展算法
  进行快速原型的工具集。

- **使用数据管道（data pipelines）实现产品自动化**。数据管道（data pipelines）允许你
  将可复用代码任务集合成作业（jobs）并在常见的运行平台（如Amazon Web Services, Hadoop）自动运行。

在本用户指南中，你将学会如何使用GraphLab Create进行如下工作：

- 修改（munge）和探索结构化和非结构化数据
- 使用先进的机器学习算法构建预测模型和推荐系统
- 将代码部署到产品环境并应用到真实业务场景

每一章都包含动手实践作业，并给出了答案及附加阅读材料。如果你有任何问题，请不要迟疑，
来我们的[论坛](http://forum.dato.com/categories/graphlab-create).提问吧。

### 开源

本指南源码使用3-clause [BSD license](LICENSE)，可以在[Github](https://github.com/dato-code/userguide)上找到。
如果你有建议或发现bug，我们欢迎你使用Github issues和pull requests报告，提交方法请参考章节[contributing](contributing.md)。

要构建该指南，请安装npm并运行如下命令：
```
npm install
npm run gitbook-dep
npm run gitbook
```
生成的html保存在`_book/index.html`。
