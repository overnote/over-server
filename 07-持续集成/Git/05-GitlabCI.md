## 一 操作过程

### 1.1 配置runner

### 1.2 编写.gitlab-ci.yml

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
