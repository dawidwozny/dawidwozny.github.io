---
title: Unatended access to sharepoint
date: 2025-03-03 12:00:00 +0100
categories: [DevOps, Azure]
tags: [azure, devops, gcd]     # TAG names should always be lowercase
---
## Install powershell microsoft graph module

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
`mgc sites list --search "site"`

## list drives in site

`mgc sites drives list --site-id {siteIdFromPreviousStep} --select id,name` 


## get root folder of the drive

`mgc drives root get --drive-id {driveIdFromPreviousStep}`


## get folders on the drive (children of root item)
`mgc drives items children list --drive-id  {driveIdFromPreviousStep} --drive-item-id {rootItemIdFromPreviousStep}  --select id,name,folder`
