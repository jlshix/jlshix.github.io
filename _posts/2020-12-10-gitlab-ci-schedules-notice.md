---
title: "gitlab ci schedules 时间不准的问题"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - gitlab
  - ci
---

上篇文章提到可以使用定时任务的方式触发 gitlab-ci 的 pipeline.
但在实际情况下, 出现了并没有严格按照设定的时间执行任务的情况.


在我司内网的 gitlab 某项目下, 设定一个每天上午八点执行的定时任务, 发现其 `Next Run` 是第二天的 `8:19am`. 因为此定时任务也无需太精确的时间, 就等第二天看看情况.

第二天果然是八点十九分才开始执行, 而非八点整.

于是打开 google 搜索 "gitlab schedules next run not correct".
果然发现了几个 issue:

- [#255329 scheduler shows wrong time for next run](https://gitlab.com/gitlab-org/gitlab/-/issues/255329)
- [#18757 Cron triggers builds not on correct time](https://gitlab.com/gitlab-org/gitlab/-/issues/18757)
- [#22978 Scheduling Pipelines Wrong Time of execution?](https://gitlab.com/gitlab-org/gitlab/-/issues/22978)


根据提供的信息来看都是有一样的表现: gitlab 有个执行计划任务(后称用户计划任务)的计划任务(后称系统计划任务). 所以:

- 由于 `Cron` 的最小粒度是分, 则系统计划任务每分钟执行一次才可以保证用户计划任务准时执行.

- 若系统计划任务设成每小时执行一次, 那么所有的用户计划任务都要到系统计划任务执行时才会得到调度.

果然, 在 [Schedules高级配置](https://docs.gitlab.com/ee/ci/pipelines/schedules.html#advanced-configuration)
中提到, 这个系统计划任务由 `Sidekiq` 执行.

开头简短地说明了这个问题:

> The pipelines are not executed exactly on schedule because schedules are handled by Sidekiq, which runs according to its interval.

> For example, only two pipelines are created per day if:

> You set a schedule to create a pipeline every minute (* * * * *).
> The Sidekiq worker runs on 00:00 and 12:00 every day (0 */12 * * *).

`Sidekiq` 的配置写在[Gitlab CI/CD配置](https://docs.gitlab.com/ee/user/gitlab_com/index.html#gitlab-cicd). 

我司的 gitlab 对应页面上 `Scheduled Pipeline Cron` 的 `Default` 值赫然写着 `19 * * * *`.
这下破案了.


如果要修改这个值, 则需要:

> To change the Sidekiq worker’s frequency:

> 1. Edit the gitlab_rails['pipeline_schedule_worker_cron'] value in your instance’s gitlab.rb file.
> 2. Reconfigure GitLab for the changes to take effect.
