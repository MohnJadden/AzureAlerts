## Auto-resolve on Azure alert rules can't be switched off and on via a schedule.  If it's on, then auto-resolve won't actually fire action groups.
## This script will allow you to leave auto-resolve off, which in turn will allow action groups to fire.  The script will clear out existing alerts after a period of time which you set as the $timeRange parameter. 
## The script will run in an Automation Account.  You will need to specify or provide the following parameters: 
## Replace the <alertRuleID> with the rule ID of the alert rule.  This can be found in the properties of the alert rule itself - for example, in the portal, go to Monitor -> Alert Rules -> click on the rule in question -> Properties -> Resource ID field.
## Replace the <timeRange> with how far back you want the rule to check - e.g. 30m, 1h, 15d, etc.  

$alertRuleId = "<alertRuleID>"
$timeRange = "<timeRange>

# Authenticate to Azure by using the Run As service principal and certificate.  Leave this part of the script alone.
        Write-Output "Authenticating to Azure with service principal and certificate"
        $ConnectionAssetName = "AzureRunAsConnection"
        Write-Output "Get connection asset: $ConnectionAssetName"
        $Conn = Get-AutomationConnection -Name $ConnectionAssetName
        if ($Conn -eq $null) {
            throw "Check if $ConnectionAssetName exists in the Automation account."
        }
        Write-Output "Authenticating to Azure with service principal."
        Connect-AzAccount -ServicePrincipal `
            -Tenant $Conn.TenantID `
            -ApplicationId $Conn.ApplicationID `
            -CertificateThumbprint $Conn.CertificateThumbprint | Write-Output


# List any instances of the alert from the previous day
Get-AzAlert -MonitorCondition "Fired" -State "New" -AlertRuleID $alertRuleID -TimeRange $timeRange |

# Select the individual alerts and split out their IDs to the $_.Id value
Select @{n='Id';e={
            $_.Id.split('/')[-1] 
        }} | 
% { # Close the alert
    Update-AzAlertState -AlertId $_.Id -State "Closed"
}
