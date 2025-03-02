---
title: Unatended access to sharepoint
date: 2025-03-03 12:00:00 +0100
categories: [DevOps, Azure]
tags: [azure, devops, gcd]     # TAG names should always be lowercase
---


## Delegated credentials flow - preparation for application credentials flow
Delegated credentials in esence is a method for an application to act on behalf of a user. 
This method of interacting with service like Sharepoint, or any Azure Services, is not suitable for unatended access like CI/CD pipelines. If you can't use Sharepoint and need to use Onedrive then you can try solution desribed [here](https://marczak.io/posts/2023/03/azure-ad-graph-application-vs-delegated-flow/) but generaly is more tricky than application credential flow. 

I deliberately called this section 'preparation for application credentails flow' becuase I will not be providing solutions with delegated credentials flow. 

Instead I will use [Microsoft Graph Cli](https://learn.microsoft.com/en-us/graph/cli/overview) and [Azure Cli](https://learn.microsoft.com/en-us/cli/azure/) to setup everything. 


### Create sharpoint site
Since I want to limit the scope where my application have access to (and also not to mess up my companys existing site during testing) I have created a new sharepoint site. I have also removed default document library and created own with descriptive name. It is important to say that that document library is called drive in Graph Api. You can find short instruction for creating site [here](https://support.microsoft.com/en-us/office/create-a-site-in-sharepoint-4d1e11bf-8ddc-499d-b889-2b48d10b1ce8).


### Install [Azure Cli](https://learn.microsoft.com/en-us/cli/azure/)
Follow instructions given [here](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?pivots=winget).

### Install [Microsoft Graph Cli](https://learn.microsoft.com/en-us/graph/cli/overview)
Follow instructions given [here](https://learn.microsoft.com/en-us/graph/cli/installation?tabs=windows).

### Create App Registration

[Service Prinipal](https://learn.microsoft.com/en-us/entra/identity-platform/app-objects-and-service-principals?tabs=browser)

Service principal is an object which holds information about your application in Entra ID. The most important information from our point of view is where the application has access to - permissions. You can create it manually using Azure web interface, although it is easier to describe and repeat the process using set of commands.

#### Log in with [Azure Cli](https://learn.microsoft.com/en-us/cli/azure/)

Run the following command. This should open your browser automatically and perform authentication.
``` powershell
az login
```

#### Create new app registration
Run the following command and store appId from the output.
``` powershell
az ad app create `
  --display-name <your-app-name> `
```

### Assign permissions to app
When you do it for the first time it is probably easier to do it from azure portal. Once you click on permission you will see the same information I share here.

Also if you don't trust me on these permission( i would not copy paste this cryptic permissions from the internet ;)
maybe it is better to do it manually.

The permissions we need are:
* Sites.Selected
* Sites.Selected

These are Graph permissions.

=Role means this is application permision and not delegated permission

``` powershell
az ad app permission add `
  --id <appId> `
  --api 00000003-0000-0000-c000-000000000000 ` 
  --api-permissions 883ea226-0bf2-4a8f-9f9d-92c9162a727d=Role
```

Then you need to accpet it as admin. Az cli will output the command you need to run and will look like this.
``` powershell
az ad app permission grant --id <permission-id> --api 00000003-0000-0000-c000-000000000000
```



`Install-Module Microsoft.Graph -Scope CurrentUser`

`Import-Module Microsoft.Graph`

## Authenticate with own credentials
Running the following command will open browser and perform authentication.<br>
`Connect-MgGraph -Scopes "Sites.Read.All", "Sites.Manage.All"`


## List sites in your organisation (the one you have access to)

This a bit of a trick.

For the reason described [here](https://stackoverflow.com/questions/75917021/sites-list-getting-empty-results-in-graph-api)
it is not possible to use `https://graph.microsoft.com/v1.0/sites` and i did not want to complicate this post with creating application flow authentication.
`mgc sites list --search "**"` won't work when using  mgc (it does work when using graph explorer)

All the sharpoint sites will have a word "site" in it so it is use as 
`mgc sites list --search "site" --select id,webUrl,displayName`

## list drives in site

`mgc sites drives list --site-id {siteIdFromPreviousStep} --select id,name` 


## get root folder of the drive

`mgc drives root get --drive-id {driveIdFromPreviousStep}`


## get folders on the drive (children of root item)
`mgc drives items children list --drive-id  {driveIdFromPreviousStep} --drive-item-id {rootItemIdFromPreviousStep}  --select id,name,folder`


## list permission on selcted folder (any but will be working with root one)
`mgc drives items permissions list --drive-id  {driveIdFromPreviousStep} --drive-item-id {rootItemIdFromPreviousStep} `


## set permission

``` json
{
    "roles": [
        "fullcontrol"
    ],
    "grantedTo": {
        "application": {
            "id": "{applicationId}"
        }
    }
}
```

`{"roles":["fullcontrol"],"grantedTo":{"application":{"id":"{applicationId}"}}}`


## listing permisions to site
`mgc sites permissions list 

## giving permission to site
`mgc sites permissions create --site-id {siteIDdfkdfdf} --body '{"roles":["fullcontrol"],"grantedTo":{"application":{"id":"appIDKKDKDKDKDKFJDKFLKDF"}}}'`


## copying file
` mgc drives items content get --drive-id {driveId} --drive-item-id {dirveItemID} --output-file c:\test-file.txt`

## login in with application flow

