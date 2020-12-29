---
title: Speed Up Your App Migration to SaaS with qlik-cli
description: Use qlik-cli to move Qlik Sense applications from Windows to the Cloud
categories: Blogs
type: Blog
tags: ["Migrate"]
products: Qlik Sense SaaS
---

# Speed Up Your App Migration to SaaS with qlik-cli

As interest in Qlik Sense SaaS offerings gain more interest from existing
customers, one of the first questions that comes up in conversations is
“How do I migrate my apps from Qlik Sense on Windows to the cloud?”
Of course there are multiple options, each with their pros and cons.

## Options

### Option 1: the manual export/import

You can export and import applications through the user interfaces of each
solution, but if you have a lot of apps that can be tedious and time consuming.

### Option 2: Qlik Sense multi-cloud

If you intend to carry-on with a hybrid deployment, using multi-cloud to
distribute apps for consumption to Qlik Sense SaaS can handle the task, but it
does require some post import per app before changes to the app are
synchronized.

### Option 3: qlik-cli

For large enterprises moving groups of apps to Qlik Sense Enterprise SaaS,
qlik-cli is a great option because you can automate many of the steps from
export to import, placement, and content publishing. Qlik-cli makes it easier
to integrate Qlik Devops – or Qlik Ops as I like to call it – into your
analytics application lifecycle.

In this blog I’m writing about option 3 because CLIs are all the rage these
days for their power, flexibility, and ease of use. Here is my experience
working with qlik-cli to migrate Qlik Sense apps.

Recently, I created a
[tutorial](https://qlik.dev/tutorials/migrate-apps-from-qlik-sense-on-windows-to-qlik-sense-saas)
taking readers step-by-step through the basics of migrating an app from Qlik
Sense on Windows to Qlik Sense SaaS using qlik-cli. Migrating Qlik Sense from
one place to another has never been easy, and I’m not claiming to have developed
the one-click method for doing it. But with a little upfront investment in
configuring access to each environment, exporting and importing can be as easy
as two commands:

```powershell
## Export the app from Qlik Sense on Windows
qlik qrs app export create $WinAppId --skipdata true --insecure --output-file $filepath/$filename

## Import the app to SaaS and return the new GUID for the app
$SaaSAppId=qlik app import --file $filepath/$filename --mode new --name $appname | ConvertFrom-Json |  Select-Object -Property resourceId
```

Of course there are many more things you want to do after importing the app,
like making sheets public, moving the app to a space, and changing the owner.
All of these steps are covered in the
[tutorial](https://qlik.dev/tutorials/migrate-apps-from-qlik-sense-on-windows-to-qlik-sense-saas)
. In general, here are the tips and tricks I picked up from my experience.

## Be prepared

Before embarking on your app migration journey, make sure to configure your Qlik
Sense environments so qlik-cli can connect to them. The easiest way to use
qlik-cli with Qlik Sense Windows is to [create a virtual proxy using
JWT](https://qlik.dev/tutorials/using-qlik-sense-on-windows-repository-api-qrs-with-qlik-cli)
. And the easiest way to use qlik-cli with Qlik Sense SaaS is an
[API Key](https://qlik.dev/tutorials/generate-your-first-api-key).

## Context is key

Once the infrastructure is ready to support incoming connections, use the
`qlik context` command to switch back and forth between environments while using
qlik-cli. The `qlik context use` command is a real timesaver compared to typing
out `--context` for each command you write.

```powershell
##qlik context create <contextName> --server <serverUrl> --server-type cloud --api-key <ApiKey>

qlik context create QSESaaS --server https://example.us.qlikcloud.com --server-type cloud --api-key eyJhbGciOi…
```

## Which Qlik commands work with which Qlik?

Both Qlik Sense on Windows and Qlik Sense SaaS share API endpoint names and
commands, but there are nuances between the QRS API available in Windows and the
Management APIs in SaaS. To distinguish commands in qlik-cli, use `qlik` for
SaaS commands, and `qlik qrs` for QRS commands. The only exceptions are the
`qlik app` building, analysis, and advanced commands which connect to apps
through the engine based upon the current context.

### Sample command to SaaS:

`qlik app space update $SaasAppId.resourceId --spaceId $space.id`

### Sample command to qrs:

`qlik qrs app ls --insecure --raw | ConvertFrom-Json | Select-Object -Property name, id`

## Unpublished (personal) app objects do not migrate

The challenges with personal content workflow are not new in the Qlik Ecosystem,
and unfortunately it remains an issue for Windows and SaaS Qlik Sense instances.
For now, if you need to have personal content migrated the easiest way forward
is to make the content (sheets, stories, bookmarks) public.

## Wrap up

This is just a sample of the power you can wield over Qlik Sense on Windows and
SaaS. Other Qlik Ops topics coming soon include manipulating app load script to
support section access in SaaS, and the concept of app unbuilding to commit apps
and app changes to git.

Do you have a Qlik Ops topic you want to learn more about? Leave a comment below
and we’ll add it to the roadmap. Thanks for reading!

## References

Tutorial related to this blog -
https://qlik.dev/tutorials/migrate-apps-from-qlik-sense-on-windows-to-qlik-sense-saas

Qlik-cli - https://qlik.dev/libraries-and-tools/qlik-cli

Create a virtual proxy for JWT in QSEoW -
https://qlik.dev/tutorials/using-qlik-sense-on-windows-repository-api-qrs-with-qlik-cli

Create an API Key - https://qlik.dev/tutorials/generate-your-first-api-key
