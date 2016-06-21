---
services: azure-storage
platforms: python
author: lmazuel
---

# Managing Azure Storage with Python

This sample explains how to manage your Storage account using the Azure Python SDK.

**On this page**

- [Run this sample](#run)
- [What is example.py doing?](#example)
    - [Check availability](#check-available)
    - [Create an account](#create-account)
    - [Get properties of an account](#get-properties)
    - [List storage accounts](#list-storage-accounts)
    - [Get the account keys](#get-keys)
    - [Regenerate keys](#regenerate-keys)
    - [Delete an account](#delete-account)
- [More information](#more-info)

<a name="run"></a>
## Run this sample

1. If you don't already have it, [install Python](https://www.python.org/downloads/).

2. We recommend to use a [virtual environnement](https://docs.python.org/3/tutorial/venv.html) to run this example, but it's not mandatory. You can initialize a virtualenv this way:

    ```
    pip install virtualenv
    virtualenv mytestenv
    cd mytestenv
    source bin/activate
    ```

3. Clone the repository.

    ```
    git clone https://github.com/Azure-Samples/storage-python-manage.git
    ```

4. Install the dependencies using pip.

    ```
    cd storage-python-manage
    pip install -r requirements.txt
    ```

5. Create an Azure service principal either through
[Azure CLI](http://azure.microsoft.com/documentation/articles/resource-group-authenticate-service-principal-cli/),
[PowerShell](http://azure.microsoft.com/documentation/articles/resource-group-authenticate-service-principal/)
or [the portal](http://azure.microsoft.com/documentation/articles/resource-group-create-service-principal-portal/).

6. Export these environment variables into your current shell. 

    ```
    export AZURE_TENANT_ID={your tenant id}
    export AZURE_CLIENT_ID={your client id}
    export AZURE_CLIENT_SECRET={your client secret}
    export AZURE_SUBSCRIPTION_ID={your subscription id}
    ```

7. Run the sample.

    ```
    python example.py
    ```

<a id="example"></a>
## What is example.py doing?

The sample walks you through several storage management operations.
It starts by setting up a ResourceManagementClient and StorageManagementClient objects using your subscription and credentials.

```python
import os
from azure.common.credentials import ServicePrincipalCredentials
from azure.mgmt.resource import ResourceManagementClient
from azure.mgmt.storage import StorageManagementClient
from azure.mgmt.storage.models import StorageAccountCreateParameters

subscription_id = os.environ.get(
    'AZURE_SUBSCRIPTION_ID',
    '11111111-1111-1111-1111-111111111111') # your Azure Subscription Id
credentials = ServicePrincipalCredentials(
    client_id=os.environ['AZURE_CLIENT_ID'],
    secret=os.environ['AZURE_CLIENT_SECRET'],
    tenant=os.environ['AZURE_TENANT_ID']
)
resource_client = ResourceManagementClient(credentials, subscription_id)
storage_client = StorageManagementClient(credentials, subscription_id)
```

It also sets up a ResourceGroup object (resource_group_params) to be used as a parameter in some of the API calls.

```python
resource_group_params = {'location':'westus'}
```

There are a couple of supporting functions (`print_item` and `print_properties`) that print a Azure object and it's properties.

<a name="check-available"></a>
### Check availability

Check the availability and/or the validity of a given string as a storage account.

```python
bad_account_name = 'invalid-or-used-name'
availability = storage_client.storage_accounts.check_name_availability(bad_account_name)
print('The account {} is available: {}'.format(bad_account_name, availability.name_available))
print('Reason: {}'.format(availability.reason))
print('Detailed message: {}'.format(availability.message))
```

<a name="create-account"></a>
### Create a storage account

```python
storage_async_operation = storage_client.storage_accounts.create(
    GROUP_NAME,
    STORAGE_ACCOUNT_NAME,
    StorageAccountCreateParameters(
        location='westus',
        account_type='Standard_LRS'
    )
)
storage_account = storage_async_operation.result()
```

<a name="get-properties"></a>
### Get properties of a storage account

```python
storage_account = storage_client.storage_accounts.get_properties(
    GROUP_NAME, STORAGE_ACCOUNT_NAME)
```

<a name="list-storage-accounts"></a>
### List storage accounts

```python
for item in storage_client.storage_accounts.list():
    print_item(item)
```

<a name="get-keys"></a>
### Get the account keys

```python
storage_keys = storage_client.storage_accounts.list_keys(GROUP_NAME, STORAGE_ACCOUNT_NAME)
storage_keys = {v.key_name: v.value for v in storage_keys.keys}
print('\tKey 1: {}'.format(storage_keys['key1']))
print('\tKey 2: {}'.format(storage_keys['key2']))
```

<a name="regenerate-keys"></a>
### Regenerate a given key

```python
storage_keys = storage_client.storage_accounts.regenerate_key(
    GROUP_NAME,
    STORAGE_ACCOUNT_NAME,
    'key1')
storage_keys = {v.key_name: v.value for v in storage_keys.keys}
print('\tNew key 1: {}'.format(storage_keys['key1']))
```

<a name="delete-account"></a>
### Delete a storage account

```python
storage_client.storage_accounts.delete(GROUP_NAME, STORAGE_ACCOUNT_NAME)
```

<a name="more-info"></a>
## More information

- [Azure SDK for Python](http://github.com/Azure/azure-sdk-for-python) 
