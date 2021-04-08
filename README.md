# DevOpsAgentDSC

PowerShell DSC __Class based Resource__ for installing Azure DevOps Agents

__Requirements__
* PowerShell Version 5.0 +
* Server 2012 +

```powershell
    # sample configuation data
@{
    AllNodes = @(
        @{
            NodeName                    = 'LocalHost'
            PSDscAllowPlainTextPassword = $true
            PSDscAllowDomainUser        = $true

            DevOpsAgentPoolPresent      = @(
                @{poolName = '{0}-{1}-{2}-{3}-Apps1' ; orgUrl = 'https://dev.azure.com/AzureDeploymentFramework/' },
                @{poolName = '{0}-{1}-{2}-{3}-Infra01' ; orgUrl = 'https://dev.azure.com/AzureDeploymentFramework/' }
            )

            DevOpsAgentPresent          = @(
                @{
                    name = '{0}-{1}-{2}-{3}-Apps101'; pool = '{0}-{1}-{2}-{3}-Apps1'; Ensure = 'Present';
                    Credlookup = 'DomainCreds' ; AgentBase = 'D:\vsts-agent' ; AgentVersion = '2.184.2'
                    orgUrl = 'https://dev.azure.com/AzureDeploymentFramework/'
                },
                @{
                    name = '{0}-{1}-{2}-{3}-Apps102'; pool = '{0}-{1}-{2}-{3}-Apps1'; Ensure = 'Present';
                    Credlookup = 'DomainCreds' ; AgentBase = 'D:\vsts-agent'; AgentVersion = '2.184.2'
                    orgUrl = 'https://dev.azure.com/AzureDeploymentFramework/'
                },
                @{
                    name = '{0}-{1}-{2}-{3}-Infra01'; pool = '{0}-{1}-{2}-{3}-Infra01'; Ensure = 'Present';
                    Credlookup = 'DomainCreds' ; AgentBase = 'D:\vsts-agent'; AgentVersion = '2.184.2'
                    orgUrl = 'https://dev.azure.com/AzureDeploymentFramework/'
                }
            )
        }
    )
}
```


```powershell


configuration DevOpsAgentDSC 
{
    param (
        [string]$Prefix = 'ACU1',
        [string]$OrgName = 'BRW',
        [string]$AppName = 'HAA',
        [string]$Enviro = 'D3',
        [PSCredential]$PAT
    )
    Import-DscResource -ModuleName DevOpsAgentDSC

    node $Node.NodeName
    {
        Foreach ($DevOpsAgentPool in $node.DevOpsAgentPoolPresent)
        {
            $poolName = $DevOpsAgentPool.poolName -f $Prefix, $OrgName, $AppName, $Enviro
                
            DevOpsAgentPool $poolName
            {
                PoolName = $poolName
                PATCred  = $PAT #$credLookup['DevOpsPAT']
                orgURL   = $DevOpsAgentPool.orgUrl
            }
        }

        Foreach ($DevOpsAgent in $node.DevOpsAgentPresent)
        {
            $agentName = $DevOpsAgent.name -f $Prefix, $OrgName, $AppName, $Enviro
            $poolName = $DevOpsAgent.pool -f $Prefix, $OrgName, $AppName, $Enviro
            
            DevOpsAgent $agentName
            {
                PoolName     = $poolName
                AgentName    = $agentName
                AgentBase    = $DevOpsAgent.AgentBase
                AgentVersion = $DevOpsAgent.AgentVersion
                orgURL       = $DevOpsAgent.orgUrl
                Ensure       = $DevOpsAgent.Ensure
                PATCred      = $PAT #$credLookup['DevOpsPAT']
                # Credential = $credLookup[$Agent.Credlookup]
            }
        }
    }
}
```

Full sample available here

- DSC Configuration
    - [ADF/ext-DSC/DSC-AppServers.ps1](https://github.com/brwilkinson/AzureDeploymentFramework/blob/main/ADF/ext-DSC/DSC-AppServers.ps1#L882)
- DSC ConfigurationData
    - [ADF/ext-CD/JMP-ConfigurationData.psd1](https://github.com/brwilkinson/AzureDeploymentFramework/blob/main/ADF/ext-CD/JMP-ConfigurationData.psd1#L54)