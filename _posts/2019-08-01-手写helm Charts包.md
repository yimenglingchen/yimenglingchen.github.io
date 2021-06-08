---

layout: post

title:  手写helm Charts包

tag: CICD

---
# 手写helm Charts包

helm中Charts作为其软件包格式，如何编写，如何发布时开发者使用helm需要掌握的一个内容。

## 1、Charts包文件框架（jenkins-helm包为例）

```
│  .helmignore
│  Chart.yaml
│  OWNERS
│  README.md
│  values.yaml
└─templates
    │  config.yaml
    │  home-pvc.yaml
    │  jcasc_config.yaml
    │  jenkins-agent-svc.yaml
    │  jenkins-backup-cronjob.yaml
    │  jenkins-backup-rbac.yaml
    │  jenkins-master-deployment.yaml
    │  jenkins-master-ingress.yaml
    │  jenkins-master-networkpolicy.yaml
    │  jenkins-master-svc.yaml
    │  jobs.yaml
    │  NOTES.txt
    │  rbac.yaml
    │  secret.yaml
    │  service-account.yaml
    │  _helpers.tpl
    │
    └─tests
            jenkins-test.yaml
            test-config.yaml
```

​	解释一下每一块中内容的作用：

### 1.1、templates模块

​		这个模块下放置的是所有k8s资源编排文件，但都是模板，内部使用默认变量或者`{{Values.zz}}`写法，原因是这些值在不同的环境下部署是需要发现变化的，那么就把这些内容定义成变量，在其他地方写值，部署的时候回渲染成真实的值。

​	 举一个简单的例子：

```
{{ if .Values.rbac.install }}
{{- $serviceName := include "jenkins.fullname" . -}}
{{- if .Capabilities.APIVersions.Has "rbac.authorization.k8s.io/v1beta1" }}
apiVersion: rbac.authorization.k8s.io/v1beta1
{{- else if .Capabilities.APIVersions.Has "rbac.authorization.k8s.io/v1alpha1" }}
apiVersion: rbac.authorization.k8s.io/v1alpha1
{{- else }}
apiVersion: rbac.authorization.k8s.io/v1
{{- end }}
kind: {{ .Values.rbac.roleBindingKind }}
metadata:
  name: {{ $serviceName }}-role-binding
  labels:
    app: {{ $serviceName }}
    chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    release: "{{ .Release.Name }}"
    heritage: "{{ .Release.Service }}"
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: {{ .Values.rbac.roleKind }}
  name: {{ .Values.rbac.roleRef }}
subjects:
- kind: ServiceAccount
  name: {{ $serviceName }}
  namespace: {{ .Release.Namespace }}
{{ end }}

```

### 1.2、values.yaml

​		这个文件中定义了所有在`templates`中k8s编排文件需要的变量的值，可以使用

```
$ helm install --dry-run --debug .
```

​    命令来测试将values中的内容赋给`templates`中的模板文件是否有异常或者不对的对方。

   此文件内容可以在安装或者个更新应用的时候，通过

```
$ helm upgrade release-name charts.tgz --set key=value
```

命令覆盖。

​	简单看一个values.yaml文件内容：

```
Master:
  Name: jenkins-master
  Image: "jenkins/jenkins"
  ImageTag: "lts"
  ImagePullPolicy: "Always"
# ImagePullSecret: jenkins
  Component: "jenkins-master"
  NumExecutors: 0
  # configAutoReload requires UseSecurity is set to true:
  UseSecurity: true
```

### 1.3、Charts.yaml 

​	此文件描述了这个应用包的信息，应用的名称，版本，charts的版本等等，可以具体看一个：

```
name: jenkins
home: https://jenkins.io/
version: 0.36.0
appVersion: lts
description: Open source continuous integration server. It supports multiple SCM tools
  including CVS, Subversion and Git. It can execute Apache Ant and Apache Maven-based
  projects as well as arbitrary scripts.
sources:
- https://github.com/jenkinsci/jenkins
- https://github.com/jenkinsci/docker-jnlp-slave
- https://github.com/nuvo/kube-tasks
- https://github.com/jenkinsci/configuration-as-code-plugin
maintainers:
- name: lachie83
  email: lachlan.evenson@microsoft.com
- name: viglesiasce
  email: viglesias@google.com
- name: maorfr
  email: maorfr@gmail.com
icon: https://wiki.jenkins-ci.org/download/attachments/2916393/logo.png
```

### 1.4、OWNERS

​		此文件描述了这个应用的作者或者是拥有者

```
approvers:
- lachie83
- viglesiasce
- maorfr
reviewers:
- lachie83
- viglesiasce
- maorfr
```

