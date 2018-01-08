# oracle
数据库登录10.142.6.174 
1. 安装参数
    vi /etc/grub.conf.bak
    vi /etc/security/limits.conf.bak
    vi /etc/sysctl.conf.bak
2. 安全检查

     (1)登录用户：monitoruser/jf_wljk_3012
     (2)用户权限：
     su oracle
     export ORACLE_SID=zhwgdb1

     select * from dba_role_privs where grantee='MONITORUSER'
     (3)端口
     vi $ORACLE_HOME/network/admin/tnsnames.ora
     (4)rac切换，参看上述配置
     (5)10.142.1.113备份脚本
     
3. 报告导出
    su oracle
    export ORACLE_SID=zhwgdb1
    sqlplus / as sysdba

    @$ORACLE_HOME/rdbms/admin/awrrpt
    @$ORACLE_HOME/rdbms/admin/addmrpt
    @$ORACLE_HOME/rdbms/admin/ashrpt


4. session

    （1）ora事件分析...
     (2)最大并发数
     select * from v$license


jvm
jvisualvm


资源管理--资源推送