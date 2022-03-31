# Automation-of-SAP-Hybris-Commerce-Cloud-using-Azure-Devops-CI-CD

![image](https://user-images.githubusercontent.com/90609419/161049453-1d44b46f-168f-4e77-a077-511d85d1903c.png)


$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", "Bearer ")
$headers.Add("Content-Type", "application/json")
$body = @{
  "applicationCode"= "commerce-cloud";
  "branch"= "";
  "name"= ""
}

$body =$body  | ConvertTo-Json 

$response = Invoke-RestMethod 'https://portalrotapi.hana.ondemand.com/v2/subscriptions//builds' -Method 'POST' -Headers $headers -Body $body
$resp1 = $response.code


Do {
Start-Sleep -s 90
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", "Bearer ")
$headers.Add("Content-Type", "application/json")

$response1 = Invoke-RestMethod "https://portalrotapi.hana.ondemand.com/v2/subscriptions//builds/$resp1/progress" -Method 'GET' -Headers $headers
$response1 | ConvertTo-Json
$status = $response1.buildStatus}
until ($status -eq "Success"  -or $status -eq "FAIL" )

if ($status -eq "Success")
{
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", "Bearer ")
$headers.Add("Content-Type", "application/json")

 $body = @{
  "buildCode"= $response.code;
  "databaseUpdateMode"= "NONE";
  "environmentCode"= ""
  "strategy"= "ROLLING_UPDATE"
}


$body =$body  | ConvertTo-Json
$body

$response2 = Invoke-RestMethod 'https://portalrotapi.hana.ondemand.com/v2/subscriptions//deployments' -Method 'POST' -Headers $headers -Body $body
Write-Host "Build is finished Successfully ,Deployment is started"
$resp2 = $response2.code
$resp2

Do {
Start-Sleep -s 90
$response3 = Invoke-RestMethod "https://portalrotapi.hana.ondemand.com/v2/subscriptions//deployments/$resp2/progress" -Method 'GET' -Headers $headers
$response3 | ConvertTo-Json
$status2 = $response3.deploymentStatus}
until ($status2 -eq "DEPLOYED"  -or $status2 -eq "FAIL")

if ($status2 -eq "DEPLOYED")
{
Write-Host "Deployment is Successful Proceeding to Purge the akamai"
}

else
{
Write-Host "Deployment is Failed"
}
}
else 
{
write-host("Your Build is Failed please check on SAP portal for the build logs ")
}

