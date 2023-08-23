---
layout:     post
title:      "Studio 3T 重置试用，实现无限期免费试用"
subtitle:   " \"Studio3T for Mac\""
date:       2023-08-23 11:04:13 +08:00
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - Mac
    - MongoDB
    - Studio 3T
    - Windows
---

# Studio 3T for Mac 重置试用，实现无限期免费试用

> 运行脚本前请先退出软件。

## Studio 3T for Windows 重置

新建一个 `Reset3t.bat` 文件，文件内添加一下脚本。

```
@echo off
ECHO 重置Studio 3T的使用日期......
REG DELETE "HKEY_CURRENT_USER\Software\JavaSoft\Prefs\3t\mongochef\enterprise" /f
RMDIR /s /q %USERPROFILE%\.3T\studio-3t\soduz3vqhnnja46uvu3szq--
RMDIR /s /q %USERPROFILE%\.3T\studio-3t\Lwm3TdTxgYJkXBgVk4s3
RMDIR /s /q %USERPROFILE%\AppData\Local\t3\dataman\mongodb\app\AppRunner
RMDIR /s /q C:\Users\Public\t3\dataman\mongodb\app\AppRunner
RMDIR /s /q %USERPROFILE%\AppData\Local\Temp\t3\dataman\mongodb\app\AppRunner
RMDIR /s /q %USERPROFILE%\AppData\Local\ftuwWNWoJl-STeZhVGHKkQ--
RMDIR /s /q %USERPROFILE%\AppData\Local\Temp\ftuwWNWoJl-STeZhVGHKkQ--
RMDIR /s /q %USERPROFILE%\.cache\ftuwWNWoJl-STeZhVGHKkQ--
ECHO 重置完成, 按任意键退出......
pause>nul
EXIT
```

## Studio 3T for Mac 重置

### Shell脚本版

```shell
#!/bin/sh
#删除根目录下的缓存文件
rm -f ~/Library/Preferences/3t.*
rm -rf ~/.3T
rm -rf ~/.cache/ftuwWNWoJl-STeZhVGHKkQ--
 
#找到无权限和无法操作之外的文件
ftPath=`find /var/folders -name "ftuwWNWoJl-STeZhVGHKkQ--" -print 2>&1 | fgrep -v "Permission denied" | fgrep -v "Operation not permitted"`
t3Path=`dirname ${ftPath}`/t3
 
if [ -e ${ftPath} ];then
    rm -rf ${ftPath}
fi
 
if [ -e ${t3Path} ];then
    rm -rf ${t3Path}
fi
 
echo "删除成功"
```

执行完脚本，重新打开`Studio 3T`即可。

### AppleScript 脚本版

新建一个`Reset3t.applescript` 文件，把以下内容添加到文件中。

```applescript
-- 重置Studio 3T
display dialog "重置Studio 3T，请先导出链接信息，重置后链接信息会丢失。已导出请选择Next。" with title "Reset 3T" buttons { "Cancel" , "Next" } default button 2 with icon 1 giving up after 10
if button returned of result is "Next" then
    my reset3T()
end if
-- 重置后需重启系统生效
on reset3T()
    set folderPaths to {"~/Library/Preferences/3t.*", "~/.3T", "~/.cache/ftuwWNWoJl-STeZhVGHKkQ--"}
    set separator to linefeed -- 换行符

    set concatenatedString to "请先关闭Studio 3T" & separator
    set concatenatedString to "将删除以下文件夹：" & separator
    repeat with aPath in folderPaths
        set concatenatedString to concatenatedString & aPath & separator
    end repeat

    -- 清除最后一个换行符
    if length of concatenatedString > length of separator then
        set concatenatedString to text 1 thru -((length of separator) + 1) of concatenatedString
    end if

    display dialog concatenatedString with title "Reset 3T" buttons { "Cancel" , "Reset" } default button 2 with icon 1 giving up after 10
    if button returned of result is "Reset" then
        repeat with aPath in folderPaths
            try
                do shell script "rm -rf " & aPath with administrator privileges
            end try
        end repeat
        display dialog "重置完成，需重启系统生效，重启之前不要打开Studio3T，是否重启系统？" with title "Reset 3T" buttons { "Restart later", "OK" } default button 1 with icon 1 giving up after 10
        if button returned of result is "OK" then
            do shell script "sudo shutdown -r now" with administrator privileges
        end if
    end if
end reset3T
```

可以使用脚本编辑器打开执行，也可以使用`osascript /path/to/Reset3t.applescript`命令执行.

> 注意： Studio 3T for Mac 重置后，需要重启电脑。重启之前，不要重新打开`Studio 3T for Mac`，否则重置会失败。