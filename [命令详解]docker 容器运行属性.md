常用操作
=========
### 常用
`-d --detach`
后台运行
`--name <name>`
指定容器名字
`-e  --env`
设定容器环境变量
`-e  --env <VAR>:<VALUE>`
将VAR设置为VALUE
`-e  --env <host var>`
将宿主机的var环境变量应用到容器中
`--read-only`
容器只读
`--rm`
退出后自动删除
### volume
**绑定挂载卷：**
`-v --volume <host location>:<container location>:<flag>`
挂载宿主机制定目录/文件到容器指定位置（会抹掉容器原目录/文件）
**Docker管理卷：**
`-v --volume <location>`
在制定位置建立一个管理卷
**共享卷**
`--volumes-from <container ID>`
共享指定容器的卷，卷在容器中的位置相同。
flag:
```
ro		readonly
Z		with selenix (shared among multiple containers.)
z		with selenix (private and unshared.)
```

### 网络
**网络方式**
`--net <method>`
```
--net="none"						无网络
--net="bridge"						桥接（默认）
--net="host"						完全开放
--net="container:<container ID>"	与其他容器公用同一网络
```
**端口暴露**
`-p --publish <container port>`
将容器上的一端口暴露于宿主机随机端口
`-p --publish <host port>:<container port>`
将容器上的一端口暴露于宿主机指定端口
`-p --publish <host ip>::<container port>`
将容器上的一端口暴露于宿主机指定ip随机端口
`-p --publish <host ip>:<host port>:<container port>`
将容器上的一端口暴露于宿主机指定ip指定端口
`-P --publish-all`
将容器上开放的所有端口暴露于宿主机随机端口
**容器间通信**
`--link <container IP>:<host>`
建立到指定容器的一条链接，在本容器内，host将解析到目标容器
**其他**
`--hostname <hostname>`
指定容器主机名
`--dns=[<dns1>,<dns2>]`
`--dns <dns>`
指定dns服务器
### 交互
`-i  --interactive`
打开容器STDIN
`-t --tty`
分配一个终端
`docker exec -it <container ID> bash`
进入一个容器
### 重启选项
`--restart=<method>`
```
--restart=no					不重启（默认）
--restart=on-failure[:retry]	遇到错误时重启（最多几次）
--restart=always				始终重启
--restart=unless-stopped		除非docker启动时容器已经处于退出状态，否则一直重启
```
太长不看版
=========
```
      --add-host list                  Add a custom host-to-IP mapping (host:ip)
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight), between 10 and 1000, or 0 to
                                       disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight) (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container
      --cidfile string                 Write the container ID to the file
      --cpu-count int                  CPU count (Windows only)
      --cpu-percent int                CPU percent (Windows only)
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
  -c, --cpu-shares int                 CPU shares (relative weight)
      --cpus decimal                   Number of CPUs
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
  -d, --detach                         Run container in background and print container ID
      --detach-keys string             Override the key sequence for detaching a container
      --device list                    Add a host device to the container
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
      --device-read-bps list           Limit read rate (bytes per second) from a device (default [])
      --device-read-iops list          Limit read rate (IO per second) from a device (default [])
      --device-write-bps list          Limit write rate (bytes per second) to a device (default [])
      --device-write-iops list         Limit write rate (IO per second) to a device (default [])
      --disable-content-trust          Skip image verification (default true)
      --dns list                       Set custom DNS servers
      --dns-option list                Set DNS options
      --dns-search list                Set custom DNS search domains
      --entrypoint string              Overwrite the default ENTRYPOINT of the image
  -e, --env list                       Set environment variables
      --env-file list                  Read in a file of environment variables
      --expose list                    Expose a port or a range of ports
      --group-add list                 Add additional groups to join
      --health-cmd string              Command to run to check health
      --health-interval duration       Time between running the check (ms|s|m|h) (default 0s)
      --health-retries int             Consecutive failures needed to report unhealthy
      --health-start-period duration   Start period for the container to initialize before starting
                                       health-retries countdown (ms|s|m|h) (default 0s)
      --health-timeout duration        Maximum time to allow one check to run (ms|s|m|h) (default 0s)
      --help                           Print usage
  -h, --hostname string                Container host name
      --init                           Run an init inside the container that forwards signals and
                                       reaps processes
  -i, --interactive                    Keep STDIN open even if not attached
      --io-maxbandwidth bytes          Maximum IO bandwidth limit for the system drive (Windows only)
      --io-maxiops uint                Maximum IOps limit for the system drive (Windows only)
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
  -l, --label list                     Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g., 92:d0:c6:0a:29:33)
  -m, --memory bytes                   Memory limit
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --name string                    Assign a name to the container
      --network string                 Connect a container to a network (default "default")
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --platform string                Set platform if server is multi-platform capable
      --privileged                     Give extended privileges to this container
  -p, --publish list                   Publish a container's port(s) to the host
  -P, --publish-all                    Publish all exposed ports to random ports
      --read-only                      Mount the container's root filesystem as read only
      --restart string                 Restart policy to apply when a container exits (default "no")
      --rm                             Automatically remove the container when it exits
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --sig-proxy                      Proxy received signals to the process (default true)
      --stop-signal string             Signal to stop a container (default "SIGTERM")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
  -t, --tty                            Allocate a pseudo-TTY
      --ulimit ulimit                  Ulimit options (default [])
  -u, --user string                    Username or UID (format: <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
  -v, --volume list                    Bind mount a volume
      --volume-driver string           Optional volume driver for the container
      --volumes-from list              Mount volumes from the specified container(s)
  -w, --workdir string                 Working directory inside the container
```
