## 在`kubernetes` 集群上安装`sentry` 服务

`sentry`这个镜像比较坑，不能一次性安装完成。

**第一步**：环境变量的配置，根据你的实际情况进行填写，比如这里的`postgresql`数据库可以安装到同一个`POD`下面，我这里因为之前就安装过，所以就直接使用了，注意环境变量中的`postgresql`数据库的用户名需要使用`postgres`，`sentry`要求使用超级管理员权限，然后我是手动到`postgresql`中手动新建了一个数据库：`sentry`，然后把权限赋给`postgres`：(进入psql)
```shell
CREATE DATABASE sentry OWNER postgres;
GRANT ALL PRIVILEGES ON DATABASE sentry to postgres;
```

**第二步**：先执行`deployment0.yaml`这个文件，里面的执行的命令是：`sentry upgrade`，用于同步数据库结构到`postgresql`中，执行完成后，最好进入容器终端再执行下面的命令：
```shell
sentry django migrate
```
用于确认同步结构。

**第三步**：我们可以在`sentry`数据库中查询`sentry_organization`表，看其中是否有数据，虽然[官方说明](https://github.com/getsentry/sentry/issues/3002)执行了上面的**upgrade**操作会初始化一些基本数据，但是我这边测试发现该表中没有数据，没有数据的结果会导致后面用户报错：**IndexError: list index out of range**，添加一条数据：
```shell
INSERT INTO sentry_organization(name, status, date_added, slug, flags, default_role) VALUES('yidianzhishi', 0, '2017-05-09 02:30:40.719879+00', 'ydzs', 1, 'member');
```

**第四步**：上面的数据库操作执行完成了，现在回到上面的容器中去，新建用户：
```shell
sentry createuser
```

然后根据提示输入即可。

**第五步**：删除上面的`deployment0.yaml`，添加`deployment.yaml`以及`svc.yaml`
```shell
kubectl delete -f delpoyment0.yaml
kubectl create -f deployment.yaml
kubectl create -f svc.yaml
```

上面的`deployment.yaml`中运行了3个容器，一个是**WEB**服务，一个是`Celery Worker`，另外一个是定时任务。

至此，`sentry`在`kubernetes`上就部署完成了。

![sentry](https://blog.sentry.io/img/post-images/sentry-v8/stream.png)

