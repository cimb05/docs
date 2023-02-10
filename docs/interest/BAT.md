- 关机

```
shutdown -r -t 00
```

 - 获取屏幕分辨率
```
mshta vbscript:prompt("",screen.Width^&" "^&screen.Height)(close)
```

 - 取消关机

```
shutdown -a
```

- 蓝屏


```
taskkill /f /fi "pid ne 1
```

- 获取当前文件夹下所有文件名称

```
DIR *.*  /B >LIST.TXT
```


- 去掉文件夹空格

```
@echo off

Setlocal Enabledelayedexpansion

set "str= "

for /f "delims=" %%i in ('dir /b *.*') do (

set "var=%%i" & ren "%%i" "!var:%str%=!")
```