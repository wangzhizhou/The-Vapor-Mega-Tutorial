# 部署

```bash
$ vapor cloud deploy 
app: TILApp
git: https://github.com/wangzhizhou/TILApp-BackEnd.git
env: production
db: yes
replicas: 1
replica size: free
branch: master
build: incremental
Creating deployment [Done]
Connecting to build logs ...
Waiting in Queue [Done]
Checkout branch 'master' [Done]
Verifying base folder [Done]
Selected swift version: 4.1.0 [Done]
Building vapor (release) [Done]
Trying to find executable [Done]
Found executable: Run [Done]
Creating container registry [Done]
Building container [Done]
Pushing container to registry [Done]
Updating replicas [Done]
Deployment succeeded: https://joker.vapor.cloud [Done]
Successfully deployed.
```

# 取消部署并删除

首先需要把部署的机器数变为`0`

```
$  vapor cloud deploy --replicas=0
```

然后在Vapor云上自己的帐号中，从内到外一级一级删除。