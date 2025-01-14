### Using Powershell to query DNS Information

PowerShell provides several functions and cmdlets that you can use to query DNS information. Here’s 
a list of some key functions and cmdlets related to DNS querying in PowerShell:

* Resolve-DnsName: This cmdlet is used to query DNS information, including resolving 
FQDNs to IP addresses and vice versa. It provides various query types, such as A (IPv4 address), 
AAAA (IPv6 address), MX, NS, and PTR (reverse lookup).

* Test-DnsServer: This cmdlet is specifically designed to test DNS servers for specific records. 
It helps you diagnose DNS server issues and verify the presence of DNS records.

* [System.Net.Dns]: PowerShell provides access to the .NET Framework’s System.Net.
Dns class, which contains static methods for querying DNS.

It’s important to note that some of these cmdlets and functions may require administrative privileges 
to perform certain DNS queries, especially for internal network resources. Also, keep in mind that the 
availability and behavior of these cmdlets and functions may vary based on your PowerShell version 
and the modules installed on your system. 

Before using any of these functions or cmdlets, it’s a good practice to refer to the official PowerShell 
documentation or use the built-in help system to understand their usage, options, and examples.

PowerShell provides the Resolve-DnsName cmdlet, which allows you to query DNS servers to 
resolve FQDNs into IP addresses. Here’s how you can use it

```
$FQDN = "www.snowcapcyber.com.com"
$Result = Resolve-DnsName -Name $FQDN
if ($Result.QueryResults.Count -gt 0) {
 $IPAddress = $Result.QueryResults[0].IPAddress
 Write-Host "IP address of $FQDN: $IPAddress "
} else {
 Write-Host "Unable to resolve IP for $FQDN"}
```
In this example, we do the following:

1. We define the FQDN to be resolved ($FQDN).
2. We use Resolve-DnsName to query the DNS server for the IP address of the given FQDN.
3. If the query results in at least one IP address, we extract and display the first IP address from 
the result.
4. If the query fails or no IP addresses are returned, an appropriate message is displayed.
Hence, the Resolve-DnsName cmdlet in PowerShell is a versatile tool that allows you to query
various types of DNS records, including MX, NS, and PTR records. Let’s explore each of these query 
types with examples:
* Querying MX records: MX records are used to specify the mail servers responsible for receiving 
email messages on behalf of a domain. You can use the Resolve-DnsName cmdlet to query 
MX records for a specific domain.

    Here’s an example:
```
$Domain = " snowcapcyber.com"
$MXRecords = Resolve-DnsName -Name $Domain -Type MX
if ($MXRecords) {
 $MXRecords | ForEach-Object {
 $MailServer = $_.NameExchange
 $Preference = $_.Preference
 Write-Host "MX Record: $Preference
Write-Host "Mail Server: $MailServer"}
} else {
 Write-Host "No MX records found for $Domain"}
```

In this example, replace snowcapcyber.com with the domain you want to query. The code 
queries and displays the MX records along with their preferences and associated mail servers.
