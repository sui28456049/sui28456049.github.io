---
title: php 测试那些事
date: 2018-01-23 09:57:02
tags: php
category: php
---

# 几种软件开发模式
## TDD
`测试驱动开发（Test-Driven Development）`

测试驱动开发是敏捷开发中的一项核心实践和技术，也是一种设计方法论，TDD首先考虑使用需求（对象、功能、过程、接口等）

主要是编写测试用例框架对功能的过程和接口进行设计，而测试框架可以持续进行验证。大行其道的一些模式对TDD的支持都非常不错，比如MVC和MVP等

## BDD
`行为驱动开发（Behavior Driven Development）`

也就是行为驱动开发。这里的B并非指的是Business，实际上BDD可以看作是对TDD的一种补充，让开发、测试、BA以及客户都能在这个基础上达成一致，JBehave之类的BDD框架

## ATDD
`验收测试驱动开发（Acceptance Test Driven Development）`

通过单元测试用例来驱动功能代码的实现，团队需要定义出期望的质量标准和验收细则，以明确而且达成共识的验收测试计划（包含一系列测试场景）来驱动开发人员的TDD实践和测试人员的测试脚本开发。面向开发人员，强调如何实现系统以及如何检验

## DDD

`领域驱动开发（Domain Drive Design）`

DDD指的是Domain Drive Design，也就是领域驱动开发,DDD实际上也是建立在这个基础之上，因为它关注的是Service层的设计，着重于业务的实现,将分析和设计结合起来，不再使他们处于分裂的状态，这对于我们正确完整的实现客户的需求，以及建立一个具有业务伸缩性的模型

#  表格对比
<table>
<thead>
<tr>
<th>区别项</th>
<th>ATDD</th>
<th>TDD</th>
<th>BDD</th>
</tr>
</thead>
<tbody>
<tr>
<td>参与者和范围</td>
<td>业务用户，开发人员，测试人员之间的沟通机制以确保需求得到充分记录</td>
<td>开发人员和测试人员之间的开发方法，以创建良好的代码单元（模块，类，功能）</td>
<td>ATDD和TDD的组合。</td>
</tr>
<tr>
<td>重点</td>
<td>专注于提取用户验收测试中的要求并用于推动开发。是一种使客户进入设计阶段的技术</td>
<td>一种模式和范例</td>
<td>专注于客户和开发者的系统行为方面，仍然是偏向于不断测试的实践来引导客户进入测试阶段，并通过逐步关注其行为进行认证。</td>
</tr>
<tr>
<td>敏捷步骤</td>
<td>1. 讨论 <br> 2. 开发 <br>3.发布 <br>不断重复</td>
<td>1. 测试 <br>2.编码<br>3.重构 <br> 不断重复</td>
<td>按预期行为逐步构建功能。<br> 它是TDD和编写使功能/行为失败的测试的延伸</td>
</tr>
<tr>
<td>输入文档</td>
<td>验收标准+示例（数据和方案）=验收测试需求文档将作为开发和测试的基础。</td>
<td>需求文档</td>
<td>用GWT格式书写的实例化文档，有时也需要验收标准</td>
</tr>
<tr>
<td>自动化要求</td>
<td>不是必须，但是可在回顾测试时实现</td>
<td>必须</td>
<td>必须</td>
</tr>
<tr>
<td>故事/特性/测试地图</td>
<td>每个故事都应对应一个验收测试</td>
<td>每个功能都需要对应一个测试</td>
<td>每个故事应对应一个行为测试</td>
</tr>
<tr>
<td>主流工具</td>
<td>· Robot Framework<br>· FitNesse<br>·FIT</td>
<td>Xunit framework<br>
</td>
<td>Cucumber JBehave, Concordian，<a href="https://www.jianshu.com/p/057dbae54134" target="_blank">lettuce</a>,Robot Framework</td>
</tr>
<tr>
<td>最终用户</td>
<td>客户</td>
<td>开发人员，测试人员</td>
<td>客户和开发者</td>
</tr>
</tbody>
</table>

# phpunit

 东西很多,需要用啥查阅文档吧:
 
 地址:`[官方文档](https://phpunit.readthedocs.io/zh_CN/latest/installation.html)`
 
 
# php selenium

类似于 python selenium ,facebook 出 了完成 webdiriver php api,挺不错,我常用来采集数据. 

# appium
测试 app, 实现了selenium一组相同  操作 api.