---
title: Migrate Jenkins Jobs and Nodes
date: 2018-08-10 21:05:13 +0800
layout: post
categories:
- Python
tags:
- Python
---

## Background

We need to migrate our Jenkins jobs and nodes from old Jenkins server to a new one.

Jenkins itself has a very convenient way to do this, by simply copying the job's config.xml, and a reload or restart of the Jenkins server will automatically has the job generated.

The problem is, there are many build histories for each job on the old server, we'd prefer a migrate without histories.

Thus need a way to extract all the config.xml file from each job.

## Analysis

We can iterate the Jenkins jobs folder on the old server, copy out the config.xml onto a shared network place.

Then copy all to the new Jenkins server. Find the details in the Solutions section

Another way to think, with the aim of using Jenkins API (for future development) and purely out of learning purpose, we can make the solution a bit more complicated, this will be discussed in the Go Further section

## Solution

1. define a function to extract specified config.xml file from the source folder. This folder is like ```<jenkins server root>\jobs\<jobname>```, config.xml is directly under it.

   since we don't need to walk through all subfolders, so we use ```next(os.walk(src))[1]``` to get all the direct child of source folder.

```
from shutil import copy2
def extract_config_files(src, target):
    for dir in next(os.walk(src))[1]:
        temp_target = os.path.join(target, dir)
        if not os.path.exists(temp_target):
            os.mkdir(temp_target)
        copy2(os.path.join(src, dir, "config.xml"), temp_target)
```

1. Connect both servers using NET USE command, and call the above extract function. Close the connection in the end

   ```
   old_jenkins_server = "<old server>"
   new_jenkins_server = "<new server>""

   subprocess.call(r"NET USE %s %s /USER:%s\%s" % (r"\\%s" % old_jenkins_server, "<password>", "domainname", "username"))
   subprocess.call(r"NET USE %s %s /USER:%s\%s" % (r"\\%s" % new_jenkins_server, "<password>", "domainname", "username"))

   src = r"\\%s\c$\Jenkins\jobs" % old_jenkins_server
   target= r"\\%s\c$\temp" % new_jenkins_server

   extract_config_files(src, target)

   subprocess.call(r"NET USE \\%s /DELETE" % new_jenkins_server)
   subprocess.call(r"NET USE \\%s /DELETE" % old_jenkins_server)
   ```

   ​

## Go Further

Now to train ourselves with more Jenkins knowledge, we use another solution to solve the same problem. But remember in real case, always use the one that cost less and get things done quickly.

To do this, we use [Jenkins Python api](https://github.com/salimfadhley/jenkinsapi), there are also other similar apis, but this one is more convenient for me.

Planned Steps for migrating Jobs:

- Connect with old server

  ```
  from jenkinsapi.jenkins import Jenkins
  old_server_url = "http://%s:8080/" % old_server_ip
  old_server = Jenkins(old_server_url)
  ```


- Get all the job list from old server

simply call the api's get_jobs function, this will return a dictionary

```
jobs = old_server.get_jobs()
```

- Copy all the job config files from old server to a shared place

```
def backup_files(src, target):
    copy2(src.decode("utf-8"), target.decode("utf-8"))
    
def backup_job_configs(jobs, src, target):
    for job in jobs:
        jobname, job_obj = job
        src = os.path.join(src_root, "jobs", jobname, "config.xml").encode("utf-8")
        target_path = os.path.join(target_root, "jobs", jobname)
        if not os.path.exists(target_path):
            os.mkdir(target_path)
        target = os.path.join(target_path, "config.xml").encode("utf-8")
        backup_files(src, target)
```

- Create new jobs on new server from the config files

```
def create_new_job(server, jobname, xml):
    with open(xml, encoding="utf-8") as file:
        config = file.read()

def create_jobs(server, jobs, xml_path):
    for job in jobs:
        try:
            jobname, _ = job
            if not job_created(server, jobname):
                create_new_job(server, jobname, os.path.join(os.path.join(xml_path, "jobs",), jobname + "_config.xml"))
                if job_created(server, jobname):
                    print(f"{jobname} is created successful")
                else:
                    print(f"{jobname} is NOT created")
        except Exception as e:
            print(jobname.encode("utf-8"), "fail with error: ", e.__str__().encode("utf-8"))
            continue
```

Now new jobs are created on the new Jenkins server.

Same can be applied to Nodes.

- Get all nodes from the old server

```
nodes = old_server.get_nodes()
```

- Create nodes on the new server

  ```
  def create_nodes(server, nodes):
      for key, value in nodes.iteritems():
           if key != "master":
               server.create_node(key , remote_fs='c:\jenkins', num_executors=1)
  ```

- you can also delete nodes if something gets wrong and you want to start over

```
def delete_nodes(new_server):
    nodes = new_server.get_nodes()
    for key, value in nodes.iteritems():
         if key != "master":
             new_server.delete_node(key )
```



To be continued

## To Go Even Further

- Encoding issue

