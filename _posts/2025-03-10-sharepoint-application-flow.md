---
title: Unatended access to sharepoint
date: 2025-03-03 12:00:00 +0100
categories: [DevOps, Azure]
tags: [azure, devops, gcd]     # TAG names should always be lowercase
---
## Use case
For me, ideal way of shipping the software to the customer is through containers and the second best is through package manager. However, this is not always the case. Sometimes you simply can't do that or you need to ship also source code along with some additional files. 

For long time, in that kind of scenarios, I was sharing special read-only folder to my sharepoint with my customer and uploading files manually. 

Out of few things I actually hate, doing manual work is my number one. There were some ways to automate that process but that would still require using the actuall login to my microsoft account. This approach in CD pipeline was no go. When it comes sharepoint, there were a ways of giving [Service Prinicipal](https://learn.microsoft.com/en-us/entra/identity-platform/app-objects-and-service-principals?tabs=browser)) permission to orgainisations sharepoint but there were not granural, meaning that you giving permission to full sharepoint in your organisation. Another no go.

Fortunatelly granural permission to sharepoint was something not only me requested and finally saw a daylight. After reading [What's the difference between Application and Degelated flows for accessing OneDrive via Graph API](https://marczak.io/posts/2023/03/azure-ad-graph-application-vs-delegated-flow/) and watching [Controlling app access on a specific SharePoint site collections is now available in Microsoft Graph](https://devblogs.microsoft.com/microsoft365dev/controlling-app-access-on-specific-sharepoint-site-collections/?WT.mc_id=AZ-MVP-5003556) decided to give it ago. I would like to share my jurney here. The process is not straight forward but worth it. On this ocasion I will share doing it manually but I might add steps to do it using [Azure Cli](https://learn.microsoft.com/en-us/cli/azure/) in the future.


## Prerequsits
* Business acount with sharepoint
* Rights to create App Registrations
* Rights to give permissions to sharepoint

## Steps
### Create App Registration [(Service Prinicipal)](https://learn.microsoft.com/en-us/entra/identity-platform/app-objects-and-service-principals?tabs=browser)
![app registrations](/assets/img/2025-03/app-registration.png)
*App Registrations*

### Add API Permissions [(Service Prinicipal)](https://learn.microsoft.com/en-us/entra/identity-platform/app-objects-and-service-principals?tabs=browser)
![app registrations](/assets/img/2025-03/api-permission.png)
*Api Permissions*

### Create sharepoint site
In this example I will give permissions to the whole sharpoint site therfore created a brand new site.

### Get site and drive information from Graph
This is where simple clicking ends. For all the quieries here I used Graph Explorer.

The following query will give you the list of all the sites in your oganisation. Pick the id of the one you want to give selective access.<br>
`GET https://graph.microsoft.com/v1.0/sites?search=*`

List site [permissions](https://learn.microsoft.com/en-us/graph/api/site-list-permissions?view=graph-rest-1.0&tabs=http):<br>
`GET https://graph.microsoft.com/v1.0/sites/{siteId}/permissions`

List site's [drives](https://learn.microsoft.com/en-us/graph/api/drive-list?view=graph-rest-1.0&tabs=http)
The following querry will give you the list of all drives in given site. Pick the id of the one you want to give selective access..
Drive is sharepoint Document Library. I have created a seprate one for the unatended access but still have normal access to it.<br>
`GET https://graph.microsoft.com/v1.0/sites/{siteId}/drives`

List folders in the drive. Just to ch:
`GET https://graph.microsoft.com/v1.0/drives/{drive-id}/root/children`

Pick the folder and check it's permisions. By default they should be empty

`GET https://graph.microsoft.com/v1.0/sites/{siteId}/drives/{driveId}/items/{itemId}/permissions`

### Give read/write permissions to your service principal
[Selective Permissions](https://learn.microsoft.com/en-us/graph/permissions-selected-overview?tabs=http)


`POST https://graph.microsoft.com/v1.0/sites/{siteId}/drives/{driveId}/items/{itemId}/permissions`
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

