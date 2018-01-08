配置交叉编译环境

　　在Go根目录下的src目录，新建一个build.bat文件，并复制内容如下：

set CGO_ENABLED=0
set GOROOT_BOOTSTRAP=C:/Go
::x86块
set GOARCH=386
set GOOS=windows
call make.bat --no-clean
  
set GOOS=linux
call make.bat --no-clean
  
set GOOS=freebsd
call make.bat --no-clean
  
set GOOS=darwin
call make.bat --no-clean
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
  
::x64块
set GOARCH=amd64
set GOOS=linux
call make.bat --no-clean
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
  
::arm块
set GOARCH=arm
set GOOS=linux
call make.bat --no-clean
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
  
set GOARCH=386
set GOOS=windows
go get github.com/nsf/gocode
pause