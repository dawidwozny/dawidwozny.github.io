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
Once I read this [blog post](https://learn.microsoft.com/en-us/graph/permissions-selected-overview?tabs=http) I decided to give it a go.

This is not going to be full CI/CD pipeline solution but fundations allowing application to manipulate files on Sharepoint. What is most important, the application will use 'application credential flow' meaning it will have own identity and will have full access to single Sharepoint site. 

In the future I plan to investigate more granural level access and describe using [rclone](https://rclone.org/) for upload of the artifacts.

## Setup - Sharepoint
The first step is to create Sharepoin site. You can find short instruction for creating site [here](https://support.microsoft.com/en-us/office/create-a-site-in-sharepoint-4d1e11bf-8ddc-499d-b889-2b48d10b1ce8). Something to keep in mind is that document library is called drive when accessing it through Microsoft Graph Api

## Setup - EntraID
The next thing is to setup App Registration in EntraID. App Registration is an object in Azure holding information about your application. One of these information are api permissions the application have.

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

 The output be like this:
``` json
{
  "appId": "cryptic-guid",
  "password": "cryptic-password",
  "tenant": "cryptic-guid"
}
```

> Store **password**. I will call it **client-secret** in the following steps.<br>
> Store **tennant**. I will call it **tenant-id** in the following steps.<br>
> Store **appId**. I will call it **client-id** in the following steps.

### Assign permissions to app
When you do it for the first time it is probably easier to do it from azure portal. However, I am writing this post also for my future self and I prefer to do it this way. Once you click on permission azure portal you will see the same information I am sharing here.

We need give Sites.Selected permission to Graph Api.

``` powershell
az ad app permission add `
  --id <appId> `
  --api 00000003-0000-0000-c000-000000000000 `
  --api-permissions 883ea226-0bf2-4a8f-9f9d-92c9162a727d=Role
```
`=Role` means this is an application permision and not a delegated permission.<br>
`00000003-0000-0000-c000-000000000000` is Microsoft Graph Api id.<br>
`883ea226-0bf2-4a8f-9f9d-92c9162a727d` is Site.Selected permission id.<br>
`883ea226-0bf2-4a8f-9f9d-92c9162a727d=Role` means that Site.Selected is an application permission.

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
 mgc login --scopes Sites.Manage.All --strategy InteractiveBrowser
```

### List sites
This step is a bit tricky.
For the reason described [here](https://stackoverflow.com/questions/75917021/sites-list-getting-empty-results-in-graph-api)
it is not possible to use simple `https://graph.microsoft.com/v1.0/sites` and i did not want to complicate this post with creating additional application flow authentication.
`mgc sites list --search "**"` won't work when using  mgc. It does work when using [Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer).

All the sharpoint sites will have a word "site" in it, so you can use the folowing command. Additionaly the output is limited only to id, webUrl and displayName.
``` powershell
mgc sites list --search "site" --select id,webUrl,displayName
```

One item of the array will look like this:
```json
  {
    "id": "your-company.sharepoint.com,guid-1,guid-2",
    "webUrl": "https://your-company-name.sharepoint.com/sites/YourSite",
    "displayName": "YourSite"
  },
```
The display name should match the name of your Sharpoint site.
> Note:  You need to use full 'your-company.sharepoint.com,guid-1,guid-2' as site-id. 

### List site permisions
``` powershell
 mgc sites permissions list --site-id <site-id>
```

### Create site permission
We don't need more than read.
This is orgin of [body](https://learn.microsoft.com/en-us/graph/api/site-post-permissions?view=graph-rest-1.0&tabs=http)

```powershell
mgc sites permissions create `
 --site-id <site-id> `
 --body '{"roles":["fullcontrol"],"grantedToIdentities":[{"application":{"id":"<client-id>","displayName":"<som-name>"}}]}'
```

### Confirm permission

```powershell
mgc sites permissions get `
 --site-id <site-id> `
 --permission-id <permission-id>
```

## Testing - delegated credentials flow
I call this section 'delegated credentials flow' because this is what is being used when loging in using `mgc login --scopes Sites.Manage.All --strategy InteractiveBrowser`. You delegate your own permissions to mgc.
This is not so exciting but let's play with some command here and then repeat the process with 'application credentials flow'.

### List drives in site
``` powershell
mgc sites drives list `
  --site-id <site-id> `
  --select id,name
```
The name should match document library name in Sharepoint.

### Get root folder of the drive
Files are not stored directly on drive object but on the root of the drive. Therfore we need to find it's id.

``` powershell
mgc drives root get `
  --drive-id  <drive-id>
```

### List drive items (folders/files) on the drive's root item
``` powershell
mgc drives items children list `
 --drive-id  <drive-id> `
 --drive-item-id <root-item-id> `
 --select id,name,folder
```

### List drive itmes (folder/files) in a drive item (folder)
You can run exactly the same command as before but just using chosen folder id.

``` powershell
mgc drives items children list `
 --drive-id  <drive-id> `
 --drive-item-id <your-nested-folder-item-id> `
 --select id,name,folder
```

### Download file from Sharepoint
Upload some file to Sharpoint manually. Then find it's id using `mgc drives items children list` command and download.

``` powershell
mgc drives items content get `
  --drive-id <drive-id> `
  --drive-item-id <drive-item-id> `
  --output-file c:\download-test-file.txt 
```

### Update existing file content
Create local file manualy. Then update content of the file on Sharepoint with content of the local file.
``` powershell
mgc drives items content put `
  --drive-id <drive-id> `
  --drive-item-id <drive-item-id> `
  --input-file c:\update-test-file.txt
 ```

### Logout
``` powershell
 mgc logout
 ```

## Testing - application credentials flow
Now the fun starts. Provided we have done everything correctly we should be able to perform all the tests but not using our credentials but application credentials.

### Login
There are various ways of doing it. For full list of strategies run `mgc login --help`. I have decided on simplicity here (at least in terms of authentication) therfore using enviroment variables and client secret.

#### Set enviromental variables:
The following is not the best method for setting environmental variables because variables will be stored in your powershell history. Choose your own method, just wanted to present what variables are required.
``` powershell
$env:AZURE_TENANT_ID = <tenant-id>
$env:AZURE_CLIENT_ID = <client-id>
$env:AZURE_CLIENT_SECRET = <client-secret>
```

#### Run login command:
``` powershell
mgc login --strategy Environment
```

### Test with application credentials flow
This where fun starts. If the setup is correct you will be able to do exactly the same test as with delegated credentials.


## Summary
As I mentioned in the begining this is no complete solution for CI/CD but it is the most difficult part. 
Using `mgc` is definitely not most convinient way of manipulating files on Sharepoint. You probably noticed that I haven't even tried things like creating file or directory. In the future will present how using [rclone](https://rclone.org/) can make things simpler.

