## 基准测试
### 1. mysqlslap 
- 常用参数说明
    - `--auto-generate-sql` 由系统自动生成sql脚本进行测试
    - `--auto-generate-sql-add-autoincrement` 在生成的表中增加自增ID （Innodb里需要用）
    - `--auto-generate-sql-load-type` 指定测试中使用的查询类型（例如读、写...）
    - `--auto-generate-sql-write-number` 指定初始化数据时生成的数据量
    - `--concurrency` 指并发线程的数量
    - `--engine` 指定测试表的存储引擎，可以用逗号分割多个存储引擎（）
    - `--no-drop` 指定不清理测试数据； 不能和上面的多个存储引擎一起使用，因为上面的多个引擎的测试，是测试完一个就把数据删除了；
    - `--iterations` 指定测试运行的次数; 同样的，因为每次都生成新的数据，所以使用这个数据的时候，就不可以使用上面这个数据；
    - `--number-of-queries` 指定每一个线程执行的查询数量；
    - `--debug-info` 指定输出额外的内存及CPU统计信息
    - `--number-int-cols` 指定测试表中包含的INT类型列的数量
    - `--number-char-cols` 指定测试表中包含的varchar类型列的数量
    - `--create-schema` 指定了用于执行测试的数据库名字
    - `--query` 用于指定自定义的sql脚本
    - `--only-print` 并不会运行测试脚本，而是把生成的脚本打印出来

### 2. sysbench
可以对io, cpu, 内存等等进行测试.. 很全面且灵活，比较常用，适合对服务器进行测试。
1. 常用参数
    - `--test` 用于指定所要执行的测试类型，支持以下参数
        - `fileio` 文件系统IO性能测试
        - `cpu` CPU性能测试
        - `memory` 内存性能测试
        - `Oltp` 测试要指定具体的lua脚本
        Lua脚本位于 `sysbench-0.5/sysbench/tests/db`
    - `--mysql-db` 用于指定执行基准测试的数据库名
    - `--mysql-table-engine` 用于指定所使用的存储引擎
    - `--oltp-tables-count` 执行测试的表的数量
    - `--oltp-table-size` 指定每个表中的数据行数
    - `--num-threads` 指定测试的并发线程数量
    - `--max-time` 指定最大的测试时间
    - `--report-interval` 指定间隔多久输出依次统计信息
    - `--mysql-user` 指定执行测试的MYSQL用户
    - `--mysql-password` 指定执行测试的MYSQL用户的密码
    - `prepare` 用于准备测试数据 
    - `run` 用于实际进行测试
    - `cleanup` 用于清理测试数据

2. 测试举例:
   - 基础组件的测试:
     - 测试cpu性能: 计算小于10000的素数:  <br>``sysbench --test=cpu --cpu-max-prime=10000 run``
     - 测试io性能:
        - 随机读写: `sysbench --test=fileio --num-threads=8 --init-rng=on --file-total-size=1G --file-test-mode=rndrw  run `
        - 生成一个G的文件：`sysbench --test=fileio --file-total-size=1G prepare`
    - 数据库性能测试:
        1. 新建一个数据库 `create database imooc;`
        2. 设置权限 `grant all privileges on *.* to sbest@'localhost' identified by '123456';`
        3. 准备测试数据(表和表中的数据)   
            - 插入10个表，每个表10000行记录，使用innodb引擎，使用的脚本是`sysbench/tests/db/`下面的脚本
            - `sysbench --test=./oltp.lua --mysql-table-engine=innodb --oltp-table-size=10000 --mysql-db=imooc --mysql-user=sbest --mysql-password=123456 --oltp-tables-count=10 prepare`
        4. 保存一下当前的系统状态(运行前的系统状态), 运行脚本
        5. 开始测试 `sysbench --test=./oltp.lua --mysql-table-engine=innodb --oltp-table-size=10000 --mysql-db=imooc --mysql-user=sbest --mysql-password=123456 --oltp-tables-count=10 run` <br>最后一个改成了`run`
        6. 数据结果分析.