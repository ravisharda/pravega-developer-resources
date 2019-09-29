# .NET & Windows Commands

## Powershell

* Checking Windows update version: `[System.Environment]::OSVersion.Version`
* Invoking a Web request: `Invoke-WebRequest 'https://.....'`
* Fetching system information: `Get-ComputerInfo`
* Fetching disk info: `Get-Disk` and `Get-PhysicalDisk`
* Services
  * Starting: `Start-Service <service-name>`
  * Stopping: `Stop-Service <service-name>`
  * Cycle a service: `Restart-Service <service-name>`
* Performance
  * Find the five processes using the most memory: `ps | sort –p ws | select –last 5`
* Deleting all log files: `ls *.log -Recurse | foreach {rm $_}`  

## CMD

## Package managers

* See https://github.com/chocolatey/choco/wiki/CommandsList for Chocolatey package manager commands. 

