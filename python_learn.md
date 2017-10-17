### install easy_install
```
curl https://bootstrap.pypa.io/ez_setup.py -o - | sudo python
sudo easy_install pip
```
there may be a problem
> ImportError: No module named extern
to sole this problem, you can use this method:

```
curl https://pypi.python.org/packages/cf/86/c861f334f2af20a577348c3eb9e05b8d0bbf3a9b7a7126609541becd7033/extern-0.2.1.tar.gz -o extern.tar.gz

tar zxvf extern.tar.gz
sudo python setup.py install
```

#### install pip

```
sudo easy_install pip
```

