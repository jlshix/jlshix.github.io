---
title: "gitlab ci schedules 功能初探"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - gitlab
  - ci
---

提交代码后根据情况触发流水线, 这是 gitlab-ci 的常用方式. 其实还有另外一种方式, 那就是**定时任务**.

打开项目的 `CI/CD` 菜单, 除了 `Pipelines`, `Jobs` 之外, 还有一个 `Schedules`, 即定时任务的入口.

初次进入后一片空白, 只有一个简短的[帮助链接](https://docs.gitlab.com/ee/ci/pipelines/schedules.html)
和一个 `New schedule` 按钮.

点击 `New schedule` 按钮新建一个任务, 可见有如下选项:

- `Description`: 简单描述

- `Interval Pattern`: 间隔模式, 可选择默认的每天, 每周, 每月, 也可以根据 `Cron` 语法自定义.
  五个占位符分别表示 `分时日月周`, 其中周日用 `0` 表示, 某些系统也支持 `7`.

- `Cron Timezone`: 调度时区, 选择当前时区即可

- `Target Branch`: 目标分支, 执行时读取选定分支的最新提交, 使用其 `gitlab-ci.yml` 的配置启动任务.

- `Variables`: 指定环境变量, 可以是 gitlab-ci 预置的环境变量, 也可以是自定义的环境变量.

- `Activated`: 是否设为活动


填写相关信息后就可以 `Save pipeline schedule` 了.


回到 `Schedules` 页面可以看到新建的定时任务.

- `Next Run` 属性显示下次在什么时候运行,

- `Last Pipeline` 属性显示最后一次调度是哪个 `Pipeline`. 

- 在 `Pipelines` 页面的 `Pipeline` 属性下, 带有 `Scheduled` 标签的即为调度生成的 Pipeline.
