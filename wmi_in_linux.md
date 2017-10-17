### download wmi_client autoconf wmi
link:http://pan.baidu.com/s/1qYLWz5U  pass:w1g8
#### details
> when install wmi on mac there are someting need to be care:

```
cd wmi-1.3.14
vi GNUmakefile
### add ZENHOME=.
### then you nead intall autoconf,and then
sudo make "CPP=gcc -E -ffreestanding"
sudo cp wmic /usr/local/bin
```

This is a python example to test the wmi:

```python
import wmi_client_wrapper as wmi

wmic = wmi.WmiClientWrapper(
    username="administrator",
    password="szsyl123",
    host="191.168.3.158"
)

output = wmic.query("SELECT * FROM Win32_Processor")
print(output)
```
