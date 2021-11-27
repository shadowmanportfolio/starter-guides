# 实验 - 面向开发者的OpenShift入门

* [概述](#概述)
* [部署Workshop](#部署Workshop)
  * [在Red Hat产品演示系统上部署](#在Red-Hat产品演示系统上部署)
  * [部署到OpenShift集群](#部署到OpenShift集群)
* [运行Workshop](#运行Workshop)
* [仅部署实验指南](#仅部署实验指南)
* [开发](#开发)
* [删除Workshop](#删除Workshop)

## 概述

| | |
--- | ---
| 观众体验水平 | 入门级 |
| 支持用户数 | 每个集群最多100个 |
| 平均完成时间 | 90 分钟 |

本课程旨在从开发人员的角度介绍如何使用OpenShift。

容器是一种将应用程序及其所有依赖打包的标准化方式，以简化部署并加快交付速度。与虚拟机不同，容器不会绑定到操作系统。只有应用程序代码、运行时、库和设置被打包在容器中。因此，与虚拟机相比，容器更轻量、可移植和高效。

对于希望启动项目的开发人员来说，OpenShift通过流畅的工作流程和经过验证的集成实现了高效的应用程序开发。

### 目标

* 使用OpenShift命令行客户端和web控制台。
* 使用预先存在的容器镜像部署应用程序。
* 使用应用程序标签来识别组件部件。
* 扩展应用程序以处理网络流量。
* 向集群外的用户公开应用程序。
* 查看和处理应用程序生成的日志。
* 访问应用程序容器并与之交互。
* 允许其他用户在您的应用程序上进行协作。
* 从Git存储库中的源代码部署应用程序。
* 从OpenShift开发人员目录部署数据库。
* 配置应用程序，使其能够访问数据库。
* 设置webhook以启用自动应用程序构建。
* 还可能涉及与正在部署的应用程序使用的特定编程语言相关的其他主题。

Workshop支持4种编程语言练习：

* Java
* Node.js
* Python
* .NET C#

### 使用组件

整个Workshop包括以下几个部分:

* Etherpad - 用户可以申请用户名
* 每个用户的GOGS服务器和GOGS存储库
* Nexus - 目前仅由Java版本的workshop使用
* OCP 维护视图 - 集群可视化[维护视图](https://github.com/hjacobs/kube-ops-view) 实例
* 从这个存储库初始化的Homeroom workshop应用程序实例和4个初学者实验指南(Java, Node.js, Python, PHP)。

可以在[这里](http://lab-getting-started-ocp4-starter-guide.apps.osd4-demo.u6k6.p1.openshiftapps.com/workshop/common-workshop-summary)找到Java实验指南的一个示例。

## 部署Workshop

本workshop可在[红帽产品演示系统(RHPDS)](https://rhpds.redhat.com)进行部署。

### 在Red Hat产品演示系统上部署

登录到RHPDS后，突出显示 **Services** 侧栏，并选择 **Catalogs** 菜单。Workshop可在 **Workshops** 文件夹下的目录中找到，名为 **OCP4 - Getting Started Workshop**。配置表单中的 `City or Customer` 字段将用于创建一个在集群URL中对参与者可见的GUID。

按照[运行Workshop](#运行Workshop)部分的说明开始Workshop。

**Note:** 通过RHPDS部署OCP4集群最多需要75分钟。

### 部署到OpenShift集群

如果您想手动部署它，您可以申请基本的OpenShift 4.8 workshop，并通过下面的说明部署入门workshop。

**先决条件**

* 来自[红帽产品演示系统(RHPDS)](https://rhpds.redhat.com)的OpenShift 4.8 Workshop集群。该集群在 **Workshops** 文件夹的目录中可用，命名为 **OpenShift 4.8 Workshop**。
* 在OpenShift 4.8 Workshop集群上的所有命名空间安装 **OpenShift Pipelines Operator** 。

[AgnosticD](https://github.com/redhat-cop/agnosticd) 用于部署Workshop，它提供了一个用于构建和配置应用程序环境的部署基础设施。

1. 首先，使用 `oc login` 命令，登录到您想部署workshop的OpenShift集群。您需要以集群管理权限登录。

2. 接下来，克隆AgnosticD存储库(或者你的分支，如果你正在进行更改)：

```
git clone https://github.com/redhat-cop/agnosticd
```

3. 在终端中，cd到agnosticd存储库中:

```
cd agnosticd
```

Python头文件(Python.h)是必需的。在Mac OS X上，如果你已经用Homebrew安装了Python，那么你应该已经有了它。

在Fedora:

```bash
sudo dnf install python3-devel
```

4. 安装 Virtual Env:

```bash
python3 -mvenv ~/virtualenv/ansible2.9-python3.6-2021-01-22
. ~/virtualenv/ansible2.9-python3.6-2021-01-22/bin/activate
 pip install -r https://raw.githubusercontent.com/redhat-cop/agnosticd/development/tools/virtualenvs/ansible2.9-python3.6-2021-01-22.txt
```

这将使您进入一个启用了AgnosticD的虚拟env的bash shell。在此环境中，您将能够运行或测试AgnosticD工作负载。

5. cd 进入 ansible 目录：

```
cd ansible
```

6. 设置环境 GUID
```
GUID=<YOUR_GUID>
```
7. 运行以下脚本以部署初学者workshop的所有组件。

更改 `num_users` 和 `user_count` 的值，以匹配您希望为workshop提供的用户数量。(注意：这些值必须是相同的，即如果你想为你的实验提供20个用户，设置 `"num_users": 20, "user_count": 20`)。
目标主机和ocp用户名可以保留为默认值，也可以根据需要更改。

```
TARGET_HOST=localhost
ocp_username=opentlc-mgr
# WORKLOAD SPECIFICS
WORKSHOP_PROJECT=labs
workloads=("ocp-workload-etherpad" \
           "ocp-workload-gogs" \
           "ocp4-workload-nexus-operator" \
           "ocp-workload-gogs-load-repository" \
           "ocp4-workload-homeroomlab-starter-guides")

for WORKLOAD in ${workloads[@]}
do
  ansible-playbook -c local -i ${TARGET_HOST}, configs/ocp-workloads/ocp-workload.yml \
      -e ansible_python_interpreter=python \
      -e ocp_workload=${WORKLOAD} \
      -e guid=${GUID} \
      -e project_name=${WORKSHOP_PROJECT} \
      -e etherpad_project=${WORKSHOP_PROJECT} \
      -e gogs_project=${WORKSHOP_PROJECT} \
      -e ocp4_workload_nexus_operator_project=${WORKSHOP_PROJECT} \
      -e project_name=${WORKSHOP_PROJECT} \
      -e ocp_username=${ocp_username} \
      --extra-vars '{"num_users": 5, "user_count": 5, "ACTION": "create"}'
done
```

## 运行Workshop

### 参与者开始 Workshop

部署完成后，导航到OpenShift控制台。如果您使用RHPDS OCP4 - Getting Started Workshop来提供环境，则在部署完成后，您将收到学生和管理员登录凭证的电子邮件。

在OpenShift控制台的左侧菜单中，选择Networking -> Routes，并将页面顶部的项目更改为 `labs`。

* `etherpad` 路由用于Etherpad部署。在路由的末尾添加 `/p/workshop` ，并与实验室参与者共享该URL，以便他们可以声明用户名。
* `homeroom` 路由是启动workshop入口的路由。在实验室参与者声明用户名后，将这个URL给他们。

如果您需要为任何路由提供直接URL, URL的结构如下所示。

* 一个GUID[已经创建](#在Red-Hat产品演示系统上部署)作为您RHPDS实验室的自定义属性。这个GUID用于集群域名的两个部分:基本域和集群名称。
* 路由url的格式是 
**ROUTE-NAME**-labs.apps.cluster-**GUID**.**GUID**.example.opentlc.com  
例如，如果你的GUID是 `abc-1234`，而路由名称是 `myroute`，路由URL将是http://`myroute`-lab.apps.cluster-`abc-1234`.`abc-1234`.example.opentlc.com

## 仅部署实验指南

**Note**: 对于本workshop，您通常希望按照上面的说明部署完整的workshop。通常情况下，只有当您正在对实验指南内容进行更改并希望快速验证和查看更改时，才会部署实验指南。

1. 要仅部署实验指南，请首先将这个Git存储库(或者您的分支，如果您正在进行更改的话)克隆到您自己的机器上。使用命令:

```
git clone --recurse-submodules https://github.com/shadowmanportfolio/starter-guides.git
```

 ``--recurse-submodules`` 选项确保Git子模块被检出。如果忘记使用此选项，在克隆存储库之后，运行:

```
git submodule update --recursive --remote
```

2. 接下来，在OpenShift中创建一个将要部署工作坊的项目。必须以集群管理员身份登录才能部署指南。

```
oc new-project workshops
```

3. 从Git仓库的顶层，现在运行：

```
.workshop/scripts/deploy-spawner.sh
```

部署的名称为 ``lab-getting-started``。

4. 您可以通过运行以下命令来确定访问Workshop的URL的主机名：

```
oc get route lab-getting-started-spawner
```

当访问workshop的URL时，将提示您输入用户名和密码。使用您的电子邮件地址或其他一些用户名的唯一标识符。这只用于确保您得到一个独特的会话，并可以在需要时从不同的浏览器或计算机附加到相同的会话。您必须提供的密码是 ``openshift``。

## 开发

上面创建的部署将使用来自[Quay.io](https://quay.io/)的镜像，这是一个基于储存库的 ``ocp-4.8`` 分支的容器自动化平台。

要更改研讨会内容并对其进行测试，请在Git存储库中编辑文件，然后运行:

```
.workshop/scripts/build-workshop.sh
```

这将替换活动部署使用的现有镜像。

如果您正在运行Workshop的现有实例，请从研讨会环境仪表板的菜单右上角选择 "Restart Workshop" 。

当您对更改感到满意时，将它们推回远程Git存储库。

## 删除Workshop

要删除spawner和任何活动的会话，包括项目，运行:

```
.workshop/scripts/delete-spawner.sh
```

要删除workshop镜像的构建配置，请运行以下命令:

```
.workshop/scripts/delete-workshop.sh
```

## 下一步

由于Homeroom是EOL，新的开发现在基于[Antora](https://antora.org)。可以在任何OCP集群上运行的独立版本在[这里](https://github.com/redhat-scholars/openshift-starter-guides)。
