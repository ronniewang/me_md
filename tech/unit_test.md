---
title: 关于单元测试
date: 2015-04-06
tags:
---

# 关于单元测试

本笔记主要来源于《单元测试的艺术》一书和本人些许经验

## 概述

* 什么是单元测试
* 单元测试的核心技术
* 怎样写好单元测试

## 什么是单元测试

### 单元测试定义

一个单元测试是一段自动化的代码，这段代码调用被测试的工作单元，之后对这个工作单元的单个最终结果的某些假设进行验证。单元测试几乎都是用单元测试框架编写的，能快速运行。单元测试可靠，可读，可维护。只要产品代码不发生变化，单元测试的结果是稳定的

### 集成测试

一般具有如下属性的，就认为是集成测试

* 时间长
* 不稳定
* 有外部依赖

### 单元测试的好处

* 可做开发文档
* 重构或修改代码更有信心
* 对自己的程序设计更清晰

### 测试的三种类型

* 测试返回值
* 测试系统状态改变
* 测试第三方调用

### 单元测试框架

* 编写测试更容易
  * 提供基础类和接口
  * 含有代码级别的标记，用于标记测试方法的属性，@Test
  * 提供断言类，用于验证代码

* 可以控制测试的执行策略
  * 发现测试
  * 自动运行
  * 显示运行期间状态
  * 用命令行自动话

* 显示更详细的运行结果
  * 已运行测试数目
  * 未运行测试数目
  * 失败的测试数目
  * 失败的原因
  * ASSERT消息
  * 失败的代码位置
  * 异常信息

### 第一个单元测试

```
public class Sample {

    public static String hello(String name) {

        return "Hello " + name;
    }
}

public class SampleTest {
    @Test
    public void testHello() {
        //prepare
        String expected # "Hello Ronnie";
        //call
        String result # Sample.hello("Ronnie");
        //validate
        assertEquals(result, expected, "not correct");
    }
}
```

*单元测试三段式*

* 准备预期结果
* 调用测试方法
* 验证预期结果

----

## 单元测试核心技术

### Mock/Stub

* Mock和Stub都是模拟对象
* Mock对象会使测试失败，测试时会对Mock对象进行断言
* Stub就是模拟对象的行为，不需要被测试

使用Stub破除依赖，使用Mock验证交互

深入讨论： http://martinfowler.com/articles/mocksArentStubs.html

*Mock框架实例*

[source,java]
package com.spider.service;

import com.spider.entity.ServiceStateEntity;
import com.spider.entity.ServiceStateHistoryEntity;
import com.spider.global.ServiceName;
import com.spider.repository.ServiceStateHistoryRepository;
import com.spider.repository.ServiceStateRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.invocation.InvocationOnMock;
import org.mockito.runners.MockitoJUnitRunner;
import org.mockito.stubbing.Answer;

import java.util.Date;

import static org.mockito.Mockito.*;

@RunWith(MockitoJUnitRunner.class)
public class HeartBeatServiceImplTest {

    @InjectMocks
    private HeartBeatServiceImpl heartBeatService;

    @Mock
    private ServiceStateRepository serviceStateRepository;

    @Mock
    private ServiceStateHistoryRepository serviceStateHistoryRepository;

    @Test(expected # NullPointerException.class)
    public void heartBeat_PassNullArgument_ThrowNullPointerException() {
        heartBeatService.heartBeat(ServiceName.LiJiRobot.getName(), null, null, true, "");
    }

    @Test
    public void heartBeat_InsertServiceState_HistoryRepositoryShouldNotBeCalled() {
        when(serviceStateRepository.findByService(isA(String.class)))
                .thenAnswer(new Answer<ServiceStateEntity>() {
            @Override
            public ServiceStateEntity answer(InvocationOnMock invocation) throws Throwable {
                return null;
            }
        });
        when(serviceStateRepository.save(isA(ServiceStateEntity.class)))
                 .thenAnswer(new Answer<ServiceStateEntity>() {
            @Override
            public ServiceStateEntity answer(InvocationOnMock invocation) throws Throwable {
                return new ServiceStateEntity();
            }
        });
        heartBeatService.heartBeat(ServiceName.LiJiRobot.getName(), new Date(), new Date(), true, null);
        verify(serviceStateRepository).findByService(isA(String.class));
        verify(serviceStateRepository).save(isA(ServiceStateEntity.class));
        verifyNoMoreInteractions(serviceStateRepository);
    }

    @Test
    public void heartBeat_UpdateServiceState_HistoryRepositoryShouldBeCalled() {
        when(serviceStateRepository.findByService(isA(String.class)))
                 .thenAnswer(new Answer<ServiceStateEntity>() {
            @Override
            public ServiceStateEntity answer(InvocationOnMock invocation) throws Throwable {
                return new ServiceStateEntity();
            }
        });
        when(serviceStateRepository.save(isA(ServiceStateEntity.class)))
                .thenAnswer(new Answer<ServiceStateEntity>() {
            @Override
            public ServiceStateEntity answer(InvocationOnMock invocation) throws Throwable {
                return new ServiceStateEntity();
            }
        });
        heartBeatService.heartBeat(ServiceName.LiJiRobot.getName(), new Date(), new Date(), true, null);
        verify(serviceStateRepository).findByService(isA(String.class));
        verify(serviceStateRepository).save(isA(ServiceStateEntity.class));
        verify(serviceStateHistoryRepository).save(isA(ServiceStateHistoryEntity.class));
        verifyNoMoreInteractions(serviceStateRepository);
    }
}

## 怎样写好单元测试

### 测试的层次和组织

* 运行自动化测试的自动化构建
* 持续集成服务器
* 构建脚本
** 持续集成构建脚本
** 每日构建脚本
** 部署构建脚本

* 测试代码和构建脚本都应该在版本控制之下
* 分离集成测试和单元测试
* 测试类的组织
** 将测试映射到项目
** 将测试映射到类
    ** 一个被测类关联一个测试类
    ** 一个功能点关联一个测试类
** 将测试映射到具体的工作单元，testMethod_Senario_Behavior

### 优秀的单元测试

1. 可靠性（测试成功了，就说明产品代码没有问题，测试失败了，就说明产品代码写错了，无需怀疑是测试代码的问题）
2. 可维护性（当项目紧的时候，开发功能尚需加班加点，没有人会为不可维护的测试代码耗费精力）
3. 可读性（啥也不说了，好多学科没学好就是因为课本的可读性太差了）

（其实和优秀的产品代码是一样的标准）

*编写可靠的单元测试*

* 决定何时修改或删除测试
** 产品代码缺陷，如果测试没错确实是产品代码错误，那就修改产品代码，这正是单元测试的有用之处
** 测试代码缺陷，很让人郁闷，测试本应该是正确的（拒绝#>诧异#>调试#>接受和顿悟），感觉受到10000点伤害。。。
** 语义或API变更
** 冲突或无效的测试，多数是由于冲突的需求引起的
** 重命名或者重构测试
** 删除重复测试

* 避免测试中的逻辑
** switch，if，while，for就是逻辑
** 有逻辑就会使程序复杂，就更容易出错，而且难以命名，说明测试并不是执行了一项任务

* 只测试一个关注点（我的理解就是只测试一个逻辑分支）
* 分离单元测试和集成测试
** 集成测试运行时间长
** 依赖外部因素，不稳定

* 推动代码审查

*编写可维护的单元测试*

* 只测试公共契约（也就是共有方法），如果有私有方法一定需要测试，试试一下方式
** 使方法成为公共方法
** 把方法抽取到新类（逻辑独立，而且使用的对象状态只和本方法有关，可以考虑）
** 使方法成为静态方法

* 去除重复代码
** 使用辅助方法（factory method）
** 以可维护的方式使用setup方法，一下是一些常见的拙劣用法
*** 在setup中初始化只有部分测试使用的对象
*** 冗长难懂
*** 在setup中准备伪对象（一般伪对象之和某个特定测试相关联）
*** 可以考虑不使用setup方法，有时可以考虑参数化测试来替代
* 实施测试隔离，测试之间不应该有依赖关系，一下是常见反模式举例
** 强制测试执行顺序
** 测试调用测试
** 内存共享状态损坏（测试结束后忘记回滚状态）
** 外部共享状态损坏
* 避免对不同关注点多次断言
* 对象比较，对于一次需要比较一个对象的多个状态的情况，推荐使用一下方式
** 重写equals方法
** 重写toString方法
* 避免过度指定
** 指定纯内部行为
** 在需要存根时使用模拟对象
** 不必要的顺序指定或精度匹配（例如，对集合中元素的顺序做断言）

**编写可读的测试**

* 命名单元测试
** methodUnderTest_Scenario_Behavior
* 命名变量
** 明确表示该变量的意义，例如correctResult而不是num
* 有意义的断言
** 不要重复框架可以输出的信息
** 如果没什么可说的，就别说
* 断言和操作分离，也就是不要把断言和操作写在一行里
** assertEquals(result, calcResult());
* 不要乱用setUp和tearDown


## 设计和流程

### 在组织中引入单元测试

* 逐步成为变革的倡导者（略）
* 成功之道（略）
* 失败原因（略）
* 影响因素（略）
* 质疑和回答
** 单元测试会给现有的流程增加多少时间
*** 相比没有单元测试，找bug和修改bug的时间会更长，而且对自己的修改是否引入其他bug不能明确的了解
** 是否会抢了QA的饭碗
*** 单元测试只会让开发更有信心，代码质量更好，但是没有问题是不可能的
** 证明单元测试有效的方法（ JCoverage）
** 单元测试有用的证据
*** 无需多言，这么多人在推荐，不可能所有人都是傻子
** QA部门还是能找到缺陷的原因（集成测试，验收测试）
** 大量没有测试的代码，何处开始（从问题最多的组件开始）
** 代码都调试通过了，为什么还需要测试
*** 你怎么知道别人的代码有没有问题？
*** 别人怎么知道你的代码有没有问题？
*** 你怎么知道修改后没有破坏原有功能？

大部分缺陷不是来自代码本身，而是由人们之间的误解，不断变化的需求以及领域知识的缺少导致的

## 遗留代码（略）

## 设计与可测试性（略）
