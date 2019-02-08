---
title: Batch Setup Java Web Start for Jenkins Slaves
date: 2018-08-10 21:05:13 +0800
layout: post
categories:
- Python
tags:
- Python
---

# Background

We have migrated our Jenkins server to a new machine. This caused all our existing automation slave machines offline, an update on the java web start script is needed. Windows Platform.

The problem is, there are quite many slave machines impacted, it's tedious to update them manually.

Need a way to update them in batch

## Analysis

For each slave machine, following tasks are needed:

- Create a new batch file, which contains the java web start command

- Download the agent.jar from jenkins node page

  - since the agent.jar file is same for all nodes, this can be simplified to download manually on on machine, and then dispatch to all slaves

- Setup to run batch file automatically when slave machine is started.

  - This can be solved by putting the batch file under windows startup menu

    ```
    %appdata%\Microsoft\Windows\Start Menu\Programs\Startup
    ```

This turns out to be a task on Windows:

```
Copy files to remote network machines
```

In python, there is a useful module **shutil** which can help copy files.

To access network machines, windows has a native command "**NET USE**".

With help of above info, we can now start.

## Solution

Steps as below. 

1. Prepare data

   Data includes:

   - Jenkins server address
   - Jenkins node
   - remote slaves (mapped to Jenkins node)

   So use dictionary to setup the mapping between Jenkins node and remote slaves

   ```
   remote_slaves = {
   	"<Jenkins Node 1>":"<remote_slave1>",
   	...
   }
   ```

   Then for each remote slave, do following steps.

2. Access remote machine via NET USE

   The NET USE command format is as below:

   ```
   net use <remote machine> <password> /USER:<domain>/<username>
   ```

   To execute the "**NET USE**" command in python, we need a module "**subprocess**"

```
command = r"NET USE %s %s /USER:%s\%s" % (r"\\%s" % remote_machine, "password", "domain_name", "username")
subprocess.call(command)
```

3. Copy "agent.jar" file to remote machine using shutil

```
import shutil
src = "path to agent.jar"
target = "path to location on remote machine"
shutil.copy2(src, target)
```

shutil has other copy functions as well, detailed difference please refer to []()

here the target variable, I set it using a not-good-enough format. I was planning to use %%appdata% to shorten it but no luck.

```
loc = os.path.join(f"\\\\{remote_machine}\c$", r"Users\	<username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup")
copy2("agent.jar", os.path.join(loc,"agent.jar"))
```

4. Create new batch file on remote machine

```
bat_text = f"""java -jar agent.jar -jnlpUrl http://{jenkins_server_ip}:8080/computer/{nodename}/slave-agent.jnlp"""
target_file = os.path.join(loc, "Java_Web_Start_Bat_File.bat")
if not os.path.exists(target_file):
    with open(target_file, "w") as f:
    f.write(bat_text)
assert os.path.exists(target_file) == True
```

​

5. Close NET USE connection

```
subprocess.call(r"NET USE \\%s /DELETE" % remote_machine)
```

​

## To Go Further

- Ideally it would be great to run the batch file immediately, without restarting the remote machine.

  But I haven't found a good way to achieve it on Windows yet.

  There seems some ways like SSH, third-party tools. but SSH failed to connect my remote machines (might need some work on the machines).

  Third-party tools include one which I ever saw in DAMA's st service, called Fabric, might investigate some time later.

  Welcome anyone who have suggestions on this, appreciated.

  ​

- The current code is straight-forward, missing reusability, and need further modification

