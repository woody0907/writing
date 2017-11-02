###create a python wmi images
* make a Dockerfile
* vi Dockerfile
```
   FROM daocloud.io/python:2.7    #this speed is faster than [FROM python:2.7]
```
* build the image 
```
docker build woody/pythonwmi .
```

* creat and start a new container

```
docker run -dit --name=pythonwmi woody/pythonwmi
```

* copy files to the new container
```
docker cp ~\develop\python_libs pythonwmi:\python_libs

```

* install wmi_client_wrapper

```
docker exec pythonwmi docker exec -i pytonwmi python /python_libs/wmi-client-wrapper-0.0.12/setup.py install
```

* install wmic

```
docker exec -it pythonwmi /bin/bash
rm /python_libs/wmi_1.3.14
tar -jxvf wmi-1.3.14.tar.bz2
vi GNUmakefile
### add ZENHOME=.
### then you nead intall autoconf,and then
sudo make "CPP=gcc -E -ffreestanding"
sudo cp wmic /usr/local/bin```

* test pythonwmic
```
docker exec pytonwmi python lwmi.py 192.168.168.114 administrator xxxxxx;
```

* export the container
```
docker export pythonwmi|gzip > pythonwmi.gz
```

* import the container
```
cat pythonwmi.gz | docker import - pythonwmi
```











