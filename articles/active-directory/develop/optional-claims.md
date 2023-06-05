---
title: Provide optional claims to your app
description: How to add optional claims to the SAML 2.0 and JSON Web Tokens (JWT) tokens issued by Microsoft identity platform.
author: davidmu1
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 03/28/2023
ms.author: davidmu
ms.custom: aaddev, curation-claims
ms.reviewer: ludwignick
---

# Provide optional claims to your app

You can use optional claims to:

- Select claims to include in tokens for your application.
- Change the behavior of certain claims that the Microsoft identity platform returns in tokens.
- Add and access custom claims for your application.

For the lists of standard claims, see the [access token](access-tokens.md) and [id_token](id-tokens.md) claims documentation.

While optional claims are supported in both v1.0 and v2.0 format tokens and SAML tokens, they provide most of their value when moving from v1.0 to v2.0. In the Microsoft identity platform, smaller token sizes are used to ensure optimal performance by clients. As a result, several claims formerly included in the access and ID tokens are no longer present in v2.0 tokens and must be asked for specifically on a per-application basis.

| Account Type | v1.0 tokens | v2.0 tokens |
|--------------|-------------|-------------|
| Personal Microsoft account | N/A | Supported |
| Azure AD account | Supported | Supported |

## v1.0 and v2.0 optional claims set

The set of optional claims available by default for applications to use are listed in the following table. You can use custom data in extension attributes and directory extensions to add optional claims for your application. When you add claims to the access token, the claims apply to access tokens requested *for* the application (a web API), not claims requested *by* the application. No matter how the client accesses your API, the right data is present in the access token that's used to authenticate against your API.

> [!NOTE]
>The majority of these claims can be included in JWTs for v1.0 and v2.0 tokens, but not SAML tokens, except where noted in the Token Type column. Consumer accounts support a subset of these claims, marked in the User Type column.  Many of the claims listed don't apply to consumer users (they have no tenant, so `tenant_ctry` has no value).

The following table lists the v1.0 and v2.0 optional claim set.

| Name | Description | Token Type | User Type | Notes |
|------|-------------|------------|-----------|-------|
| `acct` | Users account status in tenant | JWT, SAML | | If the user is a member of the tenant, the value is `0`. If they're a guest, the value is `1`. |
| `auth_time` | Time when the user last authenticated. | JWT | | |
| `ctry` | User's country/region | JWT |  | This claim is returned if it's present and the value of the field is a standard two-letter country/region code, such as FR, JP, SZ, and so on. |
| `email` | The reported email address for this user | JWT, SAML | MSA, Azure AD | This value is included if the user is a guest in the tenant. For managed users (the users inside the tenant), it must be requested through this optional claim or, on v2.0 only, with the OpenID scope. This value isn't guaranteed to be correct, and is mutable over time - never use it for authorization or to save data for a user. For more information, see [Secure applications and APIs by validating claims](claims-validation.md). If you require an addressable email address in your app, request this data from the user directly, using this claim as a suggestion or prefill in your UX. |
| `fwd` | IP address | JWT | | Adds the original IPv4 address of the requesting client (when inside a VNET). |
| `groups` | Optional formatting for group claims | JWT, SAML | | The `groups` claim is used with the GroupMembershipClaims setting in the [application manifest](reference-app-manifest.md), which must be set as well. |
| `idtyp` | Token type | JWT access tokens | Special: only in app-only access tokens | The value is `app` when the token is an app-only token. This claim is the most accurate way for an API to determine if a token is an app token or an app+user token. |
| `login_hint` | Login hint | JWT | MSA, Azure AD | An opaque, reliable login hint claim that's base64 encoded. Don't modify this value. This claim is the best value to use for the `login_hint` OAuth parameter in all flows to get SSO. It can be passed between applications to help them silently SSO as well - application A can sign in a user, read the `login_hint` claim, and then send the claim and the current tenant context to application B in the query string or fragment when the user selects on a link that takes them to application B. To avoid race conditions and reliability issues, the `login_hint` claim *doesn't* include the current tenant for the user, and defaults to the user's home tenant when used. In a guest scenario where the user is from another tenant, a tenant identifier must be provided in the sign-in request. and pass the same to apps you partner with. This claim is intended for use with your SDK's existing `login_hint` functionality, however that it exposed. |
| `sid` | Session ID, used for per-session user sign out | JWT | Personal and Azure AD accounts. | |
| `tenant_ctry` | Resource tenant's country/region | JWT | | Same as `ctry` except set at a tenant level by an admin. Must also be a standard two-letter value. |
| `tenant_region_scope` | Region of the resource tenant | JWT | | |
| `upn` | UserPrincipalName | JWT, SAML | | An identifier for the user that can be used with the `username_hint` parameter. Not a durable identifier for the user and shouldn't be used for authorization or to uniquely identity user information (for example, as a database key). Instead, use the user object ID (`oid`) as a database key. For more information, see [Secure applications and APIs by validating claims](claims-validation.md). Users signing in with an [alternate login ID](../authentication/howto-authentication-use-email-signin.md) shouldn't be shown their User Principal Name (UPN). Instead, use the following ID token claims for displaying sign-in state to the user: `preferred_username` or `unique_name` for v1 tokens and `preferred_username` for v2 tokens. Although this claim is automatically included, you can specify it as an optional claim to attach other properties to modify its behavior in the guest user case. You should use the `login_hint` claim for `login_hint` use - human-readable identifiers like UPN are unreliable. |
| `verified_primary_email` | Sourced from the user's PrimaryAuthoritativeEmail | JWT | | |
| `verified_secondary_email` | Sourced from the user's SecondaryAuthoritativeEmail   | JWT | | |
| `vnet` | VNET specifier information. | JWT | | |
| `xms_cc` | Client Capabilities | JWT | Azure AD | Indicates whether the client application that acquired the token is capable of handling claims challenges. Service applications (resource servers) can make use of this claim to authorize access to protected resources. This claim is commonly used in Conditional Access and Continuous Access Evaluation scenarios. The service application that issues the token controls the presence of the claim in it. This optional claim should be configured as part of the service app's registration. For more information, see [Claims challenges, claims requests and client capabilities](claims-challenge.md?tabs=dotnet). |
| `xms_pdl` | Preferred data location | JWT | | For Multi-Geo tenants, the preferred data location is the three-letter code showing the geographic region the user is in. For more information, see the [Azure AD Connect documentation about preferred data location](../hybrid/how-to-connect-sync-feature-preferreddatalocation.md). |
| `xms_pl` | User preferred language | JWT | | The user's preferred language, if set. Sourced from their home tenant, in guest access scenarios. Formatted LL-CC ("en-us"). |
| `xms_tpl` | Tenant preferred language| JWT | | The resource tenant's preferred language, if set. Formatted LL ("en"). |
| `ztdid` | Zero-touch Deployment ID | JWT | | The device identity used for `Windows AutoPilot`. |

> [!WARNING]
> Never use `email` or `upn` claim values to store or determine whether the user in an access token should have access to data. Mutable claim values like these can change over time, making them insecure and unreliable for authorization.

## v2.0-specific optional claims set

These claims are always included in v1.0 tokens, but not included in v2.0 tokens unless requested. These claims are only applicable for JWTs (ID tokens and access tokens).

| JWT Claim | Name | Description | Notes |
|-----------|------|-------------|-------|
| `ipaddr` | IP Address | The IP address the client logged in from. | |
| `onprem_sid` | On-premises Security Identifier | | |
| `pwd_exp` | Password Expiration Time | The number of seconds after the time in the `iat` claim at which the password expires. This claim is only included when the password is expiring soon (as defined by "notification days" in the password policy). | |
| `pwd_url` | Change Password URL | A URL that the user can visit to change their password. This claim is only included when the password is expiring soon (as defined by "notification days" in the password policy). | |
| `in_corp` | Inside Corporate Network | Signals if the client is logging in from the corporate network. If they're not, the claim isn't included. | Based off of the [trusted IPs](../authentication/howto-mfa-mfasettings.md#trusted-ips) settings in MFA. |
| `family_name` | Last Name | Provides the last name, surname, or family name of the user as defined in the user object. For example, `"family_name":"Miller"`. | Supported in MSA and Azure AD. Requires the `profile` scope. |
| `given_name` | First name | Provides the first or "given" name of the user, as set on the user object. For example, `"given_name": "Frank"`. | Supported in MSA and Azure AD. Requires the `profile` scope. |
| `upn` | User Principal Name | An identifier for the user that can be used with the `username_hint` parameter. Not a durable identifier for the user and shouldn't be used for authorization or to uniquely identity user information (for example, as a database key). For more information, see [Secure applications and APIs by validating claims](claims-validation.md). Instead, use the user object ID (`oid`) as a database key. Users signing in with an [alternate login ID](../authentication/howto-authentication-use-email-signin.md) shouldn't be shown their User Principal Name (UPN). Instead, use the following `preferred_username` claim for displaying sign-in state to the user. | Requires the `profile` scope. |

## v1.0-specific optional claims set

Some of the improvements of the v2 token format are available to apps that use the v1 token format, as they help improve security and reliability. These improvements only apply to JWTs, not SAML tokens.

| JWT Claim | Name | Description | Notes |
|-----------|------|-------------|-------|
|`aud` | Audience | Always present in JWTs, but in v1 access tokens it can be emitted in various ways - any appID URI, with or without a trailing slash, and the client ID of the resource. This randomization can be hard to code against when performing token validation. Use `additionalProperties` for this claim to ensure it's always set to the resource's client ID in v1 access tokens. | v1 JWT access tokens only|
|`preferred_username` | Preferred username | Provides the preferred username claim within v1 tokens. This claim makes it easier for apps to provide username hints and show human readable display names, regardless of their token type. It's recommended that you use this optional claim instead of using, `upn` or `unique_name`. | v1 ID tokens and access tokens |

### `additionalProperties` of optional claims

Some optional claims can be configured to change the way the claim is returned. These `additionalProperties` are mostly used to help migration of on-premises applications with different data expectations. For example, `include_externally_authenticated_upn_without_hash` helps with clients that can't handle hash marks (`#`) in the UPN.

| Property name | `additionalProperty` name | Description |
|---------------|--------------------------|-------------|
| `upn` | | Can be used for both SAML and JWT responses, and for v1.0 and v2.0 tokens. |
| | `include_externally_authenticated_upn` | Includes the guest UPN as stored in the resource tenant. For example, `foo_hometenant.com#EXT#@resourcetenant.com`. |
| | `include_externally_authenticated_upn_without_hash` | Same as listed previously, except that the hash marks (`#`) are replaced with underscores (`_`), for example `foo_hometenant.com_EXT_@resourcetenant.com`. |
| `aud` | | In v1 access tokens, this claim is used to change the format of the `aud` claim.  This claim has no effect in v2 tokens or either version's ID tokens, where the `aud` claim is always the client ID. Use this configuration to ensure that your API can more easily perform audience validation. Like all optional claims that affect the access token, the resource in the request must set this optional claim, since resources own the access token. |
| | `use_guid` | Emits the client ID of the resource (API) in GUID format as the `aud` claim always instead of it being runtime dependent. For example, if a resource sets this flag, and its client ID is `bb0a297b-6a42-4a55-ac40-09a501456577`, any app that requests an access token for that resource receives an access token with `aud` : `bb0a297b-6a42-4a55-ac40-09a501456577`. Without this claim set, an API could get tokens with an `aud` claim of `api://MyApi.com`, `api://MyApi.com/`, `api://myapi.com/AdditionalRegisteredField` or any other value set as an app ID URI for that API, and the client ID of the resource. |

#### `additionalProperties` example

```json
"optionalClaims": {
    "idToken": [
        {
            "name": "upn",
            "essential": false,
            "additionalProperties": [
                "include_externally_authenticated_upn"
            ]
        }
    ]
}
```

This `optionalClaims` object causes the ID token returned to the client to include a `upn` claim with the other home tenant and resource tenant information. The `upn` claim is only changed in the token if the user is a guest in the tenant (that uses a different IDP for authentication).

## Configure optional claims

> [!IMPORTANT]
> Access tokens are **always** generated using the manifest of the resource, not the client. In the request `...scope=https://graph.microsoft.com/user.read...`, the resource is the Microsoft Graph API.  The access token is created using the Microsoft Graph API manifest, not the client's manifest.  Changing the manifest for your application never causes tokens for the Microsoft Graph API to look different. To validate that your `accessToken` changes are in effect, request a token for your application, not another app.

You can configure optional claims for your application through the Azure portal or application manifest.

1. Go to the [Azure portal](https://portal.azure.com). 
1. Search for and select **Azure Active Directory**.
1. Under **Manage**, select **App registrations**.
1. Choose the application for which you want to configure optional claims based on your scenario and desired outcome.
1. Under **Manage**, select **Token configuration**.
   - The UI option **Token configuration** blade isn't available for apps registered in an Azure AD B2C tenant, which can be configured by modifying the application manifest. For more information, see  [Add claims and customize user input using custom policies in Azure Active Directory B2C](../../active-directory-b2c/configure-user-input.md)  

Configure claims using the manifest:

1. Select **Add optional claim**.
1. Select the token type you want to configure.
1. Select the optional claims to add.
1. Select **Add**.
1. Under **Manage**, select **Manifest**. A web-based manifest editor opens, allowing you to edit the manifest. Optionally, you can select **Download** and edit the manifest locally, and then use **Upload** to reapply it to your application.

    The following application manifest entry adds the `auth_time`, `ipaddr`, and `upn` optional claims to ID, access, and SAML tokens.

    ```json
    "optionalClaims": {
        "idToken": [
            {
                "name": "auth_time",
                "essential": false
            }
        ],
        "accessToken": [
            {
                "name": "ipaddr",
                "essential": false
            }
        ],
        "saml2Token": [
            {
                "name": "upn",
                "essential": false
            },
            {
                "name": "extension_ab603c56068041afb2f6832e2a17e237_skypeId",
                "source": "user",
                "essential": false
            }
        ]
    }
    ```

1. When finished, select **Save**. Now the specified optional claims are included in the tokens for your application.

The `optionalClaims` object declares the optional claims requested by an application. An application can configure optional claims that are returned in ID tokens, access tokens, and SAML 2 tokens. The application can configure a different set of optional claims to be returned in each token type.

| Name | Type | Description |
|------|------|-------------|
| `idToken` | Collection | The optional claims returned in the JWT ID token. |
| `accessToken` | Collection | The optional claims returned in the JWT access token. |
| `saml2Token` | Collection | The optional claims returned in the SAML token. |

If supported by a specific claim, you can also modify the behavior of the optional claim using the `additionalProperties` field.

| Name | Type | Description |
|------|------|-------------|
| `name` | Edm.String | The name of the optional claim. |
| `source` | Edm.String | The source (directory object) of the claim. There are predefined claims and user-defined claims from extension properties. If the source value is null, the claim is a predefined optional claim. If the source value is user, the value in the name property is the extension property from the user object. |
| `essential` | Edm.Boolean | If the value is true, the claim specified by the client is necessary to ensure a smooth authorization experience for the specific task requested by the end user. The default value is false. |
| `additionalProperties` | Collection (Edm.String) | Other properties of the claim. If a property exists in this collection, it modifies the behavior of the optional claim specified in the name property. |

## Configure directory extension optional claims

In addition to the standard optional claims set, you can also configure tokens to include Microsoft Graph extensions. For more information, see [Add custom data to resources using extensions](/graph/extensibility-overview).

Optional claims support extension attributes and directory extensions. This feature is useful for attaching more user information that your app can use. For example, other identifiers or important configuration options that the user has set. If your application manifest requests a custom extension and an MSA user logs in to your app, these extensions aren't returned.

### Directory extension formatting

When configuring directory extension optional claims using the application manifest, use the full name of the extension (in the format: `extension_<appid>_<attributename>`). The `<appid>` is the stripped version of the appId (or Client ID) of the application requesting the claim.

Within the JWT, these claims are emitted with the following name format:  `extn.<attributename>`. Within the SAML tokens, these claims are emitted with the following URI format: `http://schemas.microsoft.com/identity/claims/extn.<attributename>`

## Configure groups optional claims

This section covers the configuration options under optional claims for changing the group attributes used in group claims from the default group objectID to attributes synced from on-premises Windows Active Directory. You can configure groups optional claims for your application through the Azure portal or application manifest. Group optional claims are only emitted in the JWT for user principals. Service principals aren't included in group optional claims emitted in the JWT.

> [!IMPORTANT]
> The number of groups emitted in a token are limited to 150 for SAML assertions and 200 for JWT, including nested groups. For more information about group limits and important caveats for group claims from on-premises attributes, see [Configure group claims for applications with Azure AD](../hybrid/how-to-connect-fed-group-claims.md).

Complete the following steps to configure groups optional claims using the Azure portal:

1. Sign in to the [Azure portal](https://portal.azure.com).
1. After you've authenticated, choose your tenant by selecting it from the top-right corner of the page.
1. Search for and select **Azure Active Directory**.
1. Under **Manage**, select **App registrations**.
1. Select the application you want to configure optional claims for in the list.
1. Under **Manage**, select **Token configuration**.
1. Select **Add groups claim**.
1. Select the group types to return (**Security groups**, or **Directory roles**, **All groups**, and/or **Groups assigned to the application**): 
    - The **Groups assigned to the application** option includes only groups assigned to the application. The **Groups assigned to the application** option is recommended for large organizations due to the group number limit in token. To change the groups assigned to the application, select the application from the **Enterprise applications** list.  Select **Users and groups** and then **Add user/group**. Select the group(s) you want to add to the application from **Users and groups**.
    - The **All Groups** option includes **SecurityGroup**, **DirectoryRole**, and **DistributionList**, but not **Groups assigned to the application**. 
1. Optional: select the specific token type properties to modify the groups claim value to contain on premises group attributes or to change the claim type to a role.
1. Select **Save**.

Complete the following steps to configure groups optional claims through the application manifest:

1. Sign in to the [Azure portal](https://portal.azure.com).
1. After you've authenticated, choose your Azure AD tenant by selecting it from the top-right corner of the page.
1. Search for and select **Azure Active Directory**.
1. Select the application you want to configure optional claims for in the list.
1. Under **Manage**, select **Manifest**.
1. Add the following entry using the manifest editor:

   The valid values are:

   - "All" (this option includes SecurityGroup, DirectoryRole, and DistributionList)
   - "SecurityGroup"
   - "DirectoryRole"
   - "ApplicationGroup" (this option includes only groups that are assigned to the application)

   For example:

    ```json
    "groupMembershipClaims": "SecurityGroup"
    ```

   By default group object IDs are emitted in the group claim value. To modify the claim value to contain on premises group attributes, or to change the claim type to role, use the `optionalClaims` configuration as follows:

1. Set group name configuration optional claims.

   If you want groups in the token to contain the on premises group attributes in the optional claims section, specify which token type optional claim should be applied to. You also specify the name of optional claim requested and any other properties desired.  
   
   Multiple token types can be listed:

   - idToken for the OIDC ID token
   - accessToken for the OAuth access token
   - Saml2Token for SAML tokens.

   The Saml2Token type applies to both SAML1.1 and SAML2.0 format tokens.

   For each relevant token type, modify the groups claim to use the `optionalClaims` section in the manifest. The `optionalClaims` schema is as follows:

    ```json
    {
        "name": "groups",
        "source": null,
        "essential": false,
        "additionalProperties": []
    }
    ```

   | Optional claims schema | Value |
   |------------------------|-------|
   | `name` | Must be `groups` |
   | `source` | Not used. Omit or specify null. |
   | `essential` | Not used. Omit or specify false. |
   | `additionalProperties` | List of other properties. Valid options are `sam_account_name`, `dns_domain_and_sam_account_name`, `netbios_domain_and_sam_account_name`, `emit_as_roles` and `cloud_displayname`. |

   In `additionalProperties` only one of `sam_account_name`, `dns_domain_and_sam_account_name`, `netbios_domain_and_sam_account_name` are required.  If more than one is present, the first is used and any others ignored. You can also add `cloud_displayname` to emit display name of the cloud group. This option works only when `groupMembershipClaims` is set to `ApplicationGroup`.

   Some applications require group information about the user in the role claim. To change the claim type from a group claim to a role claim, add `emit_as_roles` to `additionalProperties`.  The group values are emitted in the role claim.

   If `emit_as_roles` is used, any application roles configured that the user is assigned aren't in the role claim.

The following examples show the manifest configuration for group claims:

Emit groups as group names in OAuth access tokens in `dnsDomainName\sAMAccountName` format.

```json
"optionalClaims": {
    "accessToken": [
        {
            "name": "groups",
            "additionalProperties": [
                "dns_domain_and_sam_account_name"
            ]
        }
    ]
}
```

Emit group names to be returned in `netbiosDomain\sAMAccountName` format as the roles claim in SAML and OIDC ID tokens.

```json
"optionalClaims": {
    "saml2Token": [
        {
            "name": "groups",
            "additionalProperties": [
                "netbios_domain_and_sam_account_name",
                "emit_as_roles"
            ]
        }
    ],
    "idToken": [
        {
            "name": "groups",
            "additionalProperties": [
                "netbios_domain_and_sam_account_name",
                "emit_as_roles"
            ]
        }
    ]
}
```

Emit group names in the format of `sam_account_name` for on-premises synced groups and `cloud_display` name for cloud groups in SAML and OIDC ID tokens for the groups assigned to the application.

```json
"groupMembershipClaims": "ApplicationGroup",
"optionalClaims": {
    "saml2Token": [
        {
            "name": "groups",
            "additionalProperties": [
                "sam_account_name",
                "cloud_displayname"
            ]
        }
    ],
    "idToken": [
        {
            "name": "groups",
            "additionalProperties": [
                "sam_account_name",
                "cloud_displayname"
            ]
        }
    ]
}
```

## Optional claims example

There are multiple options available for updating the properties on an application's identity configuration to enable and configure optional claims:

- You can use the Azure portal
- You can use the manifest.
- It's also possible to write an application that uses the [Microsoft Graph API](/graph/use-the-api) to update your application. The [OptionalClaims](/graph/api/resources/optionalclaims) type in the Microsoft Graph API reference guide can help you with configuring the optional claims.

In the following example, the Azure portal and manifest are used to add optional claims to the access, ID, and SAML tokens intended for your application. Different optional claims are added to each type of token that the application can receive:

- The ID tokens contain the UPN for federated users in the full form (`<upn>_<homedomain>#EXT#@<resourcedomain>`).
- The access tokens that other clients request for this application includes the `auth_time` claim.
- The SAML tokens contain the `skypeId` directory schema extension (in this example, the app ID for this app is `ab603c56068041afb2f6832e2a17e237`). The SAML token exposes the Skype ID as `extension_ab603c56068041afb2f6832e2a17e237_skypeId`.

Configure claims in the Azure portal:

1. Sign in to the [Azure portal](https://portal.azure.com).
1. After you've authenticated, choose your tenant by selecting it from the top-right corner of the page.
1. Search for and select **Azure Active Directory**.
1. Under **Manage**, select **App registrations**.
1. Find the application you want to configure optional claims for in the list and select it.
1. Under **Manage**, select **Token configuration**.
1. Select **Add optional claim**, select the **ID** token type, select **upn** from the list of claims, and then select **Add**.
1. Select **Add optional claim**, select the **Access** token type, select **auth_time** from the list of claims, then select **Add**.
1. From the Token Configuration overview screen, select the pencil icon next to **upn**, select the **Externally authenticated** toggle, and then select **Save**.
1. Select **Add optional claim**, select the **SAML** token type, select **extn.skypeID** from the list of claims (only applicable if you've created an Azure AD user object called skypeID), and then select **Add**.

Configure claims in the manifest:

1. Sign in to the [Azure portal](https://portal.azure.com).
1. After you've authenticated, choose your tenant by selecting it from the top-right corner of the page.
1. Search for and select **Azure Active Directory**.
1. Find the application you want to configure optional claims for in the list and select it.
1. Under **Manage**, select **Manifest** to open the inline manifest editor.
1. You can directly edit the manifest using this editor. The manifest follows the schema for the [Application entity](./reference-app-manifest.md), and automatically formats the manifest once saved. New elements are added to the `optionalClaims` property.

    ```json
    "optionalClaims": {
        "idToken": [
            {
                "name": "upn",
                "essential": false,
                "additionalProperties": [
                    "include_externally_authenticated_upn"
                ]
            }
        ],
        "accessToken": [
            {
                "name": "auth_time",
                "essential": false
            }
        ],
        "saml2Token": [
            {
                "name": "extension_ab603c56068041afb2f6832e2a17e237_skypeId",
                "source": "user",
                "essential": true
            }
        ]
    }
    ```

1. When you're finished updating the manifest, select **Save** to save the manifest.

## See also

- [Application manifest](reference-app-manifest.md)
- [ID tokens](id-tokens.md)
- [Access tokens](access-tokens.md)

## Next steps

- Learn more about the [tokens and claims](security-tokens.md) in the Microsoft identity platform.
