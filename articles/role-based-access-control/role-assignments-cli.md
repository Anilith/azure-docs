---
title: Manage access to Azure resources using RBAC and Azure CLI | Microsoft Docs
description: Learn how to manage access to Azure resources for users, groups, and applications using role-based access control (RBAC) and Azure CLI. This includes how to list access, grant access, and remove access.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman

ms.assetid: 3483ee01-8177-49e7-b337-4d5cb14f5e32
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/20/2019
ms.author: rolyon
ms.reviewer: bagovind
---
# Manage access to Azure resources using RBAC and Azure CLI

[Role-based access control (RBAC)](overview.md) is the way that you manage access to Azure resources. This article describes how you manage access for users, groups, and applications using RBAC and Azure CLI.

## Prerequisites

To manage access, you one of the following need:

* [Bash in Azure Cloud Shell](/azure/cloud-shell/overview)
* [Azure CLI](/cli/azure)

## List roles

To list all available role definitions, use [az role definition list](/cli/azure/role/definition#az-role-definition-list):

```azurecli
az role definition list
```

The following example lists the name and description of all available role definitions:

```azurecli
az role definition list --output json | jq '.[] | {"roleName":.roleName, "description":.description}'
```

```Output
{
  "roleName": "API Management Service Contributor",
  "description": "Can manage service and the APIs"
}
{
  "roleName": "API Management Service Operator Role",
  "description": "Can manage service but not the APIs"
}
{
  "roleName": "API Management Service Reader Role",
  "description": "Read-only access to service and APIs"
}

...
```

The following example lists all of the built-in role definitions:

```azurecli
az role definition list --custom-role-only false --output json | jq '.[] | {"roleName":.roleName, "description":.description, "roleType":.roleType}'
```

```Output
{
  "roleName": "API Management Service Contributor",
  "description": "Can manage service and the APIs",
  "roleType": "BuiltInRole"
}
{
  "roleName": "API Management Service Operator Role",
  "description": "Can manage service but not the APIs",
  "roleType": "BuiltInRole"
}
{
  "roleName": "API Management Service Reader Role",
  "description": "Read-only access to service and APIs",
  "roleType": "BuiltInRole"
}

...
```

## List a role definition

To list a role definition, use [az role definition list](/cli/azure/role/definition#az-role-definition-list):

```azurecli
az role definition list --name <role_name>
```

The following example lists the *Contributor* role definition:

```azurecli
az role definition list --name "Contributor"
```

```Output
[
  {
    "additionalProperties": {},
    "assignableScopes": [
      "/"
    ],
    "description": "Lets you manage everything except access to resources.",
    "id": "/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
    "name": "b24988ac-6180-42a0-ab88-20f7382dd24c",
    "permissions": [
      {
        "actions": [
          "*"
        ],
        "additionalProperties": {},
        "dataActions": [],
        "notActions": [
          "Microsoft.Authorization/*/Delete",
          "Microsoft.Authorization/*/Write",
          "Microsoft.Authorization/elevateAccess/Action"
        ],
        "notDataActions": []
      }
    ],
    "roleName": "Contributor",
    "roleType": "BuiltInRole",
    "type": "Microsoft.Authorization/roleDefinitions"
  }
]
```

### List actions of a role

The following example lists just the *actions* and *notActions* of the *Contributor* role:

```azurecli
az role definition list --name "Contributor" --output json | jq '.[] | {"actions":.permissions[0].actions, "notActions":.permissions[0].notActions}'
```

```Output
{
  "actions": [
    "*"
  ],
  "notActions": [
    "Microsoft.Authorization/*/Delete",
    "Microsoft.Authorization/*/Write",
    "Microsoft.Authorization/elevateAccess/Action"
  ]
}
```

The following example lists just the actions of the *Virtual Machine Contributor* role:

```azurecli
az role definition list --name "Virtual Machine Contributor" --output json | jq '.[] | .permissions[0].actions'
```

```Output
[
  "Microsoft.Authorization/*/read",
  "Microsoft.Compute/availabilitySets/*",
  "Microsoft.Compute/locations/*",
  "Microsoft.Compute/virtualMachines/*",
  "Microsoft.Compute/virtualMachineScaleSets/*",
  "Microsoft.Insights/alertRules/*",
  "Microsoft.Network/applicationGateways/backendAddressPools/join/action",
  "Microsoft.Network/loadBalancers/backendAddressPools/join/action",

  ...

  "Microsoft.Storage/storageAccounts/listKeys/action",
  "Microsoft.Storage/storageAccounts/read"
]
```

## List access

In RBAC, to list access, you list the role assignments.

### List role assignments for a user

To list the role assignments for a specific user, use [az role assignment list](/cli/azure/role/assignment#az-role-assignment-list):

```azurecli
az role assignment list --assignee <assignee>
```

By default, only assignments scoped to subscription will be displayed. To view assignments scoped by resource or group, use `--all`.

The following example lists the role assignments that are assigned directly to the *patlong@contoso.com* user:

```azurecli
az role assignment list --all --assignee patlong@contoso.com --output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

```Output
{
  "principalName": "patlong@contoso.com",
  "roleDefinitionName": "Backup Operator",
  "scope": "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/pharma-sales-projectforecast"
}
{
  "principalName": "patlong@contoso.com",
  "roleDefinitionName": "Virtual Machine Contributor",
  "scope": "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/pharma-sales-projectforecast"
}
```

### List role assignments for a resource group

To list the role assignments that exist for a resource group, use [az role assignment list](/cli/azure/role/assignment#az-role-assignment-list):

```azurecli
az role assignment list --resource-group <resource_group>
```

The following example lists the role assignments for the *pharma-sales-projectforecast* resource group:

```azurecli
az role assignment list --resource-group pharma-sales-projectforecast --output json | jq '.[] | {"roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

```Output
{
  "roleDefinitionName": "Backup Operator",
  "scope": "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/pharma-sales-projectforecast"
}
{
  "roleDefinitionName": "Virtual Machine Contributor",
  "scope": "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/pharma-sales-projectforecast"
}

...
```

## Grant access

In RBAC, to grant access, you create a role assignment.

### Create a role assignment for a user

To create a role assignment for a user at the resource group scope, use [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create):

```azurecli
az role assignment create --role <role> --assignee <assignee> --resource-group <resource_group>
```

The following example assigns the *Virtual Machine Contributor* role to *patlong@contoso.com* user at the *pharma-sales-projectforecast* resource group scope:

```azurecli
az role assignment create --role "Virtual Machine Contributor" --assignee patlong@contoso.com --resource-group pharma-sales-projectforecast
```

### Create a role assignment for a group

To create a role assignment for a group, use [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create):

```azurecli
az role assignment create --role <role> --assignee-object-id <assignee_object_id> --resource-group <resource_group> --scope </subscriptions/subscription_id>
```

The following example assigns the *Reader* role to the *Ann Mack Team* group with ID 22222222-2222-2222-2222-222222222222 at the subscription scope. To get the ID of the group, you can use [az ad group list](/cli/azure/ad/group#az-ad-group-list) or [az ad group show](/cli/azure/ad/group#az-ad-group-show).

```azurecli
az role assignment create --role Reader --assignee-object-id 22222222-2222-2222-2222-222222222222 --scope /subscriptions/11111111-1111-1111-1111-111111111111
```

The following example assigns the *Virtual Machine Contributor* role to the *Ann Mack Team* group with ID 22222222-2222-2222-2222-222222222222 at a resource scope for a virtual network named *pharma-sales-project-network*:

```azurecli
az role assignment create --role "Virtual Machine Contributor" --assignee-object-id 22222222-2222-2222-2222-222222222222 --scope /subscriptions/11111111-1111-1111-1111-111111111111/resourcegroups/pharma-sales-projectforecast/providers/Microsoft.Network/virtualNetworks/pharma-sales-project-network
```

### Create a role assignment for an application

To create a role for an application, use [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create):

```azurecli
az role assignment create --role <role> --assignee-object-id <assignee_object_id> --resource-group <resource_group> --scope </subscriptions/subscription_id>
```

The following example assigns the *Virtual Machine Contributor* role to an application with object ID 44444444-4444-4444-4444-444444444444 at the *pharma-sales-projectforecast* resource group scope. To get the object ID of the application, you can use [az ad app list](/cli/azure/ad/app#az-ad-app-list) or [az ad app show](/cli/azure/ad/app#az-ad-app-show).

```azurecli
az role assignment create --role "Virtual Machine Contributor" --assignee-object-id 44444444-4444-4444-4444-444444444444 --resource-group pharma-sales-projectforecast
```

## Remove access

In RBAC, to remove access, you remove a role assignment by using [az role assignment delete](/cli/azure/role/assignment#az-role-assignment-delete):

```azurecli
az role assignment delete --assignee <assignee> --role <role> --resource-group <resource_group>
```

The following example removes the *Virtual Machine Contributor* role assignment from the *patlong@contoso.com* user on the *pharma-sales-projectforecast* resource group:

```azurecli
az role assignment delete --assignee patlong@contoso.com --role "Virtual Machine Contributor" --resource-group pharma-sales-projectforecast
```

The following example removes the *Reader* role from the *Ann Mack Team* group with ID 22222222-2222-2222-2222-222222222222 at the subscription scope. To get the ID of the group, you can use [az ad group list](/cli/azure/ad/group#az-ad-group-list) or [az ad group show](/cli/azure/ad/group#az-ad-group-show).

```azurecli
az role assignment delete --assignee 22222222-2222-2222-2222-222222222222 --role "Reader" --scope /subscriptions/11111111-1111-1111-1111-111111111111
```

## Next steps

- [Tutorial: Create a custom role for Azure resources using Azure CLI](tutorial-custom-role-cli.md)
- [Use the Azure CLI to manage Azure resources and resource groups](../azure-resource-manager/cli-azure-resource-manager.md)