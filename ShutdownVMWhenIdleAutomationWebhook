## This script is meant to be created as a Runbook and called as an Automation Runbook from an Automation Account in an Azure Monitor Alert Rule.  
## When triggered, it will immediately stop and deallocate the VM name passed from the alert rule.

param
(
    [Parameter (Mandatory = $false)]
    [object] $WebhookData
)

# If runbook was called from Webhook, WebhookData will not be null.
if ($WebhookData) {
    # Here for Debugging purposes
    Write-Output $WebhookData
    
    # Get the data object from WebhookData
    $WebhookBody = (ConvertFrom-Json -InputObject $WebhookData.RequestBody)
    # Get the info needed to identify the VM (depends on the payload schema)
    $schemaId = $WebhookBody.schemaId
    Write-Verbose "schemaId: $schemaId" -Verbose

    if ($schemaId -eq "AzureMonitorMetricAlert") {
        # This is the near-real-time Metric Alert schema
        $AlertContext = [object] ($WebhookBody.data).context
        Write-Output $AlertContext
        $SubId = $AlertContext.subscriptionId
        $ResourceGroupName = $AlertContext.resourceGroupName
        $ResourceType = $AlertContext.resourceType
        $ResourceName = $AlertContext.resourceName
        $status = ($WebhookBody.data).status
    } else {
        # Schema not supported
        Write-Error "The alert data schema - $schemaId - is not supported."
        throw "Not coming from AzureMonitorMetricAlert schema"
    }
    # Stop for VMs only
    if ($ResourceType -eq "Microsoft.Compute/virtualMachines")
    {
        # Authenticate to Azure by using the service principal and certificate.
        Write-Output "Authenticating to Azure with service principal and certificate"
        $ConnectionAssetName = "AzureRunAsConnection"
        Write-Output "Get connection asset: $ConnectionAssetName"
        $Conn = Get-AutomationConnection -Name $ConnectionAssetName
        if ($Conn -eq $null) {
            throw "Check if $ConnectionAssetName exists in the Automation account."
        }
        Write-Output "Authenticating to Azure with service principal."
        Add-AzureRmAccount -ServicePrincipal `
            -Tenant $Conn.TenantID `
            -ApplicationId $Conn.ApplicationID `
            -CertificateThumbprint $Conn.CertificateThumbprint | Write-Output

        if ($schemaId -eq "AzureMonitorMetricAlert") {
            Stop-AzureRmVM -Force `
                -Name $ResourceName `
                -ResourceGroupName $ResourceGroupName
            
            Write-Output "Stopping $ResourceName"
        }  
    }
} else {
    # Error
    write-Error "This runbook is meant to be started from an Azure alert webhook only."
}
