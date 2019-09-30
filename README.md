# PowerBI
Power Bi scripts
   
   Puropse:
    Puropse of this script is to get all refresh times for all workspaces in dedicated capacity. As we know, at the momnet, 
    there is no easy tool to present at what time, what dataset is refreshing and at what time.
    With this script we can get a list of workspaces, datasets and time when it's set to refresh.
   
   Requirements:
   - PowerBI PowerShell module
     https://docs.microsoft.com/en-us/powershell/power-bi/overview?view=powerbi-ps
   - You need to be admin to every workspace that you want to analyze
   - You need to enter Premium node ID
        Get-PowerBICapacity | Select ID,DisplayName
   - Excel skills are requied to comapre result and get a list of refreshes per day/time

   What scrip will give you:
   
    1. List of workspaces on the node
    2. List of datasets in a workspaces
    3. List of reports in workspace
    4. List of refreshes per dataser



# Connect to PowerBI
Connect-PowerBIServiceAccount

# Capacity
$capacityID = 'abcdefgh-0000-0000-0000-abcdefgh'

# List of workspaces on the node
$CapacityWorkspace = Get-PowerBIWorkspace -All -Scope Organization | Where-Object {$_.CapacityId -eq $capacityID}
write-Host "Number of workspaces in capacity:" $CapacityWorkspace.Count

# List of datasets in a workspaces
$AllDatasets = ""
$i=0
foreach($workspace in $CapacityWorkspace)
                                        {
    $i++
    Write-Host "Working on Workspace:" $workspace.name " | Count:" $i "from:" $CapacityWorkspace.count
    $result = Get-PowerBIDataSet -WorkspaceId $workspace.Id
                foreach($rst in $result)
                                {
                                    $line = "" | Select WorkspaceID,WorkspaceName,DatasetId,DatasetName,IsRefreshable,IsOnPremGatewayRequired,ConfiguredBy
                                    $line.WorkspaceID = $workspace.Id
                                    $line.WorkspaceName = $workspace.Name
                                    $line.DatasetId = $rst.Id
                                    $line.DatasetName = $rst.Name
                                    $line.IsRefreshable = $rst.IsRefreshable
                                    $line.IsOnPremGatewayRequired = $rst.IsOnPremGatewayRequired
                                    $line.ConfiguredBy = $rst.ConfiguredBy
                                    [array]$AllDatasets += $line
                                }
                                       }

# List of reports in workspace
$AllReports = ""
$i=0
foreach($workspace in $CapacityWorkspace)
                                        {
    $i++
    Write-Host "Working on Workspace:" $workspace.name " | Count:" $i "from:" $CapacityWorkspace.count
    $result = Get-PowerBIReport -WorkspaceId $workspace.Id
                foreach($rst in $result)
                                {
                                    $line = "" | Select WorkspaceID,WorkspaceName,DatasetId,ReportId,ReportName
                                    $line.WorkspaceID = $workspace.Id
                                    $line.WorkspaceName = $workspace.Name
                                    $line.ReportId = $rst.id
                                    $line.ReportName = $rst.Name
                                    $line.DatasetId = $rst.DatasetId
                                    [array]$AllReports += $line
                                }
                                        }



# List of refreshes per dataset
$i=0
$Datasets_WithoutMetrics = $AllDatasets | Where-Object {$_.DatasetName -notlike '*Usage Metrics*'}
foreach ($dataset in $Datasets_WithoutMetrics)
            {
                    $i++
                    Write-Host "Dataset:" $dataset.DatasetName
                    Write-Host "Count:" $i "of" $Datasets_WithoutMetrics.Count
                    $grp = $dataset.WorkspaceID.Guid
                    $dset = $dataset.DatasetId.Guid
                    $uri = "https://api.powerbi.com/v1.0/myorg/groups/$grp/datasets/$dset/refreshSchedule"
                    $Invoke_Result = Invoke-PowerBIRestMethod -Url $uri –Method GET | ConvertFrom-Json | Select days,times
                    $days = ($Invoke_Result  | Select days).days
                    $times = ($Invoke_Result | Select times).times
                    
                    foreach($day in $days){
                                                Write-Host "Day:" $day
                                foreach ($time in $times){
                                                Write-Host "Time:" $time
                                                $result = "" | Select workspaceName,workspaceID,datasetName,datasetID,RefreshDate,RefreshTime
                                                $result.workspaceName = $dataset.WorkspaceName
                                                $result.WorkspaceID = $dataset.WorkspaceID
                                                $result.DatasetName = $dataset.DatasetName
                                                $result.DatasetID = $dataset.DatasetId
                                                $result.RefreshDate = $day
                                                $result.RefreshTime = $time
                                                $result | Export-Csv refreshes.csv -Append -NoTypeInformation
                                                        }
                                              }
            }

Write-Host "Script is finished"
