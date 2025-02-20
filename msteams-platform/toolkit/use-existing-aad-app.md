---
title: Use existing Azure AD app in TeamsFx project
author: surbhigupta
description:  In this module, learn how to use the existing Azure AD app or manually create Azure AD app for TeamsFx.
ms.author: surbhigupta
ms.localizationpriority: medium
ms.topic: overview
ms.date: 05/09/2023
---

# Use existing Azure AD app in TeamsFx project

This section provides information for using existing Azure Active Directory (Azure AD) app or manually create Azure AD app for TeamsFx project. Please follow the instruction and make sure all required info is properly set in your TeamsFx project.

## Create an Azure AD app

> [!NOTE]
> You can skip this part if you already have an Azure AD app. This step can be automated by the `aadApp/create` action.

1. Go to the [Azure Portal](https://portal.azure.com) and select **Azure Active Directory**.

1. Select **App Registrations** > **New registration** to create a new Azure AD app:
   * **Name**: The name of your configuration app.
   * **Supported account types**: Select **Account in this organizational directory only**.
   * Leave the **Redirect URL** field blank for now.
   * Select **Register**.

1. When the app is registered, you'll be taken to the app's **Overview** page. Copy the **Application (client) ID**, **Object ID**, and **Directory (tenant) ID**; it's needed later. Verify that the **Supported account types** is set to **My organization only**.

## Create client secret for Azure AD app (optional)

> [!NOTE]
> You can skip this part if your application doesn't require client secret. This step can be automated by the `aadApp/create` action.

1. Go to app's **Certificates & secrets** page, select **Client Secret** and select **New client secret**.
   * **Description**: The description of your client secret.
   * **Expires**: The expire time of your client secret.
   * Select **Add**.

1. When the client secret is added, press the copy button under the **Value** column to copy the **Client Secret**.

## Create access as user scope for Azure AD app (optional)

> [!NOTE]
> You can skip this part if your M365 account has permission to update the Azure AD app. We'll create the scope for you. This step can be automated by the `aadApp/update` action.

1. Go to app's **Expose an API** page, select **Add a scope** under **Scopes defined by this API**.
   * Select **Save and continue**.
   * **Scope name**: Fill in **access_as_user**.
   * **Who can consent?**: Choose **Admins and users**.
   * **Admin consent display name**: Fill in **Teams can access app’s web APIs**.
   * **Admin consent description**: Fill in **Allows Teams to call the app’s web APIs as the current user**.
   * **User consent display name**: Fill in **Teams can access app’s web APIs and make requests on your behalf**.
   * **User consent description**: Fill in **Enable Teams to call this app’s web APIs with the same rights that you have**.
   * **State**: Choose **Enabled**.
   * Select **Add scope**.

1. On the same page, select **Add a client application** under **Authorized client applications**.
   * **Client ID**: Fill in **1fec8e78-bce4-4aaf-ab1b-5451cc387264** which is Client Id for Teams on mobile and client.
   * **Authorized scopes**: Choose the existing **access_as_user** scope.
   * Select **Add application**.

1. Click again on **Add a client application**.
   * **Client ID**: Fill in **5e3ce6c0-2b1f-4285-8d4b-75ee78787346** which is Client Id for Teams on web.
   * **Authorized scopes**: Choose the existing **access_as_user** scope.
   * Select **Add application**.

2. Go to app's **Manifest** page, copy the **id** under **oauth2Permissions** as **Access As User Scope ID**.

## Get necessary info from existing Azure AD app

> [!NOTE]
> You may skip this part if you follow the instruction above to create an Azure AD app.

1. Go to the [Azure Portal](https://portal.azure.com) and select **Azure Active Directory**.

1. Select **App Registrations** and find your existing Azure AD app.

1. Go to app's **Overview** page, copy the **Application (client) ID**, **Object ID**, and **Directory (tenant) ID**; it's needed later. Verify that the **Supported account types** is set to **My organization only**.

1. Go to app's **Certificates & secrets** page, press the copy button under the **Value** column to copy the **Client Secret**.

    > [!NOTE]
    > If you can't copy the secret, please follow the [instruction](#create-client-secret-for-azure-ad-app-optional) to create a new client secret.
    
1. Go to apps **Expose an API** page. If you've already added **access_as_user** scope under **Scopes defined by this API** and pre-auth the two Teams Client Ids, go to app's **Manifest** page, copy the **id** under **oauth2Permissions** as **Access As User Scope ID**.

## Set necessary info in TeamsFx project

> [!NOTE]
> If you don't use `aadApp/create` action to create Azure AD application, you can add required environment variables with your preferred name without following the below steps.

1. Open `teamsapp.yml` and find the `aadApp/create` action.

1. Find the environment variable names that stores information for Azure AD app in the `writeToEnvironmentFile` property. Below are the default `writeToenvironmentFile` definition if you create projects using Teams Toolkit:

   ``` yaml
    writeToEnvironmentFile:
      clientId: AAD_APP_CLIENT_ID
      clientSecret: SECRET_AAD_APP_CLIENT_SECRET
      objectId: AAD_APP_OBJECT_ID
      tenantId: AAD_APP_TENANT_ID
      authority: AAD_APP_OAUTH_AUTHORITY
      authorityHost: AAD_APP_OAUTH_AUTHORITY_HOST
   ```

1. Add values for each environment variable from step 2.

   1. Add below environment variables and their values to `env\.env.{env}` file.

      ```yml
      AAD_APP_CLIENT_ID=<value of Azure AD application's client id (application id)> # example: 00000000-0000-0000-0000-000000000000
      AAD_APP_OBJECT_ID=<value of Azure AD application's object id> # example: 00000000-0000-0000-0000-000000000000
      AAD_APP_TENANT_ID=<value of Azure AD's Directory (tenant) id> # example: 00000000-0000-0000-0000-000000000000
      AAD_APP_OAUTH_AUTHORITY=<value of Azure AD's authority> # example: https://login.microsoftonline.com/<Directory (tenant) ID>
      AAD_APP_OAUTH_AUTHORITY_HOST=<host of Azure AD's authority> # example: https://login.microsoftonline.com
      AAD_APP_ACCESS_AS_USER_PERMISSION_ID=<id of access_as_user permission> # example: 00000000-0000-0000-0000-000000000000
      ```

   1. If your application requires an Azure AD app client secret, add below environment variable and its value to `env\.env.{env}.user` file.

      ```yml
      SECRET_AAD_APP_CLIENT_SECRET=<value of Azure AD application's client secret>
      ```

      > [!NOTE]
      > Remember to update the environment variable names in the examples if you use different names in `writeToEnvironmentFile`.

1. Open Teams Toolkit extension and select **Provision in the cloud**. Wait until your project is successfully provisioned.

## Upload Azure AD app manifest to Azure portal

If Teams Toolkit failed to update Azure AD app, there'll be an error that says:

```yml
Insufficient privileges to complete the operation.
```

If you see the above message, update Azure AD app permission and follow the instructions to update permission.

1. Find the AAD app manifest under `build/aad.manifest.{env}.json`.

1. Copy the content in the manifest file.

1. Go to the [Azure Portal](https://portal.azure.com) and select **Azure Active Directory**.

1. Select **App Registrations** and find your existing Azure AD app.

1. Go to app's **Manifest** page, paste the manifest content into the editor and select **Save** to save the changes.
