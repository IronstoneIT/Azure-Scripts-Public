# Define your tag name
$tagName = "maintenanceGroup"

# Authenticate to Azure using the managed identity
$AzureContext = (Get-AzContext).Name
if (-not $AzureContext) {
    Connect-AzAccount -Identity
}

# Get all subscriptions that the Automation Account has access to
$Subscriptions = Get-AzSubscription

# Loop through all subscriptions
foreach ($Subscription in $Subscriptions) {
    # Set the current subscription
    Set-AzContext -Subscription $Subscription

    # Get all VMs with the specified tag name
    $VMs = Get-AzVM -Status | Where-Object { $_.Tags.ContainsKey($tagName) }

    foreach ($VM in $VMs) {
        # Get the VM resource
        $Resource = Get-AzResource -ResourceId $VM.Id

        # Determine the OS type and update the patch settings accordingly
        if ($VM.StorageProfile.OsDisk.OsType -eq "Windows") {
            $Resource.Properties.osProfile.windowsConfiguration.patchSettings = @{
                "patchMode" = "AutomaticByPlatform"
                "automaticByPlatformSettings" = @{
                    "bypassPlatformSafetyChecksOnUserSchedule" = $true
                }
                "assessmentMode" = "AutomaticByPlatform"
            }
        } elseif ($VM.StorageProfile.OsDisk.OsType -eq "Linux") {
            $Resource.Properties.osProfile.linuxConfiguration.patchSettings = @{
                "patchMode" = "AutomaticByPlatform"
                "automaticByPlatformSettings" = @{
                    "bypassPlatformSafetyChecksOnUserSchedule" = $true
                }
                "assessmentMode" = "AutomaticByPlatform"
            }
        }

        # Update the VM resource
        Set-AzResource -ResourceId $Resource.ResourceId -Properties $Resource.Properties -Force
    }
}
