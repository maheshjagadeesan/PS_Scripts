# PS_Scripts





#// Start of script 
#// Get year and month for csv export file 
$DateTime = Get-Date -f "yyyy-MM-dd"

$da = Get-Date
$da = $da.ToString('MM-dd-yyyy') 

#// Set CSV file name 
$CSVFile = "C:\kits\reports\PROD_ADGroups_$DateTime.csv" 


#// Create emy array for CSV data 
$CSVOutput = @() 

#// Get all AD groups in the domain 
$ADGroups = Get-ADGroup -Filter * -Properties *
#$ADGroups = Get-ADGroup -Filter {sAMAccountName -like '*EDM*'} -Properties * 

#// Set progress bar variables 
$i=0 
$tot = $ADGroups.count 

foreach ($ADGroup in $ADGroups) { 
#// Set up progress bar 
$i++ 
$status = "{0:N0}" -f ($i / $tot * 100) 
Write-Progress -Activity "Exporting AD Groups" -status "Processing Group $i of $tot : $status% Completed" -PercentComplete ($i / $tot * 100) 

#// Ensure Members variable is empty 
$Members = "" 

#// Get group members which are also groups and add to string 
$MembersArr = Get-ADGroup -filter {Name -eq $ADGroup.Name} | Get-ADGroupMember | select Name, objectClass, distinguishedName
$MembersCount = $MembersArr.count 
if ($MembersArr) { 
foreach ($Member in $MembersArr) { 
if ($Member.objectClass -eq "user") { 
$MemDN = $Member.distinguishedName 
$UserObj = Get-ADUser -filter {DistinguishedName -eq $MemDN} 
#if ($UserObj.Enabled -eq $False) { 
# continue 
#} 
} 
$Members = $Members + "," + $Member.Name 
} 
#// Check for members to avoid error for empty groups 
if ($Members) { 
$Members = $Members.Substring(1,($Members.Length) -1) 
} 
} 

#// Set up hash table and add values 
$HashTab = $NULL 
$HashTab = [ordered]@{ 
"Name" = $ADGroup.Name
"GroupMembersCount" = $MembersCount
"Description" = $ADGroup.description 
"Category" = $ADGroup.GroupCategory 
"Scope" = $ADGroup.GroupScope 
"Members" = $Members 
} 

#// Add hash table to CSV data array 
$CSVOutput += New-Object PSObject -Property $HashTab 
} 

#// Export to CSV files 
$CSVOutput | Sort-Object Name | Export-Csv $CSVFile -NoTypeInformation 

#// End of script

$body = "Seqirus AD group Inventory_$da"

#// email the report







Regards,
Mahesh

This email and any attachments are confidential and may be subject to legal or other professional privilege. Any confidentiality or privilege is not waived or lost because this email has been sent to you by mistake. You should not read, copy, adapt, use or disclose them or their contents without authorization. Any personal information in this email must be handled in accordance with all applicable privacy laws. If you are not an intended recipient of the email, please disregard its contents, contact the sender at once by return email and then delete both messages. Seqirus – A CSL Group company. 
