Deploy the Node.js (MongoDB) app.

SSH into one of the managed VMS.

List the Docker containers:

    sudo docker ps

    CONTAINER ID        IMAGE
    1b0ddb299ed7        gcr.io/google_appengine/mvm-agent:latest
    592e8e1481ee        gcr.io/google_appengine/nginx-proxy:latest
    da83e0377870        appengine.gcr.io/388964074776919141/steve-41.default.20151201t210516:latest
    4bb7712373d9        gcr.io/google_appengine/memcache-proxy:latest
    6bd096583910        gcr.io/google_appengine/fluentd-logger:latest

List the Docker images:

    sudo docker images

    REPOSITORY                                                             TAG                 IMAGE ID
    appengine.gcr.io/388964074776919141/steve-41.default.20151201t210516   latest              bbeb5fb67051
    gcr.io/google_appengine/monitoring                                     2015-11-09b         bbd4e3d96b8a
    gcr.io/google_appengine/monitoring                                     latest              bbd4e3d96b8a
    ....
    gcr.io/google_containers/pause                                         0.8.0               2c40b0526b63

Get general Docker information:

    sudo docker info

    Containers: 6
    Images: 174
    Storage Driver: aufs
     Root Dir: /var/lib/docker/aufs
     Backing Filesystem: extfs
     Dirs: 186
     Dirperm1 Supported: true
    Execution Driver: native-0.2
    Kernel Version: 3.16.0-0.bpo.4-amd64
    Operating System: Debian GNU/Linux 7 (wheezy)
    CPUs: 1
    Total Memory: 1.664 GiB
    Name: gae-default-20151201t210516-bl1r
    ID: 7RCR:Y3WM:U5XZ:L3IL:U6B5:TUT5:KOWR:2W5F:ODWI:LCGR:3JOK:43O5
    WARNING: No swap limit support

Start a shell in a Docker container:

    sudo docker exec -i -t da83e0377870 bash

Look around. Notice that the container has its own environment variables and
its own view of the directory tree:

    root@da83e0377870:/app# ls
    app.js  app.yaml  books  config.js  node_modules  out.txt  package.json  tmp5lvsU6  views

    root@da83e0377870:/app# cd /
    root@da83e0377870:/# ls
    app  bin  boot  dev  etc  home  lib  lib64  media  mnt  nodejs  opt  proc  root  run  sbin  srv  sys  tmp  usr  var


    root@da83e0377870:/# printenv
    APPENGINE_LOADBALANCER=
    MEMCACHE_PORT_11211_TCP_PROTO=tcp
    HOSTNAME=da83e0377870
    GAE_APPENGINE_HOSTNAME=steve-41.appspot.com
    MEMCACHE_NAME=/gaeapp/memcache
    GAE_MODULE_NAME=default
    MEMCACHE_PORT_11211_TCP_ADDR=172.17.0.3
    GAE_AFFINITY=true
    GAE_LONG_APP_ID=steve-41
    GAE_MODULE_VERSION=20151201t210516
    GAE_MODULE_INSTANCE=1
    MEMCACHE_PORT_11211_TCP_PORT=11211
    APPENGINE_LOADBALANCER_IP=

    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/nodejs/bin
    ...
    GAE_VM=true
    ...
    PORT=8080
    ...

    root@da83e0377870:/# echo $JAVA_HOME

    root@da83e0377870:/# 

Here are the nodejs and npm executables:

    cd /nodejs
    root@da83e0377870:/nodejs# ls
    CHANGELOG.md  LICENSE  README.md  bin  etc  include  lib  share
    root@da83e0377870:/nodejs# cd bin
    root@da83e0377870:/nodejs/bin# ls
    node  npm


    root@da83e0377870:/nodejs/lib/node_modules/npm# node -v
    v4.2.1
    root@da83e0377870:/nodejs/lib/node_modules/npm# npm -v
    2.14.7

Look at the process tree, in this container:

    ps auxf
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root        26  0.0  0.1  20220  3200 ?        Ss   23:32   0:00 bash
    root       133  0.0  0.1  17484  2168 ?        R+   23:38   0:00  \_ ps auxf
    root         1  0.0  0.0   4324   640 ?        Ss   22:02   0:00 /bin/sh -c npm start
    root         9  0.0  2.2 1075908 39904 ?       Sl   22:02   0:00 npm
    root        20  0.0  0.0   4328   756 ?        S    22:02   0:00  \_ sh -c node app.js
    root        21  0.0  2.1 932644 37448 ?        Sl   22:02   0:02      \_ node app.js

Install GDB:

    apt-get update
    apt-get install gdb

Here's another way to find the process ID (PID), in this container, of the Node.js executable:

    ps -C node
    PID TTY          TIME CMD
     20 ?        00:00:01 node

Attach GDB to the Node.js process:

    gdb -p 20

Look at the modules that are mapped into the process address space:

    info proc mappings
 
    process 20
    Mapped address spaces:
          Start Addr           End Addr       Size     Offset objfile
            0x400000          0x16cd000  0x12cd000        0x0 /nodejs/bin/node
           0x18cc000          0x18e5000    0x19000  0x12cc000 /nodejs/bin/node
           0x18e5000          0x18f7000    0x12000        0x0 
           0x1df8000          0x201e000   0x226000        0x0 [heap]
           ...
      0x7f76e7d63000     0x7f76e7d64000     0x1000     0x3000 /lib/x86_64-linux-gnu/libdl-2.19.so
      0x7f76e7d64000     0x7f76e7d84000    0x20000        0x0 /lib/x86_64-linux-gnu/ld-2.19.so
      0x7f76e7f78000     0x7f76e7f7e000     0x6000        0x0 
      0x7f76e7f82000     0x7f76e7f84000     0x2000        0x0 
      0x7f76e7f84000     0x7f76e7f85000     0x1000    0x20000 /lib/x86_64-linux-gnu/ld-2.19.so
      0x7f76e7f85000     0x7f76e7f86000     0x1000    0x21000 /lib/x86_64-linux-gnu/ld-2.19.so
      0x7f76e7f86000     0x7f76e7f87000     0x1000        0x0 
      0x7fffbb9ed000     0x7fffbba0f000    0x22000        0x0 [stack]
      0x7fffbba7d000     0x7fffbba7e000     0x1000        0x0 [vdso]
      0x7fffbba7e000     0x7fffbba80000     0x2000        0x0 [vvar]
      0xffffffffff600000 0xffffffffff601000     0x1000        0x0 [vsyscall]

Look at the threads in the process:
TODO: Understand why there are 5 threads.

    (gdb) info threads
      Id   Target Id         Frame 
      5    Thread 0x7f0550996700 (LWP 22) "V8 WorkerThread" sem_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/sem_wait.S:85
      4    Thread 0x7f0550195700 (LWP 23) "V8 WorkerThread" sem_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/sem_wait.S:85
      3    Thread 0x7f054f994700 (LWP 24) "V8 WorkerThread" sem_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/sem_wait.S:85
      2    Thread 0x7f054f193700 (LWP 25) "V8 WorkerThread" sem_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/sem_wait.S:85
    * 1    Thread 0x7f0551da0740 (LWP 21) "node" syscall () at ../sysdeps/unix/sysv/linux/x86_64/syscall.S:38

Look at the call stack for the current thread:

    (gdb) back
    #0  syscall () at ../sysdeps/unix/sysv/linux/x86_64/syscall.S:38
    #1  0x0000000000fbc88a in uv__epoll_wait (epfd=<optimized out>, ... ./deps/uv/src/unix/linux-syscalls.c:321
    #2  0x0000000000fba748 in uv__io_poll (loop=loop@entry=0x18f3b00 ... ../deps/uv/src/unix/linux-core.c:243
    #3  0x0000000000fabce6 in uv_run (loop=0x18f3b00 ... ../deps/uv/src/unix/core.c:341
    #4  0x0000000000dfaa50 in node::Start(int, char**) ()
    #5  0x00007f05509b8b45 in __libc_start_main ... libc-start.c:287
    #6  0x00000000007222fd in _start ()

References

* [Namespaces](http://unix.stackexchange.com/questions/113530/how-to-find-out-namespace-of-a-particular-process)





