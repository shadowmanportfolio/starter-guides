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

* An OpenShift 4.8 Workshop cluster from [Red Hat Product Demo System (RHPDS)](https://rhpds.redhat.com). This cluster is available in the catalog in the **Workshops** folder and is named **OpenShift 4.8 Workshop**.
* Install the **OpenShift Pipelines Operator** onto this OpenShift 4.8 Workshop cluster in all namespaces.

[AgnosticD](https://github.com/redhat-cop/agnosticd) is used to deploy the workshop, which provides a deploying infrastructure to build and configure application environments.

1. First, using the `oc login` command, log into the OpenShift cluster where you want to deploy the workshop. You need to log in with cluster admin permissions.

2. Next, clone the AgnosticD repository (or your fork of it, if you are making changes):

```
git clone https://github.com/redhat-cop/agnosticd
```

3. In your terminal, cd into the agnosticd repository:

```
cd agnosticd
```

Python headers (Python.h) are required. On Mac OS X you should have it already in place if you have installed Python with Homebrew.

On Fedora:

```bash
sudo dnf install python3-dev
```

4. Setup Virtual Env:

```bash
python3 -mvenv ~/virtualenv/ansible2.9-python3.6-2021-01-22
. ~/virtualenv/ansible2.9-python3.6-2021-01-22/bin/activate
 pip install -r https://raw.githubusercontent.com/redhat-cop/agnosticd/development/tools/virtualenvs/ansible2.9-python3.6-2021-01-22.txt
```

This will get you a bash shell into an AgnosticD enabled virtual env. In this environment, you'll be able to run or test AgnosticD workloads.

5. cd into the ansible directory:

```
cd ansible
```

6. Set you environments GUID
```
GUID=<YOUR_GUID>
```
7. Run the following script to deploy all the components of the starter workshop. 
Change the value of `num_users` and `user_count` to match the number of users you want to provision for the workshop. (Note: these values must both be the same ie if you want to provision 20 users for your lab set `"num_users": 20, "user_count": 20`).
The target hosts and ocp username can be left as the defaults or change them if needed.

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

### Starting the Workshop for Participants

Once the deployment finishes, navigate to the OpenShift Console. If you provisioned the workshop using the RHPDS OCP4 - Getting Started Workshop, you will receive a email with the student and your administrator login credentials once deployment is complete.

In the left-side menu of the OpenShift Console, go to Networking -> Routes and change the project at the top of the page to `labs`.

* The `etherpad` route is for the Etherpad deployment. Append `/p/workshop` to the end of this route and share that URL with lab participants so they can claim a username.
* The `homeroom` route is the one that launches the workshop chooser. Give this URL to lab participants after they've claimed a username.

If you need to provide the direct URL to any Routes, the structure of the URL is as follows.

* A GUID [was created](#Deploying-on-Red-Hat-Product-Demo-System) as a custom attribute of your RHPDS lab. This GUID is used in two parts of your cluster's domain name: the base domain and the cluster name.
* The Route URLs are in the form of  
**ROUTE-NAME**-labs.apps.cluster-**GUID**.**GUID**.example.opentlc.com  
If your GUID is, for example, `abc-1234`, and the route name is `myroute`, the Route URL will be http://`myroute`-lab.apps.cluster-`abc-1234`.`abc-1234`.example.opentlc.com

## 仅部署实验指南

**Note**: For this workshop, you will typically want to deploy the full workshop, per the instructions above. Deploying the lab guides only is normally only done if you are making changes to the lab guide content and want to quickly verify and view your changes.

1. To deploy the lab guides only, first clone this Git repository (or your fork of it, if you are making changes) to your own machine. Use the command:

```
git clone --recurse-submodules https://github.com/openshift-labs/starter-guides.git
```

The ``--recurse-submodules`` option ensures that Git submodules are checked out. If you forget to use this option, after having cloned the repository, run:

```
git submodule update --recursive --remote
```

2. Next create a project in OpenShift into which the workshop is to be deployed. You must be logged in as cluster admin to deploy the guides.

```
oc new-project workshops
```

3. From within the top level of the Git repository, now run:

```
.workshop/scripts/deploy-spawner.sh
```

The name of the deployment will be ``lab-getting-started``.

4. You can determine the hostname for the URL to access the workshop by running:

```
oc get route lab-getting-started
```

When the URL for the workshop is accessed you will be prompted for a user name and password. Use your email address or some other unique identifier for the user name. This is only used to ensure you get a unique session and can attach to the same session from a different browser or computer if need be. The password you must supply is ``openshift``.

## 开发

The deployment created above will use an image from [Quay.io](https://quay.io/) for this workshop, a container automation platform, based on the ``ocp-4.8`` branch of the repository.

To make changes to the workshop content and test them, edit the files in the Git repository and then run:

```
.workshop/scripts/build-workshop.sh
```

This will replace the existing image used by the active deployment.

If you are running an existing instance of the workshop select "Restart Workshop" from the menu top right of the workshop environment dashboard.

When you are happy with your changes, push them back to the remote Git repository.

## 删除Workshop

To delete the spawner and any active sessions, including projects, run:

```
.workshop/scripts/delete-spawner.sh
```

To delete the build configuration for the workshop image, run:

```
.workshop/scripts/delete-workshop.sh
```

## Next

As Homeroom is EOL, new developments are now based on [Antora](https://antora.org). A Stand-alone version to run on any OCP cluster is available [here](https://github.com/redhat-scholars/openshift-starter-guides)
