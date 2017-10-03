---
title: Jenkins 使用groovy + job-dsl 创建并触发job
date: 2017-09-30 21:05:13 +0800
layout: post
categories:
- Groovy
- Jenkins
tags:
- Groovy
- Jenkins
---

# Jenkins: 使用groovy + job-dsl 创建并触发job

## 背景：

我们的 Automation 测试脚本需要在10个不同语言的机器上跑，本地化测试产品。
我们用Jenkins启动测试执行，需要同时向1-10个机器发送文件和脚本运行命令，现有Jenkins的配置采用master/slave 机制如下：

**Dispatcher Job**:

1。 启用参数化，每个语言机器传递两个参数:

	- Choice Parameter: slave Node 名，比如这里的NodeLabel1,表示第一台目标机器

![Node Name](http://i.imgur.com/MU2lfKi.png)

	- File Parameter: 要发送给目标机器的文件

![File Name](http://i.imgur.com/eEUbWOY.png)

2。	在Build Steps中trigger seed job:

![](http://i.imgur.com/yu37Pt1.png)

**Seed Job**: 设置具体的运行脚本。

## 现有设计的问题：

运行很流畅，但是维护不方便

- 每个Slave Node都添加一个Choice Paramter 和 File Parameter，
- Build Step中也要添加10次trigger seed job,
- Steps中如有一处脚本改动，需重复改动10次
- 如果有新添加测试机器，需要在Parameters中手动添加新Node, 在Steps中也要添加。

## 改进需求：

自动获取所有Node列表，减少不必要的手动增添Node。

在Build Steps中，通过脚本trigger seed job,这样避免重复设置10次trigger step, 提高可重复性(Reusability)。

## 改进方案：

使用Groovy脚本，安装Jenkins插件：

- [Active Choices](https://wiki.jenkins-ci.org/display/JENKINS/Active+Choices+Plugin): 一个蛮好的参数化插件，可插入Groovy脚本和文件（集成Scriptlet插件）。

- [Scriptler](http://wiki.jenkins-ci.org/display/JENKINS/Scriptler+Plugin)：Groovy脚本插件，可以集中管理脚本文件。

- [Job-DSL](https://wiki.jenkins-ci.org/display/JENKINS/Job+DSL+Plugin): 一个很好的利用DSL设置和运行Job的插件。

## 具体实施：

### 准备global scripts，获取全部节点，用的是scriptler插件。

1。 在Jenkins->Scriptler中创建getAllNodeByProject.groovy文件，用来获取所有的Node机器。需要的话也可以加入一些筛选条件，比如在线与否。

scriptler中支持参数传递，这里的project就是预先设置的参数，可在job调用该脚本时进行设置。

```

	def list = jenkins.model.Jenkins.instance.slaves.findAll{
	  if (it.name.contains(project)) it.name
	}.collect{it.name}
	list.add(0,"None")
	return list
```

![](http://i.imgur.com/bnp7n0G.png)

2。 在Jenkins->Scriptler中创建getRuntimeNodes.groovy文件，用来获取dispatcher job在运行时指定的Node机器。

```

				import hudson.model.*
                def list =""
                def runable=false
                def build = Thread.currentThread()?.executable
                // get the selected node parameter value at runtime
                def nodes = build?.actions.find { it instanceof ParametersAction }?.parameters.collect{
                    (it instanceof StringParameterValue)?it.value:""
                } as List
                println "node is $nodes"
                nodes.unique().removeAll(["None"])
                nodes.remove(" ")
				nodes.remove("")
                if (nodes!=[]) {
                    list=nodes.join(",").toString()
                }
                println "list is:  $list"
                if (list!=" " && list!="") runable=true
                println "runable is $runable"
                build.addAction( new ParametersAction([new StringParameterValue("runtimeNodes",list)]))
                build.addAction( new ParametersAction([new BooleanParameterValue("runable",runable)]))
```

运行结果是，在dispatcher job runtime会新建两个参数：

- String参数 runtimeNodes: 存放运行时获取用户指定的Node 机器列表，用于后续在这些node 机器上分别trigger seed job

- 布尔型参数 runable：如果没有指定任何Node，则将runable设为false, 即不会trigger seed job

### 创建main,dispatcher和seed jobs

手动创建一个名为main的 job, 用来动态生成dispatcher and seed jobs.

#### 在main中用job-dsl创建dispatcher job

在main job中，插入一个Process Job DSLs 构建步骤, 下面的代码会在运行时创建一个名为dispatcher的job:

- 为每个language node （“Client_CHS","Client_JPN") 设置了active choice node 参数和File参数。
- 创建一个conditional step, 如果符合条件，则trigger seed job，并将用户运行时指定的所有node以NodeListFactory参数方式传给seed job。



```

				import hudson.model.*

				// get all available nodes if its name contains "client" - specific for our project use;
                def nodes = hudson.model.Hudson.instance.slaves.findAll{
                    if (it.name.contains("Client")) it.name
                }.collect{it.name}
                nodes.add(0,"None")

				//def nodes = ["Client_JPN","Client_CHS"]

                defineJob(nodes-["None"],"dispatcher") // setup the dispatcher job

                // method to define a dispatcher job
                def defineJob(nodes,jobName) {
                    return job(jobName) {
                        def list =[]
                        nodes.each {
                            def node =it
                            parameters {
                                activeChoiceParam(node) {
                                    description('Allows user choose from choices')
                                    filterable()
                                    choiceType('SINGLE_SELECT')
                                    scriptlerScript('getAllNodesByProject.groovy') {
                                        parameter('project', 'Client')
                                    }
                                }
                                fileParam(node+'_File', 'Select test case file to upload')
                            }
                        }

                        steps {
                            //systemGroovyScriptFile('${JENKINS_HOME}\\scriptler\\scripts\\getRuntimeNodes.groovy') {
                            systemGroovyCommand(readFileFromWorkspace("${JENKINS_HOME}\\scriptler\\scripts\\getRuntimeNodes.groovy")){
                            }
                            conditionalSteps {
                                condition {
                                    booleanCondition("\${runable}")
                                }

                                steps{ downstreamParameterized {
                                    trigger('seed') {
                                        /* parameters{
                                          println node
                                         nodeLabelBuildParameter {
                                            name(node)
                                            nodeLabel(node)
                                          }*/
                                        parameterFactories {
                                            nodeListBuildParameterFactory {

                                                // The name of the parameter to set.
                                                name("test")
                                                // A comma separated list of nodes to execute the job on.
                                                nodeListString("\${runtimeNodes}")

                                            }
                                            //booleanParam("f",true) /
                                        }
                                    }
                                }
                                }
                            }
                        }
                    }
                }
```

现在运行 main job，成功后你会看到 dispatcher job 和 seed job 生成了

![](http://i.imgur.com/BwHDNDn.png)

![](http://i.imgur.com/fSAXXyO.png)

进入 dispatcher job, 选择 build with parameters,会看到我们想要的参数都列出来了。

![](http://i.imgur.com/CnFpiID.png)

#### 在main中用job-dsl创建 seed job

打开main job，再次插入一个Process Job DSLs 构建步骤, 下面的代码会在运行时创建一个名为 seed 的job, seed job 会 触发实际执行的自动化脚本，这里为简单起见，我们让它执行一行命令。这里用到了Node and Label parameter 插件

```

	job("seed") {
  		concurrentBuild()
  			/*parameters{
              nodeParam("node") {
                        description('Select Test Node')
                        defaultNodes(["Test Node"])
                        allowedNodes(["Test Node"])
                    }*/

              steps {
                  batchFile('echo Hello World!')
              }
            }

```

现在再运行一次main job, 回到job 列表中，可以看到dispatcher和seed都列在其中了。

![](http://i.imgur.com/zvawqgK.png)

点击进入seed job, 便可以看到上面定义的batch command step了。

![](http://i.imgur.com/EtC33Iv.png)

现在，dispatcher和seed job都就位了，接下来我们需要修改dispatcher，让它运行时trigger seed job.

### 测试：允许dispatcher job

运行的时候我们手动或自动触发 dispatcher job.

进入dispatcher job, 选择 build with parameters,

- 将所有Node 都设为"None", 运行结果显示，seed job 没有trigger。成功
-
![](http://i.imgur.com/QYX4jk3.png)

- 指定一个或多个Node（不为None),运行结果显示，seed job在指定Nodes上触发了。成功

![](http://i.imgur.com/HsfjJ6e.png)

上述操作还没有将File Parameter的使用加上去，后续再补充。欢迎一起讨论！
