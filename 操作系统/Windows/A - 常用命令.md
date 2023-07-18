### MD5值

```powershell
 Get-FileHash  -Algorithm  MD5 <filePath>
```

### 开机自启动

```
添加exe文件的快捷方式到下面文件夹:
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
```

### 禁用自动更新

```
注册表路径
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WindowsUpdate\UX\Settings

添加DWORD: FlightSettingsMaxPauseDays
十进制: 3650

设置里修改暂停更新时间
```

![image-20230625161008577](media/image-20230625161008577.png)
