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
