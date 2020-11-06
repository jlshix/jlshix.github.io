---
title: "使用 gitlab-ci 进行自动化测试与部署"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - python
  - gitlab
  - gitlab-ci
  - CI/CD
---

之前的文章分别讲了使用 `pylint` 对代码进行静态检查, 使用 `pytest` 做测试和 `coverage` 
看测试覆盖率. 这三者用于保证代码的质量. 

当我们的代码通过了全部测试且测试覆盖率也达到一定程度后, 就可以先把代码部署到测试环境了.
在测试环境再进行一系列的测试之后完成上线要求即可部署到生产环境.

至于应用的部署, 现在使用 `docker` 打包镜像并部署到 `Kubernetes`(`k8s`) 集群已经几乎成为了
事实上的标准. 如果还没上车可以先了解一下这两个非常重要的工具.

而持续集成工具, 可以用百花齐放来形容, 无论是老牌的 `Jenkins`, github 上使用更多的 `Travis CI`,
新兴的 `drone.io`, 完全使用 python 实现的 `buildbot` 等, 都有各自的特点.

而我选择了 `gitlab-ci`, 很大原因是公司使用 `gitlab` 做代码托管, 而 `gitlab-ci` 已经整合到
`gitlab` 中, 开箱即用, 功能异常强大但又非常易用, 可以满足目前所有的需求.

持续集成工具大都具有类似的结构:

- 一个 Master 监听远程代码库的变动, 一旦发生变动, 根据预先的配置进行操作
- 一到多个 Slave, 接收并执行 Master 分配的工作.

对于 `gitlab-ci` 来讲, Master 即为 gitlab 本身. Slave 则是我们需要手动进行简单配置的
`gitlab-runner`.

gitlab 与 runner 无需部署在同一台机器上, 二者之间只要能保持长连接即可.

## gitlab-ci 概况

在配置好 runner 之后, 只需要一份 `.gitlab-ci.yml` 作为配置文件, gitlab 就会自动执行
对应任务.

`gitlab-ci` 的功能非常强大, [官方文档](https://docs.gitlab.com/ee/ci/)详述了各种功能,
我们只使用其中的一个很小的子集用于模型流程的构建. 当预先的项目配置完成后, 只需要阅读
`.gitlab-ci.yml`的[官方配置指南](https://docs.gitlab.com/ee/ci/yaml/README.html)
的部分内容, 另外还有一份[较旧版本的中文翻译](https://www.cnblogs.com/wyt007/p/12228441.html).

`job`, 即任务, 是配置文件的基本组成元素, 其 `stage` 属性指定其所属的阶段, `script` 属性指定
需要执行的命令, 也就是任务的具体内容.

使用 `git push` 将代码推送到 gitlab 时, gitlab-ci 会解析项目根目录下的 `.gitlab-ci.yml`,
若满足条件则将任务分配给 `job` 的 `tags` 属性指定的 runner 执行任务, 每次执行任务都拉取或
挂载项目代码并执行 `scripts` 属性中脚本的内容.

一次流程分为多个 `stage`, 每个 stage 可包含多个 job.

每个 stage 的 job 在不指定依赖关系的情况下并行执行, 不同 stage 的 job 按照 stage 的顺序
串行执行.



## 安装与配置

首先进入项目首页, 点击左侧选项的 `Settings -> CI/CD`, 找到 `Runners`, 
点击右侧的 Expand, 可以看到 Runners 的基础介绍.

在 `Specific Runners` 下的 `Set up a specific Runner manually` 可以看到我们需要先
安装 gitlab runner, 然后登记 url 和 token.

在 [安装runner](https://docs.gitlab.com/runner/install/) 的文档中我们直接跳到
[Install as a docker service](https://docs.gitlab.com/runner/install/docker.html).

以在 ubuntu 上执行为例, 我们需要进行以下几步:

- 拉取 runner 镜像: `docker pull gitlab/gitlab-runner:ubuntu`

- 启动容器:

```shell
docker run -d --name=runner --restart=always \
  -v ~/data/gitlab-runner:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:ubuntu

```

需要挂载两处, 一是配置文件目录, 用于修改配置文件; 二是宿主机的 `docker.sock`, 可以让
runner 直接使用宿主机的 docker 启动任务的容器, 避免 `docker in docker` 的情况.

- 执行 `docker exec -it ssr bash` 进入容器, 执行 `gitlab-ci-multi-runner --help`
可以看到帮助信息, 执行 `gitlab-ci-multi-runner register` 可以交互式地注册一个 runner:

```shell
root@109daa908892:/# gitlab-ci-multi-runner register
Runtime platform                                    arch=amd64 os=linux pid=56 revision=a998cacd version=13.2.2
Running in system-mode.

Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
# 注: gitlab 的地址
https://gitlab.com/

Please enter the gitlab-ci token for this runner:
# 注: 项目页面显示的 token
d1ieZTAcHyCmqNbztCz8

Please enter the gitlab-ci description for this runner:
# 注: 简短的描述
[109daa908892]: first runner for ci/cd

Please enter the gitlab-ci tags for this runner (comma separated):
# 注: 用于在 job 中使用 `tags` 属性指定此 runner 执行任务
runner1

Registering runner... succeeded                     runner=d1ieZTAc
Please enter the executor: custom, docker-ssh, parallels, virtualbox, docker-ssh+machine, docker, shell, ssh, docker+machine, kubernetes:
# 注: 以 docker 的方式执行
docker

Please enter the default Docker image (e.g. ruby:2.6):
# 注: job 在不指定 `image` 属性时使用的 docker image
python:3.7.3

Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

- 进入项目CI/CD配置页, 即 `Settings` -> `CI/CD` -> `Runners`
可以看到配置的 runner 已经在 `Runners activated for this project` 下了.

同时我们可以在容器中查看已生成的配置文件: `cat /etc/gitlab-runner/config.toml`

关于 toml 的语法, 参见[Learn X in Y minutes Where X=toml](https://learnxinyminutes.com/docs/toml/)


## 使用 docker in docker

如果我们需要在 docker 中使用 docker, 除了需要安装 docker 之外, 还需要挂载 docker.sock.

对于 Job Container 的镜像来讲, 需要配置 config.toml 完成 docker.sock 的挂载.

在 `runners -> runners.docker -> volumes` 中添加一项 `"/var/run/docker.sock:/var/run/docker.sock"`:

```toml
volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
```

保存即可.


## 编写 `.gitlab-ci.yml`

`gitlab-ci` 使用 `yaml` 管理项目配置, 放置于项目根目录, 定义项目应如何构建, 默认文件名为
`.gitlab-ci.yml`. 

如果不熟悉 YAML, 可以参见以下两篇文章:
- [YAML 语言教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2016/07/yaml.html)
- [Learn X in Y minutes where X=YAML](https://learnxinyminutes.com/docs/zh-cn/yaml-cn/)

配置文件中定义了一系列的 job 及其属性, 每个 job 至少要包含 script 属性:

```yaml
job1:
  script:
    - echo "hello gitlab ci"

job2:
  script:
    - export PYTHONPATH=`pwd`
```


未定义的其他属性都有各自的默认值, 如默认的 `stage` 为 `test`.


## 一个实例

现在我们既有了 `pylint`, `pytest`, `coverage` 作为代码质量保证工具, 又有一个可以将所有流程
串联起来的 `gitlab-ci`, 那么我们就可以编写自己项目的 `.gitlab-ci.yml` 了.

首先作为一个工程项目可以分为三个部分:

- 测试, 包括代码静态检查(pylint), 执行单元测试(pytest), 查看覆盖率(coverage)
- 构建, 使用源代码构建 docker image
- 发布, 将新版代码发布到开发环境或生产环境.

`.gitlab-ci.yml` 如下:

```yml
default:
  # 使用测试镜像作为默认镜像, 包含本项目代码所有的 requirements 以及 pylint, pytest, coverage
  image: project_name:test

variables:
  # 生成的镜像名称, 版本以此次提交的版本号
  IMAGE: project_name:${CI_COMMIT_SHORT_SHA}


stages:
  # 分为测试, 构建和部署三个阶段
  - test
  - build
  - deploy

lint:
  stage: test
  script:
    # 使用自定义的规则进行代码检查, 得分低于 8 分则失败, 不再执行之后的 stage
    - pylint --rcfile=.pylintrc --fail-under=8 src
  tags:
    # 使用标签为 runner1 的 runner 执行
    - runner1

coverage:
  stage: test
  script:
    # 使用 coverage 执行 pytest 单元测试, 只统计当前文件夹下的代码覆盖情况, 生成报告
    - coverage run --source=. -m pytest
    - coverage report -m
  tags:
    - runner1


build:
  # 使用官方的 docker 镜像进行构建和推送镜像操作
  image: docker:19.03
  stage: build
  script:
    - docker build . -f deploy/Dockerfiles/Dockerfile -t ${IMAGE}
    - docker push ${IMAGE}
  only:
    # 只在主分支执行此任务
    - master
  tags:
    - runner1

deploy:
  # 基于官方的 docker 镜像安装 kubectl 操作集群
  image: docker:19.03-k8s
  stage: deploy
  script:
    # 更换已有 deploy 的镜像名称并记录
    - kubectl -n ns1 set image deploy/project-name svc=${IMAGE} --record
  only:
    - master
  tags:
    - runner1
```

编写完毕后提交推送, gitlab 就会自动执行任务, 实时状态显示在项目页面的 CI/CD 选项卡中.
可以选中某个 Job 查看其实时输出. 此外还有更多功能.

执行结果如图

![执行结果](/img/in-post/2020-11-06-01.jpg)

## 总结

以上就是 `gitlab-ci` 最基本的用法了, 官方提供了非常详尽的文档, 可以在遇到更精细的需求时
从官方文档中寻找解决方案, 实现 pipeline 的高度定制化, 满足团队开发的需求.