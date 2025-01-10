```
      #initContainers:
      #  - name: init-0
      #    image: postgres:16.3
      #    imagePullPolicy: IfNotPresent
      #    terminationMessagePath: /dev/termination-log
      #    terminationMessagePolicy: File
           env:
             - name: POSTGRES_USER
               value: postgres
             - name: POSTGRES_PWD
               value: postgres
             - name: POSTGRES_DB
               value: drone
           # DROP DATABASE IF EXISTS dbname;CREATE DATABASE dbname;
           # CREATE USER 'drone' WITH PASSWORD ""; CREATE DATABASE dbname WITH OWNER drone;
      #    command: [ 'sh', '-c', 'psql -U $POSTGRES_USER -tc "SELECT 1 FROM pg_database WHERE datname = \'$POSTGRES_DB\'" | grep -q 1 || psql -U $POSTGRES_USER -c "CREATE DATABASE $POSTGRES_DB"' ]
      #    securityContext:
      #    privileged: true
      
              - name: init-0
          image: busybox
          imagePullPolicy: IfNotPresent
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          command: [ "sysctl", "-w", "net.core.somaxconn=511" ]
          securityContext:
            privileged: true
#        - name: init-1
#          image: busybox
#          imagePullPolicy: IfNotPresent
#          terminationMessagePath: /dev/termination-log
#          terminationMessagePolicy: File
#          command: [ "sh", "-c", "echo never > /sys/kernel/mm/transparent_hugepage/enabled" ]
#          securityContext:
#            privileged: true


pg_isready --host=<数据库主机地址> --port=<数据库端口> --username=<数据库用户名>
pg_isready 是 PostgreSQL 自带的一个用于检查数据库可访问性的工具。你可以使用以下命令来测试数据库是否可访问：
如果数据库可访问，命令将会返回 ok，否则将返回相关错误信息。
```