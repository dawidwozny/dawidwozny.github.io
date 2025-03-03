---
title: Unatended access to sharepoint
date: 2025-03-03 12:00:00 +0100
categories: [DevOps, Azure]
tags: [azure, devops, sharepoint]     # TAG names should always be lowercase
---

# Sharepoint - unatended access
Being able to upload some files from CI/CD pipeline directly to Sharpoint was something on my wish list for a long time.
It was possible possible but at the cost of security - the pipeline would have access to all the Sharepoint sites in your organisation.

This changed when [Site.Selected](https://learn.microsoft.com/en-us/graph/permissions-selected-overview?tabs=http) permission had been introduced. 
Once I read a blog [post](https://learn.microsoft.com/en-us/graph/permissions-selected-overview?tabs=http) giving overview of the solution I decided to give it a go.

I will save the complete solution with CI/CD pipeline for another time and lay foundations here.


## Setup - Sharepoint
Since I want to limit the scope where my application have access to (and also not to mess up my companys existing site during testing) I have created a new sharepoint site. I have also removed default document library and created own with descriptive name. It is important to say that that document library is called drive in Graph Api. You can find short instruction for creating site [here](https://support.microsoft.com/en-us/office/create-a-site-in-sharepoint-4d1e11bf-8ddc-499d-b889-2b48d10b1ce8).


## Setup - EntraID

[Service Prinipal](https://learn.microsoft.com/en-us/entra/identity-platform/app-objects-and-service-principals?tabs=browser)

Service principal is an object which holds information about your application in Entra ID. The most important information from our point of view is where the application has access to - permissions. You can create it manually using Azure web interface, although it is easier to describe and repeat the process using set of commands.

### Install Azure Cli
Follow instructions given [here](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?pivots=winget).

### Log in with Azure Cli
``` powershell
az login
```
### Create app registration
``` powershell
az ad app create `
  --display-name <your-app-name>
```

### Create service principal for the app registration
``` powershell
 az ad sp create --id <app-id>
 ```
> Store id from the output. This is something which is know as client-id and we will need that in the following steps.

### Create client secret
``` powershell
az ad app credential reset `
  --id <app-id> `
  --display-name <secret-name> 

 ```
> Store password from the output. This is something which is know as client-secret and will need that in the following steps.<br>
> Store tennant from the output. This is somethin known as tenant-id and will nee dthat later.

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

`00000003-0000-0000-c000-000000000000` is Microsoft Graph Api Id.<br>
`883ea226-0bf2-4a8f-9f9d-92c9162a727d` is Site.Selected permission.<br>
`883ea226-0bf2-4a8f-9f9d-92c9162a727d=Role` means that Site.Selected permission is given to application and it is not delegated permission


### Grant admin consnent
``` powershell
 az ad app permission admin-consent --id <app-id> 
 ```

## Setup - Microsoft Graph

### Install Microsoft Graph Cli
Follow instructions given [here](https://learn.microsoft.com/en-us/graph/cli/installation?tabs=windows).

### Login
Running the following command will open browser and perform authentication.<br>
``` powershell
 mgc login --scopes Sites.FullControl.All --strategy InteractiveBrowser
```

### List sites
This step is a bit tricky.
For the reason described [here](https://stackoverflow.com/questions/75917021/sites-list-getting-empty-results-in-graph-api)
it is not possible to use simple `https://graph.microsoft.com/v1.0/sites` and i did not want to complicate this post with creating application flow authentication.
`mgc sites list --search "**"` won't work when using  mgc (it does work when using graph explorer)

All the sharpoint sites will have a word "site" in it so you can use the folowing command. Additionaly the output  is limited only to id, webUrl and displayName.
``` powershell
mgc sites list --search "site" --select id,webUrl,displayName`
```
The display name should match the name of your Sharpoint site.

### List site permisions
``` powershell
 mgc sites permissions list --site-id <site-id>
```

### Set site permissions
```powershell
mgc sites permissions create
 --site-id <site-id>
 --body {"roles":["fullcontrol"],"grantedTo":{"application":{"id":"<client-id>"}}}
```

### List drives in site
``` powershell
mgc sites drives list --site-id <site-id> --select id,name
```
The name should match document library name in Sharepoint.

### Get root folder of the drive
Files are not stored directly on drive object but on the root of the drive. Therfore we need to find it's id.

``` powershell
mgc drives root get --drive-id  <drive-id>
```

### List folders on the drive's root item
``` powershell
mgc drives items children list `
 --drive-id  <drive-id> `
 --drive-item-id <root-item-id> `
 --select id,name,folder
```

### List permission on selcted drive item (folder)
``` powershell
mgc drives items permissions list `
  --drive-id <drive-id> `
  --drive-item-id <root-item-id> 
```
It is possible that there will be no permissions set at all. Run the command again once you set the new ones.

### Set drive item (folder) permission 

``` powershell
mgc drives items permissions create
  --drive-id <drive-id>
  --drive-item-id <root-item-id>
  --body {"roles":["fullcontrol"],"grantedTo":{"application">:{"id":"<client-id>"}}}
```

For simplicity I have granted full control here.<br>
client-id-from-service-principal is value from EntraID setup mentioned earlier.


## Testing - delegated credentials flow
This something what we could do in the begining because when were using mgc we actually used 'delegated credentials flow'
This is not so exciting but I always start with something simple and working, then gradually add complexity.

### List drive itmes (folder/files) in drive item
We were doing it before on root-item in order to add permission but this is command we can use on any foler.
``` powershell
mgc drives items children list `
 --drive-id  <drive-id> `
 --drive-item-id <drive-item-id> `
 --select id,name,folder
```

### Copy file from Sharepoint
Upload some file to Sharpoint manually. Find it's id using `mgc drives items children list` command and then download.

``` powershell
mgc drives items content get
  --drive-id <drive-id>
  --drive-item-id <drive-item-id>
  --output-file c:\test-file.txt 
```

### Logout
``` powershell
 mgc logout
 ```

## Testing - application credentials flow
Now the fun starts. Provided we have done everything correctly we should be able to perform all the tests but not using our credentials but application credentials.

### Login
There are various ways of doing it. For full list of strategies run `mgc login --help`. I have decided on simplicit here (at least for auth) therfore using enviroment variables and client secret.

#### Set enviromental variables:
This is not the best method because variables will be stored in your powershell history but I will be deleting the secret after initial test anyway. Choose your own way of setting envariomental variables.
``` powershell
$env:AZURE_TENANT_ID = <tenant-id>
$env:AZURE_CLIENT_ID = <client-id>
$env:AZURE_CLIENT_SECRET = <client-secret>
```

``` powershell
mgc login --strategy Environment
```

### Test with application credentials flow
This where fun starts. If the setup is correct you will be able to do exactly the same test as with delegated credentials.

## Extra commands
### Delete permission

