---
title: Switching from ADFS to AzureAD for AzureAD Groups in QCS
description: If your QCS tenant uses the ADFS option in identity provider settings  as a workaround for AzureAD, read this blog to learn how to migrate to the new  AzureAD option without duplicating user entries
categories: Blogs
type: Blog
tags: ["Authenticate","ADFS","AzureAD","OpenID","IdP"]
products: Qlik Cloud Services
---

# Switching from ADFS to AzureAD for AzureAD Groups in QCS

## Introduction

If you're a Qlik Enterprise SaaS early adopter, you managed to configure Azure
Active Directory as the identity provider for your tenant using the ADFS option
in the settings menu. While this enabled users to authenticate from Azure AD, it
didn't solve a key problem with this work around; native Azure group name
resolution.

The new Azure AD identity provider settings in QCS include proper group name
resolution. But you might be hesitant to switch the configuration from ADFS
because you're worried about the impact on existing access control set up in
your tenant. Will Space permissions work after the switch? Will users retain all
their personal content? Will the switch create duplicate users? This blog
resolves these concerns by showing you how to make the switch from ADFS to Azure
AD smoothly.

## Good news: The app registration in Azure AD remains valid

There's no need to reinvent the wheel and create a new app registration in Azure
to support the switch. However, there is a significant tweak to make if you want
to take advantage of Qlik's capabilities to resolve group names from Azure. Here
are the steps to prepare the existing app registration for reuse in QCS.

### Create a new client secret

In all likelihood, you misplaced - or should have lost - the client secret you
created when you set up the app registration the first time. It's easy enough to
generate a new secret for the app in the certificates and secrets section of the
config. Create a new client secret and make a copy of it for
use when you update QCS.

![certs-and-secrets](../../images/configure-azuread-idp-with-qcs/ad-config/certs-and-secrets.png)

### Remove groups optional claim from Token configuration

Say what now? Remove the groups claim? Yes, that's right, you don't need the
groups claim anymore to obtain a user's groups because the Azure AD setting in
QCS contacts the Microsoft Graph API to collect this information, resolving the
group names for Azure AD native groups and groups synchronized through
AD-Connect.

![remove-groups-claim](../../images/configure-azuread-idp-with-qcs/ad-config/remove-groups-claim.png)

### Add GroupMember.Read.All API permissions

The reason the optional groups claim is not needed is there is an API permission
allowing sharing the groups the user is a member of. When the permission
`GroupMember.Read.All` is added to the app registration, Qlik makes a request to
the MS Graph on behalf of the user to obtain the group names for which they are
a member.

![api-permission-list](../../images/configure-azuread-idp-with-qcs/ad-config/api-permission-list.png)

![ms-graph](../../images/configure-azuread-idp-with-qcs/ad-config/ms-graph.png)

![ms-graph-enabled-permissions](../../images/configure-azuread-idp-with-qcs/ad-config/ms-graph-enabled-permissions.png)

![groupmember-read-all](../../images/configure-azuread-idp-with-qcs/ad-config/groupmember-read-all.png)

> Before you continue! Make sure to grant admin consent for the new
> GroupMember.Read.All permission. If you miss this step, expect http 401 errors
> because of the request to Microsoft Graph.

![grant-consent](../../images/configure-azuread-idp-with-qcs/ad-config/grant-consent.png)

## Create a new interactive identity provider configuration in QCS

While Qlik Enterprise SaaS allows for only one active interactive identity
provider in a tenant, you can set up more to make it easier to switch from one
to another.

For converting from an ADFS configuration to an Azure AD setup it's
straightforward but there are a couple of tweaks you want to pay attention to
ensuring a smooth transition. Here are the steps:

### Copy IdP settings from the existing ADFS configuration

Take some screenshots of the ADFS set up, and record the discovery URL and
Client ID to notepad so you can reuse them in the new configuration.

![application-credentials](../../images/change-from-adfs-to-azuread-with-no-user-impact/application-credentials.png)

![claims-mapping](../../images/change-from-adfs-to-azuread-with-no-user-impact/claims-mapping.png)

### Create the Azure AD configuraiton

Create a new interactive identity provider selecting Azure AD from the provider
list.

![idp-type](../../images/change-from-adfs-to-azuread-with-no-user-impact/idp-type.png)

![provider-list](../../images/change-from-adfs-to-azuread-with-no-user-impact/provider-list.png)

Add the discovery URL for the app registration and input the Client ID and the
new client secret created earlier in the corresponding text boxes.

Fill out the claims mapping values from your ADFS settings screenshot.

Lastly, to make sure the mapping works properly expand Advanced options and
slide the email verified override switch to on.

![email_verified-switch](../../images/change-from-adfs-to-azuread-with-no-user-impact/email_verified-switch.png)

Create the configuration and save it.

After you authenticate to the new identity provider, the validation screen
appears. With the new email verified claim switch turned on, the email shows up
signifying the mapping process succeeded. Click through and activate the IdP.

![validation-screen](../../images/change-from-adfs-to-azuread-with-no-user-impact/validation-screen.png)

> Log out completely. This may require you to log out twice, once as the new
> validated user, and once as the user you logged in as to create the new
> configuration.

Log in through Azure and go to a space to add members and view friendly group names.

![group-list](../../images/change-from-adfs-to-azuread-with-no-user-impact/group-list.png)