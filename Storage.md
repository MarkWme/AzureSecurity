## Access Control

Azure Storage Accounts offer three methods of controlling access to storage accounts and the data contained within those accounts.

It is possible to separate out access control for those activities relating to the management of the storage account - the *Management Plane* - and the data stored in an account - the *Data Plane*.

### Role Based Access Control (RBAC)
RBAC works with Azure Active Directory to control access to the storage account's management plane. Specific roles can be assigned to users or groups from your Azure Active Directory tenant which can then limit the operations those users or groups can perform.

Built-in Roles are predefined groups of operations that make it easy to get started quickly in the most common scenarios. However, you can build your own custom roles by combining one or more operations ...

<command line for custom role / operations list>

Custom Roles

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


### Access Keys
All Storage Accounts have two 512-bit access keys - effectively very long, complex passwords - that, if known, permit full access to all data within a storage account. These keys should be treated with the same level of care as any highly privileged account, such as administrative accounts for servers or Active Directory domains.

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


### Azure Key Vault

## Encryption

Encryption in transit / at rest

## Monitoring

## Alerting

## Analysis

## Recommended Actions

### Access Control Recommendation

- Define a key rotation policy with an interval that adequately provides a suitable level of risk exposure based on the sensitivity of your data
- Define an emergency key rotation process to be effected should a leak occur between key rotation intervals
- The number of people / services with access to storage account access keys should be kept to a minimum. Access keys should be treated as highly sensitive information.
- Using Azure Active Directory and Role Based Access Control, restrict access to list and regenerate storage account tokens
- Client / application access to data should be through Shared Access Signature tokens
- Use Key Vault as a secure store from which access keys and SAS tokens can be retrieved
- Consider implementing a SAS token service or front end proxy between the client / application and the storage account

Access keys in key vault - Yes
SAS keys in Key Vault or via a SAS token service [Review: Best method for generating SAS tokens]


Obtain keys from Key Vault

Use 2 keys. Apps could hold both keys. When access is denied on key 1, the app can switch to key 2 immediately to avoid any delay or interuption to service. When this happens, the app should be aware that key 1 has been regenerated and should begin a background task to obtain the new key. The application is then ready to switch to the new key 1 when key 2 is regenerated.

SAS proxy / auth service