### This Page is about 'How you can send keystokes via Powershell' 

#### Works in Current version of windows 

<h4>Story Time</h4>

Sending keys though Powershell . is that amazing . Yes we can do this and make victim users Burnt out or to execute some kind of payloads.

### THIS DOESN'T NEED ADMIN PRIVILAGE :)
So Lets Goo...


#### Sending Keystokes

We can send key stokes using System.windows.Forms (assembly) . Its an type of API System that is used in Windows Machines . Its Preinstalled in all windows versions . <br>
<br>For More Info Visit <a href="https://learn.microsoft.com/en-us/dotnet/api/system.windows.forms?view=windowsdesktop-7.0" > Document </a>

```
Add-Type -AssemblyName System.Windows.Forms
```

<h4> Before Performing Key Stokes Just import the above command . you have to import it . Its Preinstalled ! </h4>
Thats it Gud to Go .

# Some Basic Keystokes Examples 

```
[System.Windows.Forms.SendKeys]::SendWait('hello')
```
This will type `hello` via sending keys.

## How to use it in proper way ( IDEA )

You can use this method in several ways . Example

<br><br>
#### Perform an Keystoke that opens a browser with specfic link and Enters Full screen !
<br>


For Edge Browsers :

```
$edge = New-Object -ComObject 'Microsoft.Edge.Application';$edge.Visible = $true;$ie.Navigate("https://smukx.github.io/hacked"); while ($ie.Busy) { Start-Sleep -Milliseconds 100 };$ie.FullScreen = $true;[System.Windows.Forms.SendKeys]::SendWait("{F11}")

2nd Option 

$ie = New-Object -ComObject InternetExplorer.Application; $ie.Visible = $true;$ie.Navigate("https://smukx.github.io/hacked"); while ($ie.Busy) { Start-Sleep -Milliseconds 100 };$ie.FullScreen = $true;[System.Windows.Forms.SendKeys]::SendWait("{F11}")

```
<br>

For Chrome Browsers :

```
$chrome = New-Object -ComObject 'Chrome.Application';$chrome.Visible = $true;$ie.Navigate("https://smukx.github.io/hacked"); while ($ie.Busy) { Start-Sleep -Milliseconds 100 };$ie.FullScreen = $true;[System.Windows.Forms.SendKeys]::SendWait("{F11}")

```
<br>

#### Send some Main Keys :)
```
Start-Sleep -Seconds 2;
[System.Windows.Forms.SendKeys]::SendWait('%');
[System.Windows.Forms.SendKeys]::SendWait('{F4}');
[System.Windows.Forms.SendKeys]::SendWait('{ENTER});
```
Sending `Alt+F4` Command via powershell 