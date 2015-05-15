###指代容器的三种方式
* 短UUID
* 长UUID
* 容器名

###附着到一个正在运行的容器
```language
docker attach [container short uuid|name|long uuid]
```
>对于ubuntu这样的容器有时需要用到shell来操作里面的东西，docker重新启动容器时没有`-i -t`标记，默认不会运行ubuntu的bash，通过`attach`就会重新出来shell

###docker logs
* docker logs [container name]...获取docker容器的日志（静态的，只读取某个时刻的日志）
* docker logs -f ...监控docker日志（动态输出，实时监控日志）
* docker logs --tail 10...介个shell的tail命令，读取最后10行
* docker logs --tail 0 -f...跟踪最新的日志

```language
docker top [container name]
```
查看容器中的进程名

###在Docker内部运行进程
* 后台进程..没有交互需求
* 交互进程..有交互需求（shell等）

```language
docker exec [-i | -t] container name
```
** 这里的容器必须是运行的，不然这条命令会报错**

###容器自动重启
