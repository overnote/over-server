## 一 CI/CD介绍

- CI：持续继承（Continuous intergration）。新提交的代码在合并到主线之前，都需要通过编译和自动化测试流程进行验证。
- CD：持续交付（Continuous delivery）。指自动发布应用的功能。

Gitlab CI 是Gitlab提供的持续继承服务，只要在你的仓库根目录 创建一个`.gitlab-ci.yml` 文件， 并为该项目指派一个Runner，当有合并请求或者 push的时候就会触发build。  

runner默认会执行三个阶段：build、test、deploy。完成上面的步骤后，每次push代码到Git仓库， Runner就会自动开始pipeline。  

大部分项目用GitLab’s CI服务跑build测试， 开发者会很快得到反馈，知道自己是否写出了BUG。 

## 二 操作过程

#### 1.1 配置runner

#### 1.2 编写.gitlab-ci.yml

在代码的根目录创建文件`.gitlab-ci.yml`，内容格式大致如下：
```
stages:
  - style
  - test
  - deploy

job1:
  stage: style
  script:
    - echo "Start reviewing code"
    - gradle sonarqube
  tags:
    - py2.7

job2:
  stage: test
  script:
    - echo "Start reviewing code"
    - gradle sonarqube
  tags:
    - mytest
```
