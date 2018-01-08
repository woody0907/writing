### oracle
setting.py

    DATABASES = {
    'default': {
        #'ENGINE': 'django.db.backends.', # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
        'ENGINE': 'django.db.backends.sqlite3', #添加数据库引擎；选项['postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'].
        'NAME': 'F:/TestPython/blog/blog/db/data.db', # 数据库文件的路径.
        # The following settings are not used with sqlite3:
        # 下面的配置不适用于sqlite3:
        'USER': '',    # 数据库登陆用户名
        'PASSWORD': '', # 数据库登陆密码
        'HOST': '',                      # Empty for localhost through domain sockets or '127.0.0.1' for localhost through TCP. # 主机名
        'PORT': '',                      # Set to empty string for default. # 端口号
    }
}

    from django.db import connection

    data = []
    cursor = connection.cursor()
    cursor.execute('select * from customer')
    for row in cursor.fetchall():
        data.append(row[0])

        from django.db import connection,transaction
        cursor = connection.cursor()
        cursor.execute("update customer set customername = '%s',tel = '%s' where Id = 1"%(un,tel))
        transaction.commit_unless_managed()  #记得提交事务
