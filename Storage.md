# Azure Storage Accounts - Best Practices for Secure Development and Operations

## Introduction

Azure Storage is a Platform as a Service offering, with access to data provided via a combination of URI's accessible over the Internet and secret access keys. There is sometimes concern, particularly in security concious or regulated industries, that because the endpoints are publicly accessible, that a loss or theft of the keys that secure access to the account could result in data loss.

Azure Storage Accounts provide built-in methods to allow you to control and monitor access to the account and the data contained within it. This document explains what those methods are and how other Azure services can be used alongside them to provide increased security, monitoring, alerting and auditing.

# Storage Accounts
Azure Storage Accounts provide a unique namespace to store and access your Azure Storage data objects. Storage Accounts give you access to Azure Storage services such as tables, queues, files, blobs and virtual machine disks.

## Access Control

Azure Storage Accounts offer three methods of controlling access to storage accounts and the data contained within those accounts. Role Based Access Control, Access Keys and Shared Access Signatures.

Access control for those activities relating to the management of the storage account - the **Management Plane** - and the data stored in an account - the **Data Plane**, can be separated out, allowing for separation of concerns to be implemented so that, for example, an administrator can manage the storage account but not have access to data.

### Role Based Access Control (RBAC)
RBAC works with Azure Active Directory to control access to the storage account's **management plane**. Specific roles can be assigned to users, groups or applications (service principals) from your Azure Active Directory tenant which can then limit the operations those users, groups or applications can perform.

Built-in Roles are predefined groups of operations that make it easy to get started quickly in the most common scenarios, such as granting read-only access or full control to a resource. You can view a list of all built-in roles using a PowerShell command like this:

```
Get-AzureRmRoleDefinition | FT Name
```

RBAC roles contain `Actions` and `NotActions`, effectively things that are allowed or not allowed to be performed by a role. You can view the `Actions` for a specific role like this:

```
(Get-AzureRmRoleDefinition -Name "Storage Account Contributor").Actions
```

You may discover that you cannot find a built-in role that meets your exact needs. In that case, Custom Roles allow you to define your own roles from a list of possible operations. To determine the operations that are available for a particular resource type, you can use a PowerShell command like the following:

```
Get-AzureRMProviderOperation Microsoft.Storage/* | FT Operation, OperationName
```

... which will list all of the operations available for Storage.  The result of that command looks like this:

Operation | Description
--------- | -----------
Microsoft.Storage/register/action | Registers the Storage Resource Provider
Microsoft.Storage/checknameavailability/read | Check Name Availability
Microsoft.Storage/storageAccounts/write | Create/Update Storage Account
Microsoft.Storage/storageAccounts/delete | Delete Storage Account
Microsoft.Storage/storageAccounts/listkeys/action | List Storage Account Keys
Microsoft.Storage/storageAccounts/regeneratekey/action | Regenerate Storage Account Keys
Microsoft.Storage/storageAccounts/read | List/Get Storage Account(s)
Microsoft.Storage/storageAccounts/listAccountSas/action | Returns Storage Account SAS Token
Microsoft.Storage/storageAccounts/listServiceSas/action | Returns Storage Service SAS Token
Microsoft.Storage/storageAccounts/services/diagnosticSettings/write | Create/Update Diagnostic Settings
Microsoft.Storage/skus/read | List Skus
Microsoft.Storage/usages/read | Get Subscription Usages
Microsoft.Storage/operations/read | Poll Asynchronous Operation
Microsoft.Storage/locations/deleteVirtualNetworkOrSubnets/action | Delete virtual network or subnets notifications

From this list you can see, for example, that operations exist for `listkeys` and `regeneratekey`. Creating a custom role using those operations will allow you to restrict a user, group or application to only being able to list access keys or regenerate new ones and perform no other management functions in the Storage Account.

### Access Keys
Whilst RBAC secures access to and operations on your Storage Account at the **management plane**, Access Keys and SAS tokens - which we'll discuss in the next section - control access to the data in your storage account. The **data plane**.

All Storage Accounts have two 512-bit access keys - effectively very long, complex passwords - that, if known, permit full access to all data within a storage account. These keys should be treated with the same level of care as any highly privileged account, such as administrative accounts for servers or Active Directory domains.

![Storage Account Access Keys](/images/storage-account-access-keys.png)

It is good practice to develop a key rotation policy, whereby the access keys are regenerated on a regular basis. This is similar to having a password policy, where users are required to change passwords on a regular basis. Regenerating a key immediately replaces it with a new key and has the effect of revoking access to all data until applications or services are updated to make use of the new key.

The keys can be regenerated through the Azure Portal, or using the PowerShell or Azure CLI command line tools, or via programmatic interfaces.

The frequency with which you want to regenerate your keys should be the result of risk analysis which would include factors such as the level of sensitivity of your data, the impact of data loss, the role and number of people who have access to the keys and the relative ease / complexity of updating keys in applications and services that access the data.

For a low risk scenario, you may wish to have a key rotation period of weeks or months, with policies covering exceptional scenarios, such as if a member of staff with access to keys leaves the organisation or if a key has been leaked. Conversely, where data is considered to be highly sensitive and the risk for data loss needs to be minimised, you could generate new keys on an hourly basis.

Management Plane apps vs Data Plane apps?

### Shared Access Signature
A shared access signature allows access to the data within a Storage Account, but with constraints to allow control over what can be accessed and how. Whereas access keys grant full access to a storage account, a shared access signature can:

- Limit access to any of blobs, containers, queues, files or tables.
- Restrict permissions to read, write, delete, list, add, create, update or process
- Specify permitted IP addresses that may access the data.
- Define start and end times during which a shared access signature is valid
- Force HTTPS to be used

A shared access signature takes the form of a string appended to the URL of the storage account. It contains the values which define the access restrictions as listed above and a signature which is generated from those values combined with one of the storage account access keys.

Because the shared access signature is signed using the storage account access key, regenerating the keys will cause all shared access signatures to become invalid. Therefore it is important to ensure that any key rotation policy gives due consideration to the impact this has on shared access signatures and the need for them to be regenerated too.

Service Level SAS

Account Level SAS

Stored Access Policy


# Azure Key Vault

## Access Control

As with Storage Accounts, Azure Key Vault has different methods available to secure access to the Management Plane, which affects the operation of the Key Vault itself and the Data Plane, which is access to the secrets and keys stored there.

### Role Based Access Control

To retrieve a current list of KeyVault operations that could be used as part of a custom role, use the following PowerShell command.

```
Get-AzureRMProviderOperation Microsoft.KeyVault/* | FT Operation, OperationName
```

Operation | Description
--------- | -----------
Microsoft.KeyVault/register/action|Register Subscription
Microsoft.KeyVault/unregister/action|Unregister Subscription
Microsoft.KeyVault/checkNameAvailability/read|Check Name Availability
Microsoft.KeyVault/vaults/read|View Key Vault
Microsoft.KeyVault/vaults/write|Update Key Vault
Microsoft.KeyVault/vaults/delete|Delete Key Vault
Microsoft.KeyVault/vaults/deploy/action|Use Vault for Azure Deployments
Microsoft.KeyVault/vaults/secrets/read|View Secret Properties
Microsoft.KeyVault/vaults/secrets/write|Update Secret
Microsoft.KeyVault/vaults/accessPolicies/write|Update Access Policy
Microsoft.KeyVault/operations/read|Available Key Vault Operations
Microsoft.KeyVault/deletedVaults/read|View Soft Deleted Vaults
Microsoft.KeyVault/locations/operationResults/read|Check Operation Result
Microsoft.KeyVault/locations/deletedVaults/read|View Soft Deleted Key Vault
Microsoft.KeyVault/locations/deletedVaults/purge/action|Purge Soft Deleted Key Vault

### Access Policies
Data Plane - Access Policies



## Encryption

Encryption in transit / at rest

## Monitoring

## Alerting

## Analysis

## Recommended Actions

### Summary

The following summarises the recommended actions, which are then described in more detail.

- Access keys should be treated as highly sensitive information. Restrict access to the storage account management plane and minimise the number of people or services with direct access to storage account access keys.
- Define a key rotation policy with an interval that meets your risk requirements.
- Define an emergency key rotation process to be effected should a leak occur between key rotation intervals.
- Provide access to storage account keys via Key Vault
- Access to data should only be through Shared Access Signature tokens.
- Authenticated applications request SAS tokens from Key Vault or a SAS token service

### 1. Restrict access to Storage Account Access Keys
Azure Storage Account access keys provide full access to everything within a storage account and therefore should be treated with the same level of care as any highly sensitive account credentials.

The built-in Owner, Contributor, Storage Account Contributor, Virtual Machine Contributor roles all grant access to view storage account access keys, therefore also allowing anyone assigned these roles access to all data.

A custom role can be defined to restrict access to storage account access keys by including the **ListKeys** and **RegenerateKey** actions and then assigning that role only to staff with the appropriate level of security clearance.

Do not use storage account access keys as a method for providing applications with access to data.


### 2. Key Rotation

A storage account access key can effectively be thought of as a long, complex password. As is normal practice in most organisations today, it is sensible to change passwords on a regular basis and the same is true for storage account access keys.

Each storage account has two access keys defined. This allows you to regenerate access keys alternately in order to minimise disruption to applications. For example, to regenerate `key1`, you would ensure that all applications that access the storage account switch over to using `key2`. You then regenerate `key1`. Then, at the next key rotation interval, you repeat the process by switching applications over to `key1`, then regenerating `key2`.

Regenerating storage account keys also has the effect of invalidating any SAS tokens signed with that key. So, regenerating the keys will also force all applications using SAS tokens to request a new token.

You need to determine a key rotation period that meets the security and risk requirements for your organisation and the sensitivity of the data in the storage account. It can sometimes take up to 10 minutes for a newly generated key to fully propagate, so beware of making the rotation period too short. In practice, the shortest realistic time period to regularly rotate keys is 1 hour.

SAS tokens can be configured with a start and end date and time which an application can easily read as the values are contained within the SAS token itself. Co-ordinate the key rotation period with SAS token generation to set expiration within the rotation period. An application will then know when the SAS token is going to expire and can request a new token before the next key rotation takes place.

You should also create an emergency key rotation policy that can force regeneration of new keys on demand should a security breach, leak of access keys, key employee termination or other event occur that necessitates a reset of all keys. Applications will need to be aware that this policy exists and that, despite what the expiration of a SAS token may be, it's possible it could become invalidated via the emergency process. Applications would need to handle permission denied errors and obtain new SAS tokens as required.

### 3. Automate Key Rotation with Azure Key Vault

To help reduce the need to provide access to the management plane of the Storage Account, you can secure the access keys by storing them in Azure Key Vault. Applications that need to obtain Storage Account access keys can retrieve them from Azure Key Vault, without needing access to the Storage Account itself. Using Key Vault also means that applications do not need to store the access keys in the application's settings or configuration files.

To implement a regular key rotation policy, you will need to write a simple application or automation script to regenerate keys on the required schedule. The tool you write to do this would use an Azure Active Directory service prinicipal to authenticate with both the Storage Account and Key Vault. That service principal can then be given only the `Set` permission in Key Vault, to allow it to write the new keys to the vault and the `ListKeys` and `RegenerateKey` actions as a custom RBAC role on the Storage Account.  The application can then regenerate the Storage Account access keys and write the new keys to Key Vault.

Note: Azure Key Vault has an integrated Storage Account access key rotation feature in preview. This feature will replace the need to create a separate application to perform regular key rotation. See [Azure Key Vault Storage Account Keys](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-ovw-storage-keys) for more information.

### 4. Use Shared Access Signature (SAS) Tokens

Applications that need access to data should be granted via a Shared Access Signature (SAS) token. SAS tokens can be configured to control access as follows.

- Restrict the SAS token to only the service required - Blob, File, Queue or Table.
- Restrict the SAS token to only the permission required - Read, Write, Delete, List, Add, Create, Update, Process
- Allow the HTTPS protocol only.
- If possible, restrict the IP address range to the known IP addresses for the application
- Set the start and expiry date and time. Co-ordinate this with your key rotation period to help avoid interruption to service.

Applications will need to obtain a new SAS token before the current one expires. Two ways to acheive this could be

1. Build a SAS Token Service. Applications then authenticate with the SAS Token service and are issued with a new SAS token. You can implement whatever business logic required to permit the issuing of new tokens and providing logging of requests.

2. Create an automation script or application that automatically generates new SAS tokens on a regular basis and synchronised with the key rotation policy. New SAS tokens are written to Key Vault. Applications then retrieve SAS tokens from Key Vault. Key Vault's diagnostic logs can be used to track requests to obtain tokens.

### Example Scenario

![Example Scenario](/images/examplescenario.png)

The above example is one possible way to implement a secure solution for accessing Storage Accounts.  The flows shown are as follows:

**Flow A - Key Rotation**
1. An Azure Function provides a key rotation service. This is executed on a timer trigger, for example, every hour on the hour.
2. The Function app uses a service principal to authenticate with Azure Active Directory.
3. The Function app regenerates the Storage Account key.
4. The Function app stores the updated Storage Account key in Key Vault.

**Flow B - SAS Token Service**
1. The SAS Token Service receives a request from an application for a new SAS token
2. The Function app uses a service principal to authenticate with Azure Active Directory.
3. The Function app retrieves the Storage Account key from Key Vault
4. The Function app generates a SAS token
5. The Function app sends the SAS token to the application

**Flow C - Application Access**
1. The application authenticates with Azure Active Directory
2. The application sends a request to the SAS Token Service
3. The application receives a new SAS token.
4. The applicaiton accesses the data using the SAS token.

