---
title: Jenkins Groovy
date: 2017-09-30 21:05:13 +0800
layout: post
categories:
- Groovy
- Jenkins
tags:
- Groovy
- Jenkins
---

# 动态创建Master/slave jobs, trigger slaves by passing parameters. Using groovy


任务: 动态创建Master/slave jobs, trigger slaves by passing parameters. Using groovy

准备：Jenkins Server with below plugins

Active Choice
Scriptler

1. Create a maven groovy project called "jenkins"

import [job-dsl dependency](https://mvnrepository.com/artifact/org.jenkins-ci.plugins/job-dsl)

```

	<!-- https://mvnrepository.com/artifact/org.jenkins-ci.plugins/job-dsl -->
	<dependency>
	    <groupId>org.jenkins-ci.plugins</groupId>
	    <artifactId>job-dsl</artifactId>
	    <version>1.60</version>
	</dependency>

```

and hudson-core dependency

```

<!-- https://mvnrepository.com/artifact/org.eclipse.hudson.main/hudson-core -->
    <dependency>
        <groupId>org.eclipse.hudson.main</groupId>
        <artifactId>hudson-core</artifactId>
        <version>3.0.0-M2</version>
    </dependency>

```
2. Create a groovy class called DefineJobs. 用来创建job，可定义各种参数，比如Active Choice Parameters, File Parameters


3.
