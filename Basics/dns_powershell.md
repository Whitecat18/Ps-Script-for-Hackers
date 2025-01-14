# DNS and Powershell

PowerShell provides several functions and cmdlets that you can use to query DNS information. Here’s
a list of some key functions and cmdlets related to DNS querying in PowerShell.

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
resolve FQDNs into IP addresses. Here’s how you can use it:

```
$FQDN = "www.snowcapcyber.com.com"
$Result = Resolve-DnsName -Name $FQDN
if ($Result.QueryResults.Count -gt 0) {
    $IPAddress = $Result.QueryResults[0].IPAddress
    Write-Host "IP address of $FQDN: $IPAddress"
} else {
    Write-Host "Unable to resolve IP for $FQDN"
}
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
        XRecords | ForEach-Object {
        $MailServer = $_.NameExchange
        $Preference = $_.Preference
        Write-Host "MX Record: $Preference"
        Write-Host "Mail Server: $MailServer"
} else {
      Write-Host "No MX records found for $Domain"
}
```

In this example, replace snowcapcyber.com with the domain you want to query. The code
queries and displays the MX records along with their preferences and associated mail servers.

* Querying NS records: NS records are used to map a domain to the authoritative name servers
that are responsible for resolving queries for that domain.

Here’s how you can query NS records using the Resolve-DnsName cmdlet:

```
$Domain = "snowcapcyber.com"
$NSRecords = Resolve-DnsName -Name $Domain -Type NS
if ($NSRecords) {
     $NSRecords | ForEach-Object {
         $NameServer = $_.NameHost
         Write-Host "Name Server: $NameServer"
} else {
    Write-Host "No NS records found for $Domain"
}
```
Replace snowcapcyber.com with the domain you want to query. The code queries and
displays the NS records associated with the domain.

* Querying PTR records: PTR records are used for reverse DNS lookups, translating IP addresses
back into domain names. Reverse DNS is often used for security and network troubleshooting.

Here’s an example of querying PTR records:

```
$IPAddress = "192.168.27.132"
$PTRRecords = Resolve-DnsName -Name $IPAddress -Type PTR
if ($PTRRecords) {
    $PTRRecords | ForEach-Object {
        $DomainName = $_.NameHost
        Write-Host "PTR Record: IP Address $IPAddress resolves to $DomainName"
} else{
    Write-Host "No PTR records found for IP address $IPAddress"
}
```

The Resolve-DnsName cmdlet is a powerful tool in PowerShell that allows you to query
various DNS records, including MX, NS, and PTR records. By using this cmdlet with different
query types, you can gather important information about mail servers, authoritative name
servers, and reverse DNS lookups as part of a penetration test. These examples demonstrate
how PowerShell can be utilized for DNS-related tasks, aiding network administrators and IT
professionals in managing and diagnosing network resources.

The Test-DnsServer cmdlet can be a valuable addition to a penetration tester’s toolkit 
when assessing the DNS infrastructure of a target. Here’s how it can be used:

* DNS enumeration: During the information-gathering phase, a penetration tester might want 
to gather DNS-related information about the target network. Using Test-DnsServer, they 
can enumerate DNS records to uncover valuable information such as domain names, mail 
exchange servers, and authoritative name servers. This information helps the tester build a 
comprehensive profile of the target. Some DNS records, such as SVR, can be used to identify 
detailed service information and, hence, potential vulnerabilities

Here’s an example:

```
$Domain = "snowcapcyber.com"
$DnsServer = "192.168.1.1"
# Enumerate MX records
$MXRecords = Test-DnsServer -IPAddress $DnsServer -Name $Domain 
-Type MX
if ($MXRecords) {
    Write-Host "MX records for $Domain:"
    $MXRecords.QueryResults | ForEach-Object {
    Write-Host "Server: $($_.MailExchange)" }
} else {
    Write-Host "No MX records found for $Domain"}
```

* DNS spoofing and cache poisoning tests: Penetration testers may attempt to exploit DNS 
vulnerabilities such as cache poisoning to redirect traffic to malicious servers. By testing the 
target’s DNS responses using Test-DnsServer, they can assess whether the DNS server is 
vulnerable to spoofing attacks.

Here’s an example:

```
$DnsServer = "192.168.1.1"
$MaliciousServer = "malicious.com"
$TargetDomain = "snowcapcyber.com"

# Test if DNS server resolves to malicious IP
$DnsResponse = Test-DnsServer -IPAddress $DnsServer -Name 
$TargetDomain -Type A
if ($DnsResponse.QueryResults.IPAddress -eq "malicious_IP") {
    Write-Host "Server vulnerable to spoofing."
} else {
    Write-Host "DNS server is not vulnerable."}
```
