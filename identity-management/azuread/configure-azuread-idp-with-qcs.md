---
title: How To Configure Qlik Cloud Services to use Azure AD as an IdP.
description: Step-by-step instructions for implementing Azure AD identity provider connectivity in Qlik Cloud Services.
categories: Blogs
type: Blog
tags: ["Authenticate","AzureAD","OpenID","IdP"]
products: Qlik Cloud Services
---

TL;DR

Step-by-step instructions for implementing Azure AD identity provider connectivity in Qlik Sense Enterprise SaaS.
Configuring an App registration in Azure AD.
Configuring group support using MS Graph permissions.
Prereqs

Please make sure to have the following before starting this process:

Microsoft Azure account
Microsoft Azure Active Directory instance
Qlik Sense Enterprise SaaS tenant
Helpful vocab

Throughout this tutorial, some words will be used interchangeably.

Qlik Sense Enterprise SaaS: Qlik Sense hosted in Qlik’s public cloud
Microsoft Azure Active Directory: Azure AD
Tenant: Qlik Sense Enterprise SaaS tenant or instance
Instance: Microsoft Azure AD
OIDC: Open Id Connect
IdP: Identity Provider
Tutorial sections

This is a long tutorial with many clicks. It’s broken up into sections to make it easier to skip the desired set of instructions:

Considerations when using Azure AD with Qlik Sense Enterprise SaaS 

Introduction 

Configure Azure AD

Create the app registration
Create the client secret
Add claims to the token configuration
Add group claim
Collect Azure AD configuration information

Configure Qlik Sense Enterprise SaaS IdP

---------------------------------------------------------

Considerations when using Azure AD with Qlik Sense Enterprise SaaS

Qlik Sense Enterprise SaaS allows for customers to bring their own identity provider to provide authentication to the tenant using the Open ID Connect (OIDC) specification (https://openid.net/connect/)


Given that OIDC is a specification and not a standard, vendors (e.g. Microsoft) may implement the capability in ways that are outside of the core specification. In this case, Microsoft Azure AD OIDC configurations do not send standard OIDC claims like email_verified. Using the Azure AD configuration in Qlik Sense Enterprise SaaS includes an advanced option to set email_verified to true for all users that log into the tenant.


The Azure AD configuration in Qlik Sense Enterprise SaaS includes special logic for contacting Microsoft Graph API to obtain friendly group names. Whether those groups originate from an on-premises instance of Active Directory and sync to Azure AD through Azure AD Connect or from creation within Azure AD, the friendly group name will be returned from the Graph API and added to Qlik Sense Enterprise SaaS.
 



Introduction

This document will guide the reader through adding the necessary application configuration in Azure AD and Qlik Sense Enterprise SaaS identity provider configuration so that Qlik Sense Enterprise SaaS users may log into a tenant using their Azure AD credentials.

Read the Considerations section to understand the limitations of Azure AD group claims or want to learn more about the OIDC specification. Steps with pictures will provide the instructions above the picture.

Configure Azure AD

Create the app registration

1. Log into Microsoft Azure by going to https://portal.azure.com.

2. Click on the Azure Active Directory icon in the browser. The overview page for the active directorywill appear.



 

3. Click on the App registrations item in the menu to the left.





4. Click the New registration button at the top of the detail window. The application registration page appears.



 

5. Begin by adding a name in the Name section to identify the application. In this example, the name of the hostname of the tenant is entered along with the word OIDC.

 

6. The next section contains radio buttons for selecting the Supported account types. In this example, the default – Accounts in this organizational directory only – is selected.



7. The last section is for entering the redirect URI. From the dropdown list on the left select “web” and then enter the callback URL from the tenant. Enter the URI https://<tenant hostname>/login/callback.





8. Complete the registration by clicking the Register button at the bottom of the page.





9. Click on the Authentication menu item on the left side of the screen.





10. On the middle of the page, the reference to the callback URI appears. There is no additional configuration required on this page.



---------------------------------------------------------

Create the client secret

11. Click on the Certificates and secrets menu item on the left side of the screen.



 

12. In the center of the Certificates and secrets page, there is a section labeled Client secrets with a button labeled New client secret. Click the button.





13. In the dialog that appears, enter a description for the client secret and select an expiration time. Click the Add button after entering the information.



 

 

14. Once a client secret is added, it will appear in the Client secrets section of the page. Copy the value of the client secret and paste it somewhere safe. After saving the configuration the value will become hidden and unavailable.



---------------------------------------------------------

Add claims to the token configuration

15. Click on the Token configuration menu item on the left side of the screen.





16. The Optional claims window appears with two buttons. One for adding optional claims, and another for adding group claims. Click on the Add optional claim button.



 

17. For optional claims, select the ID token type, and then select the claims to include in the token that will be sent to the Qlik Sense Enterprise SaaS tenant. In this example, ctry, email, tenant_ctry, upn, and verified_primary_email are checked. None of these optional claims are required for the tenant identity provider to work properly, however, they are used later on in this tutorial.



18. Some optional claims may require adding OpenId Connect scopes from Microsoft Graph to the application configuration. Click the check mark to enable and click Add.





19. The claims will appear in the window.



---------------------------------------------------------

Add group claim

20. Click on the API permissions menu item on the left side of the screen.





21. Observe the configured permissions set during adding optional claims.





22. Click the Add a permission button and select the Microsoft Graph option in the Request API permissions box that appears. Click on the Microsoft Graph banner.





23. Click on Delegated permissions. The Select permission search and the OpenId permissions list appears.



Note: email, openid, and profile should be checked in this window when it appears. If they aren't, check them now.



24. In the Select permissions search, enter the word group. Expand the GroupMember option and select GroupMember.Read.All. This will grant users logging into Qlik Sense Enteprise SaaS through Azure AD to read the group memberships they are assigned.





25. After making the selection, click the Add permissions button.



26. The added permissions will appear in the list. However, the GroupMember.Read.All permission requires admin consent to work with the app registration. Click the Grant button and accept the message that appears.





--------------------------------------------------------- 

Collect Azure AD configuration information

27. Click on the Overview menu item to return to the main App registration screen for the new app. Copy the Application (client) ID unique identifier. This value is needed for the tenant’s idp configuration.



 

28. Click on the Endpoints button in the horizontal menu of the overview.



 

29. Copy the OpenID Connect metadata document endpoint URI. This is needed for the tenant’s IdP configuration.



---------------------------------------------------------

Configure Qlik Sense Enterprise SaaS IdP

29. With the configuration complete and required information in hand, open the tenant’s management console and click on the Identity provider menu item on the left side of the screen.



 

30. Click the Create new button on the upper right side of the main panel.





31. Select Interactive from the Type drop-down menu item, and select ADFS from the Provider drop-down menu item.





32. Scroll down to the Application credentials section of the configuration panel and enter the following information:

a. ADFS discovery URL: This is the endpoint URI copied from Azure AD.

b.Client ID: This is the application (client) id copied from Azure AD.

c. Client secret: This is the value copy and pasted to a safe location from the Certificates & secrets section from Azure AD.

d. The Realm is an optional value used if you want to enter what is commonly referred to as the Active Directory domain name.



 

33. Scroll down to the Claims mapping section of the configuration panel. There are five textboxes to confirm or alter.





33a. The sub field is the subject of the token sent from Azure AD. This is normally a unique identifier and will represent the UserID of the user in the tenant. In this example, the value “sub” is left and appid is removed. To use a different claim from the token, replace the default value with the name of the desired attribute value.





33b. The name field is the “friendly” name of the user to be displayed in the tenant. For Azure AD, change the attribute name from the default value to “name”.



33c. In this example, the groups, email, and client_id attributes are configured properly, therefore, they do not need to be altered.



Note: In this example, I had to change the email claim to upn to obtain the user's email address from Azure AD. Your results may vary.



 

34. Scroll down to the Advanced options and expand the menu. Slide the Email verified override option ON to ensure Azure AD validation works. Scope does not have to be supplied.



35. The Post logout redirect URI is not required for Azure AD because upon logging out the user will be sent to the Azure log out page.

36. Click the Save button at the bottom of the configuration to save the configuration. A message will appear confirming intent to create the identity provider. Click the Save button again to start the validation process.





37. The validation procedure begins by redirecting the person configuring the IdP to the login page for the IdP.



38. After successful authentication, Azure AD will confirm that permission should be granted for this user to the tenant. Click the Accept button.




39. If the validation fails, the validation procedure will return a window like the following.



40. If the validation succeeds, the validation procedure will return a mapped claims window. If the validation states it cannot map the user's email address, it is most likely because the email_verified switch has not been turned on. Go ahead and confirm, move through the remaining steps, and update the configuration from step 34. Re-run the validation to map the email.



 

 

41. After confirming the information is correct, the account used to validate the IdP may be elevated to a TenantAdmin role. It is strongly recommended to do make sure the box is checked before clicking continue.





42. The next to last screen in the configuration will ask to activate the IdP. By activating the Azure AD IdP in the tenant, any other identity providers configured in the tenant will be disabled.





43. ‘nuff said.





44. Please log out of the tenant and re-authenticate using the new identity provider connection. Once logged in, change the url in the address bar to point to https://<tenanthostname>/api/v1/diagnose-claims. This will return the JSON of the claims information Azure AD sent to the tenant. Here is a slightly redacted example.

 

45. Verify groups resolve properly by creating a space and adding members. You should see friendly group names to choose from.



 



 



 



--------------------------------

Recap

While not hard, configuring Azure AD to work with Qlik Sense Enterprise SaaS is not trivial. Most of the legwork to make this authentication scheme work is on the Azure side. However, it's important to note that without making some small tweaks to the IdP configuration in Qlik Sense you may receive a failure or two during the validation process.



Addendum

For many of you, adding Azure AD means you potentially have a bunch of clean up you need to do to remove legacy groups. Unfortunately, there is no way to do this in the UI but there is an API endpoint for deleting groups. Attached to this document is a guide for using script to delete groups from a Qlik Sense Enterprise SaaS tenant.