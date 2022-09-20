---
layout: custom
title: Airflow tutorial
date: 2022-09-13 14:27:00 +0900
last_modified_at: 2022-09-13 14:27:00 +0900
category: mlops
tags: ["airflow"]
published: false

---
> airflow tutorial

## 1. Airflow 설치

- 참고: [https://blog.si-analytics.ai/59](https://blog.si-analytics.ai/59)

- mysql, redis 설치 (Docker)
    ```bash
    $ docker pull mysql:8.0
    $ docker pull redis:5.0
    $ docker run -d --name mysql -p 3306:3306 -v /data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD={PASSWD} mysql:8.0
    $ docker run -d --name redis -p 6379:6379 -v /data/redis:/data -e REDIS_PASSWORD={PASSWD} redis:5.0
    ```

- Airflow 설치
    - Docker 컨테이너 실행 & venv 생성
    ```bash
    $ docker exec -it {container} /bin/bash
    # su {user}
    $ cd {project dir}
    $ python -m venv .venv
    ```

    - venv 관련 에러 발생
    ```
    The virtual environment was not created successfully because ensurepip is not
    available.  On Debian/Ubuntu systems, you need to install the python3-venv
    package using the following command.

        apt-get install python3-venv

    You may need to use sudo with that command.  After installing the python3-venv
    package, recreate your virtual environment.

    Failing command: ['/home/airflow/.venv/bin/python3', '-Im', 'ensurepip', '--upgrade', '--default-pip']
    ```

    - 해결방법
    ```bash
    $ sudo apt-get install python3-venv
    ```

    - pip upgrade (optional, venv 생성 후 pip upgrade)
    ```bash
    $ pip install --upgrade pip
    ```

    - airflow 설치
    ```bash
    $ pip install 'apache-airflow[mysql,redis]==1.10.5'
    ```

    - mysqlclient 설치 관련 에러 발생
    ```
    WARNING: Discarding https://files.pythonhosted.org/packages/6b/ba/4729d99e85a0a35bb46d55500570de05b4af10431cef174b6da9f58a0e50/mysqlclient-1.3.1.tar.gz#sha256=3549e8a61f10c8cd8eac6581d3f44d0594f535fb7b29e6090db3a0bc547b25ad (from https://pypi.org/simple/mysqlclient/). Command errored out with exit status 1: python setup.py egg_info Check the logs for full command output.
    Downloading mysqlclient-1.3.0.tar.gz (76 kB)
        |████████████████████████████████| 76 kB 791 kB/s
    Preparing metadata (setup.py) ... error
    ERROR: Command errored out with exit status 1:
    command: /home/airflow/.venv/bin/python3 -c 'import io, os, sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-nbvzzgek/mysqlclient_417126ea37014ef6b1d4cc8f1f717854/setup.py'"'"'; __file__='"'"'/tmp/pip-install-nbvzzgek/mysqlclient_417126ea37014ef6b1d4cc8f1f717854/setup.py'"'"';f = getattr(tokenize, '"'"'open'"'"', open)(__file__) if os.path.exists(__file__) else io.StringIO('"'"'from setuptools import setup; setup()'"'"');code = f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' egg_info --egg-base /tmp/pip-pip-egg-info-2nfx9z9s
        cwd: /tmp/pip-install-nbvzzgek/mysqlclient_417126ea37014ef6b1d4cc8f1f717854/
    Complete output (10 lines):
    /bin/sh: 1: mysql_config: not found
    Traceback (most recent call last):
        File "<string>", line 1, in <module>
        File "/tmp/pip-install-nbvzzgek/mysqlclient_417126ea37014ef6b1d4cc8f1f717854/setup.py", line 17, in <module>
        metadata, options = get_config()
        File "/tmp/pip-install-nbvzzgek/mysqlclient_417126ea37014ef6b1d4cc8f1f717854/setup_posix.py", line 47, in get_config
        libs = mysql_config("libs_r")
        File "/tmp/pip-install-nbvzzgek/mysqlclient_417126ea37014ef6b1d4cc8f1f717854/setup_posix.py", line 29, in mysql_config
        raise EnvironmentError("%s not found" % (mysql_config.path,))
    OSError: mysql_config not found
    ----------------------------------------
    WARNING: Discarding https://files.pythonhosted.org/packages/6a/91/bdfe808fb5dc99a5f65833b370818161b77ef6d1e19b488e4c146ab615aa/mysqlclient-1.3.0.tar.gz#sha256=06eb5664e3738b283ea2262ee60ed83192e898f019cc7ff251f4d05a564ab3b7 (from https://pypi.org/simple/mysqlclient/). Command errored out with exit status 1: python setup.py egg_info Check the logs for full command output.
    ERROR: Could not find a version that satisfies the requirement mysqlclient (from versions: 1.3.0, 1.3.1, 1.3.2, 1.3.3, 1.3.4, 1.3.5, 1.3.6, 1.3.7, 1.3.8, 1.3.9, 1.3.10, 1.3.11rc1, 1.3.11, 1.3.12, 1.3.13, 1.3.14, 1.4.0rc1, 1.4.0rc2, 1.4.0rc3, 1.4.0, 1.4.1, 1.4.2, 1.4.2.post1, 1.4.3, 1.4.4, 1.4.5, 1.4.6, 2.0.0, 2.0.1, 2.0.2, 2.0.3, 2.1.0rc1, 2.1.0, 2.1.1)
    ERROR: No matching distribution found for mysqlclient
    ```

    - 해결방법
    - 참고: [https://iamiet.tistory.com/54](https://iamiet.tistory.com/54)
    ```bash
    $ sudo apt-get install python3-dev default-libmysqlclient-dev build-essential
    $ pip install 'apache-airflow[mysql,redis]==1.10.5'
    ```

    - 설치확인
    ```bash
    $ airflow version
    ```

    - wtforms 관련 에러 발생
    ```
    (.venv) kyg@2f446165ad26:~$ airflow version
    [2022-09-13 06:48:10,196] {__init__.py:51} INFO - Using executor SequentialExecutor
    Traceback (most recent call last):
        File "/home/mounted/airflow_test/.venv/bin/airflow", line 22, in <module>
            from airflow.bin.cli import CLIFactory
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/airflow/bin/cli.py", line 67, in <module>
            from airflow.www.app import (cached_app, create_app)
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/airflow/www/app.py", line 37, in <module>
            from airflow.www.blueprints import routes
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/airflow/www/blueprints.py", line 25, in <module>
            from airflow.www import utils as wwwutils
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/airflow/www/utils.py", line 35, in <module>
            from wtforms.compat import text_type
    ModuleNotFoundError: No module named 'wtforms.compat'
    ```
    - 해결방법
    ```bash
    pip install wtforms==2.2
    ```

    - werkzeug 관련 에러 발생
    ```
    [2022-09-13 07:12:18,959] {__init__.py:51} INFO - Using executor SequentialExecutor
    Traceback (most recent call last):
        File "/home/mounted/airflow_test/.venv/bin/airflow", line 22, in <module>
            from airflow.bin.cli import CLIFactory
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/airflow/bin/cli.py", line 67, in <module>
            from airflow.www.app import (cached_app, create_app)
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/airflow/www/app.py", line 37, in <module>
            from airflow.www.blueprints import routes
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/airflow/www/blueprints.py", line 25, in <m                                                                                                     odule>
            from airflow.www import utils as wwwutils
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/airflow/www/utils.py", line 39, in <module                                                                                                     >
            from flask_admin.model import filters
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/flask_admin/model/__init__.py", line 2, in                                                                                                      <module>
            from .base import BaseModelView
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/flask_admin/model/base.py", line 8, in <mo                                                                                                     dule>
            from werkzeug import secure_filename
    ImportError: cannot import name 'secure_filename'
    ```

    - 해결방법
    ```bash
    pip install werkzeug==0.16.0
    ```

    - werkzeug 관련 에러 발생
    ```
    Traceback (most recent call last):
        File "/home/mounted/airflow_test/.venv/bin/airflow", line 22, in <module>
            from airflow.bin.cli import CLIFactory
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/airflow/bin/cli.py", line 67, in <module>
            from airflow.www.app import (cached_app, create_app)
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/airflow/www/app.py", line 37, in <module>
            from airflow.www.blueprints import routes
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/airflow/www/blueprints.py", line 25, in <m                                                                                                     odule>
            from airflow.www import utils as wwwutils
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/airflow/www/utils.py", line 40, in <module                                                                                                     >
            import flask_admin.contrib.sqla.filters as sqlafilters
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/flask_admin/contrib/sqla/__init__.py", lin                                                                                                     e 2, in <module>
            from .view import ModelView
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/flask_admin/contrib/sqla/view.py", line 17                                                                                                     , in <module>
            from flask_admin.contrib.sqla.tools import is_relationship
        File "/home/mounted/airflow_test/.venv/lib/python3.6/site-packages/flask_admin/contrib/sqla/tools.py", line 4                                                                                                     , in <module>
            from sqlalchemy.ext.declarative.clsregistry import _class_resolver
    ModuleNotFoundError: No module named 'sqlalchemy.ext.declarative.clsregistry'

    ```
    
    - 해결방법
    ```bash
    pip install "SQLAlchemy==1.3.15"
    ```

    
  




- Airflow tutorial
    - Airflow 관련 DB 생성
    ```bash
    mysql -h {host} -u root -p {passwd} -P {PORT}
    mysql > create user 'airflow'@'%' identified by 'airflow';
    mysql > grant all privileges on *.* to 'airflow'@'%';
    mysql > flush privileges;
    mysql > create database airflow;
    ```