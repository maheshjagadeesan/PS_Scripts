$MBFULLC = @()
$MBFULL = @()
$SendAsAccess = @()
$groups = Get-Content C:\kits\Script\SharedMailbox.txt
Foreach ($grp in $groups)
{ 
Write-Host "$grp"

$mb = Get-MailboxPermission -Identity ${grp} | ? {$_.IsInherited -ne "true" -and $_.User -ne "NT AUTHORITY\SELF"} | select *
$MBFULL += $mb

$mbC = Get-MailboxfolderPermission -Identity ${grp}:\calendar | ? {$_.IsInherited -ne "true" -and $_.User -ne "NT AUTHORITY\SELF"} | select *
$MBFULLC += $mbC

$Sendas = Get-RecipientPermission -Identity $grp | ? {$_.IsInherited -ne "true" -and $_.Trustee -ne "NT AUTHORITY\SELF"} | select *
$SendAsAccess += $Sendas
}

$MB = $MBFULL | ft -Wrap -AutoSize | Out-String
$MBC = $MBFULLC | ft -Wrap -AutoSize | Out-String
$SA = $SendAsAccess | ft -Wrap -AutoSize | Out-String
$body = "Mailbox Permission: $grp`n`n1. Full access:`n`n$MB`n`n2.Calendar Access:`n`n$MBC`n`n 3. SendAs Access:`n`n$SA"
Send-MailMessage -From "reports@seqirus.com" -To "reports@seqirus.com" -Bcc "mahesh.jagadeesan@seqirus.com" -Subject "Mailbox_(Full_Caldendar_SendAs)Permission_report($grp)_$StartDate" -SmtpServer "10.112.47.68" -Body "$body"
