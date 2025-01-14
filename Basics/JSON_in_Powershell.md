# JSON IN POWERSHELL

Penetration testing is a critical activity that involves simulating real-world attacks to identify vulnerabilities and weaknesses in a system or network. PowerShell, a powerful scripting language native to the Windows environment, is a valuable tool for penetration testers due to its flexibility, extensive automation capabilities, and ability to interact with web services and APIs. In this section, we will explore how PowerShell can be used to handle JSON data as part of penetration testing. We will cover scenarios such as retrieving JSON data from web APIs, parsing JSON responses, extracting valuable information from JSON objects, and manipulating JSON payloads for testing purposes.

## Retrieving JSON data from web APIs

Penetration testers often need to interact with web APIs to gather information or perform assessments. PowerShell can be used to make HTTP requests to APIs and retrieve JSON data. This can be achieved using the Invoke-RestMethod cmdlet, which simplifies the process of making HTTP requests and handling responses


```
$repoUrl = "https://api.snowcapcyber.com/repo" 
$response = Invoke-RestMethod -Uri $repoUrl 
$response
```

In this example, we are sending an HTTP GET request to the specified URL using the InvokeRestMethod cmdlet. The response will be in JSON format, and PowerShell will automatically convert it into a PowerShell object. This makes it easier to access and manipulate the data.

### Parsing JSON data

Once the JSON data is retrieved, it needs to be parsed to extract specific information. PowerShell provides the ConvertFrom-Json cmdlet to convert JSON data into PowerShell objects, making it easy to access individual elements. Let’s parse the JSON response from the GitHub API to extract the repository’s name and description


```
$repoUrl = "https://api.snowcapcyber.com/repo" $response = Invoke-RestMethod -Uri $repoUrl $repoObject = ConvertFrom-Json $response Write-Host "Repository Name: $($repoObject.name)" Write-Host "Description: $($repoObject.description)"
```

In this example, we use the ConvertFrom-Json cmdlet to convert the JSON response into a PowerShell object named $repoObject. We can then access specific properties of the object, such as the repository name and description.

### JSON manipulation for payloads

During penetration testing, manipulating JSON data is essential, especially when crafting payloads for web application testing. PowerShell can easily create, modify, and send JSON payloads. Let’s create a JSON payload for an HTTP POST request to a vulnerable API:

```
$payload = @{ 
	"username" = "admin" 
	"password" = "P@ssw0rd123" 
} | ConvertTo-Json 
$headers = @{ 
	"Content-Type" = "application/json" } 
Invoke-RestMethod -Uri "https://snowcapcyber.com/api/login" -Method Post -Body $payload -Headers $headers
```

In this example, we define a PowerShell hash table named $payload with username and password fields. We then use the ConvertTo-Json cmdlet to convert the hash table into a JSON payload. The Invoke-RestMethod cmdlet sends the JSON payload in an HTTP POST request to the specified API

### Interacting with JSON from files

Penetration testers often deal with JSON data stored in files. PowerShell provides easy ways to read and write JSON data to/from files. Let’s read JSON data from a file, add a new property, and then save it back to the file:

```
$jsonFilePath = "C:\path\to\file.json" 
$jsonData = Get-Content -Raw -Path $jsonFilePath | ConvertFrom-Json 

# Add a new property $jsonData | Add-Member -Name "role" -Value "admin" 

# Save updated JSON back to the file $jsonData | ConvertTo-Json | Set-Content -Path $jsonFilePath
```

In this example, we read JSON data from a file using the Get-Content cmdlet. The -Raw parameter ensures that the content is read as a single string rather than an array of lines. We then convert the JSON content into a PowerShell object named $jsonData. After adding a new property to the object, we use the ConvertTo-Json cmdlet to convert it back to JSON format and save it back to the file using the Set-Content cmdlet.


### Web scraping and data extraction

In some scenarios, penetration testers may need to extract specific information from web pages containing JSON data. PowerShell can interact with web pages, extract JSON content, and process it accordingly. Let us extract information from a web page containing JSON data and display it:

```
$url = "https://snowcapcyber.com/data.json" 
$response = Invoke-RestMethod -Uri $url 
$data = $response.data 
foreach ($item in $data) { 
	Write-Host "Name: $($item.name)" 
	Write-Host "Age: $($item.age)" 
	Write-Host "Occupation: $($item.occupation)" 
	Write-Host ""
	}
```

In this example, we use the Invoke-RestMethod cmdlet to retrieve the JSON data from the specified URL. The response is then stored in the $response variable. We assume that the JSON data contains an array of objects with Name, Age, and Occupation properties. We use a foreach loop to iterate through each object in the array and display the extracted information. 

PowerShell is a valuable tool for processing JSON data as part of penetration testing. Its native support for JSON manipulation, ease of making web requests, and ability to parse JSON responses make it a versatile choice for working with JSON-based APIs and web services. As a penetration tester, understanding how to effectively process JSON data in PowerShell can significantly enhance your ability to gather information, exploit vulnerabilities, and carry out various security assessments. From retrieving JSON data from web APIs to crafting JSON payloads and handling parsing errors, PowerShell’s capabilities in dealing with JSON data are indispensable in the penetration tester’s toolkit.
