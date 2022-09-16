# fsetgrp-api
## Authentication and Authorization
### Authorization Header
```
"Authorization" : "Bearer \<token>"
```
---
## Common Error Handling
```
{
    "statusCode": <number>,
    "message": <string>,
    "uri" : <link>
}
```
### Details
- **statusCode** - Status code
- **message** - 
- **uri** - 

----
## FSET Resource Identifiers
The following FSET Resource Identifier (FRI) are used by this service:
- `fset:grp:group:<groupId>` -  FSET Group   
- `fset:grp:template:<templateId>` - FSET Template   
- `fset:grp:assignment:<user-type>:<role>:<environment>:<jurisdiction>:<level>` - FSET Assignment 

* * * 

## APIs
From here down are all the api calls for the service.

### Create Group
Creates a new group. When a new group is created, an FSETGrp Admin assignement is automatically added to the group for the caller. Additional FSETGrp Admin assignments may be added at a later time.

An error will be returned under the following conditions:
1. A group already exists with the same name
2. The caller does not have necessary permissions to create Organization groups
3. The group name does not meet the nameing requirements

```
create_group(GroupName='string', GroupDescription='string', Jurisdiction='string')
```
#### Parameters
- **GroupName** - Name of the group. This must consist of `[a-zA-Z][a-zA-Z0-9]{3,}`
- **GroupDescription** (optional) - Arbitrary description.
- **Jurisdiction** - Jurisdiction of group. Use `"Org"` for an organization. Don't specifiy for either group or self.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string',
    'GroupFri' : 'string',
    'IsOrgJurisdiction' : boolean
    }
}
```
#### Details
- **GroupName** (string) - Name of the group
- **GroupFri** (string) - Group FRI
- **IsOrgJurisdiction** (boolean) - `True` if the group is restricted to Organization jurisdiction assignments, `False` otherwise.

#### Enabling Assignments
- OR
  - `fset:grp:assignment:*:fset-group-admin:*:o-*:*` - This only applies when creating an "O" group.
  - `fset:grp:assignment:*:user-account-admin:*:o-*:*` - This only applies when creating a "G" group.


### List Groups
List all groups. If the caller is of user-type "Vendor", the only groups returned will be those who the caller is a member of.

An error will be returned under the following conditions:
No Error will be manifest. However, the response may be empty.

```
list_groups(Filters=[{'Name' : 'string', 'Values' : ['string']}])
```

#### Parameters
- **Filters** (optional) - One or more filters
  - `GroupName` - An expression allowing wildcard ('*') and single character ('?') anywhere in the string to filter the results
  - `Description` - An expression allowing wildcard ('*') and single character ('?') anywhere in the string to filter the results
  - `Assignments` - The FRI of one or more assignments to filter
  - `Users` - The ID of one or more users

#### Response
```
{
    'Groups': [
        {
            'GroupFri': 'string',
            'GroupName': 'string',
            'IsOrgJurisdiction' : boolean
        }
    ]
}
```
#### Details
- **GroupName** - New name of the group
- **GroupFri** - FRI of the group
- **IsOrgJurisdiction** (boolean) - `True` if the group is restricted to Organization jurisdiction assignments, `False` otherwise.

#### Enabling Assignments
No Assignments needed.


### Describe Group
Get information about a single FSET Group.

An error will be returned under the following conditions:
1. The caller's Organizational Role doesn't allow getting details for the group specified. Typical for vendors try to get info about a group they are not a member of.
2. The group does not exist

```
describe_group(GroupFri='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be described.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string',
    'GroupFri' : 'string',
    'IsOrgJurisdictions' : boolean,
    'AvailableAssignments': [
        'string',
    ],
    'Blueprints': [
        'string',
    ],
    'FullProductAccounts': [
        'string',
    ],
    'UserAssignments': [
        {
            'UserId': 'string',
            'AssignmentFris' : ['string',]
        }
    ]
}
```
#### Details
- **GroupName** - Name of the group
- **GroupFri** - FRI of the group
- **IsOrgJurisdiction** (boolean) - `True` if the group is restricted to Organization jurisdiction assignments, `False` otherwise.
- **AvailableAssignments** - List of available assignement FRIs associated to the group
- **Blueprints** - List of Blueprint FRIs associated to the group
- **FullProductAccounts** - List of all Full-Product accounts assoicated with the group
- **UserAssignments** - List of all user assignments in the group
  - **UserId** - ID of the User
  - **AssignmentFris** - List of all assignment FRIs associated with the User

#### Enabling Assignments
No enabling assignments needed.

### Delete Group
Deletes an existing group.

An error will be returned under the following conditions:
1. The group does not exist
2. Group has active assignements and/or associated resources that require deleting first.

```
delete_group(GroupFri='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be deleted.

#### Response
```
{ }
```

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:o-*:*`


### Update Group Attributes
Update group attributes. If neither the GroupName or GroupDescription is specified, no error os manifest.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have an enabling assignment to update group attributes
4. The group name does not meet the nameing requirements

```
update_group_attributes(GroupFri='string', GroupName='string',
        GroupDescription='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **GroupName** (optional) - Name of the group. This must consist of `[a-zA-Z]([a-zA-Z0-9])+`
- **GroupDescription** (optional) - Arbitrary description.

#### Response
```
{ 'GroupAttributeInfo' : {
    'GroupName' : 'string',
    'GroupDescription' : 'string',
    'GroupFri' : 'string'
    }
}
```
#### Details
- **GroupName** - New name of the group
- **Description** - The new description
- **GroupFri** - FRI of the group

#### Enabling Assignments
- OR
  - `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*`
  - `fset:grp:assignment:*:fset-group-admin:*:o-*:*`


### Add Assignments to a Group
Adds an assignment or set of assignments to the group. Once an assignement has been added, a member of the group can be granted the assignments. If the assignment has not been added to the group, an assignment can not be granted. 

If neither an assignment or template (group of assignments) is specified, the operation silently does nothing. If a duplicate assignment is present, the operation does not manifest an error. This API is idempotent.

If a template is specified, only the individual assignments contained in the template will be persisted on the group but not the template.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have an enabling assignment to add assignments
3. Invalid assignment due to mixed Jurisdictions
4. Invalid assignment due to Organization Jurisdiction restriction. Assignment with Organization jurisdiction can only be in one group.
5. Invalid template
6. Invalid assignment

```
add_assignment(GroupFri='string', AssignmentFri='string', TemplateName='string')
```
#### Parameters
- **GroupFri** - Id of the group to be updated.
- **AssignmentFri** (optional) - FRI of the rule being added to the group.
- **TemplateName** (optional)- Name of a template containing a set of assignments, all of which will be added. Duplicates will be silenty ignored.

#### Response
```
{ 'GroupRuleInfo' : {
    'GroupName' : 'string'
    'GroupFRI' : 'string'
    'Assignments' : [
        {'AssignmentFri' : 'string'}
    ]
}
```
#### Details
- **GroupName** - Name of the group
- **GroupFri** - FRI of the group
- **Assignments** - List of assignements assocated to the group
  - **AssignmentFri** - FRI of an assignment

#### Enabling Assignments
- OR
  - `fset:grp:assignment:*:fset-group-admin:*:o-*:*`
  - `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*`

### Remove Assignment from a Group
Remove an assignment from the group. Before an Assignment can be removed all users associated to the Assignement must be disassociated.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have an enabling assignment to remove the assignment
3. Assignment is not in the group
4. Assignment can not be removed from the group. This mainly applies to the FSETGrp Admin assignments. There must always be at least one FSETGrp Admin assigned.
5. Assignment is associated to one or more users.

```
remove_assignment(GroupFri='string', AssignmentFri='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **AssignmentFri** - FRI of an assignment

#### Response
```
{ 'GroupTemplateInfo' : {
    'GroupName' : 'string'
    'GroupFri' : 'string'
    'Assignments' : [
        {'AssignmentFri' : 'string'}
        ]
    }
}
```
#### Details
- **GroupName** - Name of the group
- **GroupFri** - FRI of the group
- **Assignments** - List of remaining assignments assocated to the group
- **AssignmentFri** - FRI of an assignment

#### Enabling Assignments
- OR
  - `fset:grp:assignment:*:fset-group-admin:*:o-*:*`
  - `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*`

### Create Assignment Template
Create an Assignment Template that can later be used to add rules to a group in an efficient manner.

An error will be returned under the following conditions:
1. The template already exists
2. The caller does not have an enabling assignment to create templates
3. The caller does not have an enabling assignment to read the group
4. Invalid assignment specified
5. Mixed Jurisdiction assignments not allowed

```
create_assignment_template(TemplateName='string', TemplateDescription ='string', Assignments=['string'], GroupFri='string')
```
#### Parameters
- **TemplateName** - Arbitrary name of the new template.
- **TemplateDescription** (optional) - Arbitrary description of the template
- **Assignments** (conditional) - List of individual assignment FRIs. A list of assignments OR a group FRI can be specified but not both.
- **GroupFri** (conditional)- A group FRI from which a template will be created.A list of assignments OR a group FRI can be specified but not both.

#### Response
```
{ 'TemplateInfo' : {
    'TemplateName' : 'string',
    'Assignments' : [
        {'AssignmentFri' : 'string'}
        ]
    }
}
```
#### Details
- **TemplateName** - Name of the template
- **Assignments** - List of assignments in the template
- **AssignmentFri** - FRI of a rule

#### Enabling Assignments
- AND
  - OR
    - `fset:grp:assignment:*:fset-group-admin:*:o-*:*`
    - `fset:grp:assignment:*:fset-group-admin:*:g-*:*`
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` Only if a group was specified.
  

### Delete Assignment Template
Delete an Assignment Template.

An error will be returned under the following conditions:
1. The template does not exist
2. The caller does not have an enabling assignment to delete templates

```
delete_assignment_template(TemplateFri='string')
```
#### Parameters
- **TemplateFri** - FRI of the template to be deleted.

#### Response
```
{ }
```
#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:*:*`
- 
### List Assignment Templates
List all Templates.

An error will be returned under the following conditions:
1. The caller does not have an enabling assignment to list templates

```
list_assignment_templates()
```
#### Response
```
{ 'Templates' : 
    [
        {
            'TemplateFri' : 'string',
            'TemplateName' : 'string',
            'TemplateDescription' : 'string'
        }
    ]
}
```
#### Details
- **TemplateFri** - FRI of the template
- **TemplateName** - Name of the template
- **TemplateDescription** - Description of the template

#### Enabling Assignments
- `fset:grp:assignment:*:fset-group-admin:*:*:*` 


### Add Blueprint to a Group
Adds a Blueprint to the group. Before a Blueprint can be added to a group, the Blueprint must exists in the Blueprint Metadata Service and must have a valid owner. In order to add a blueprint, the caller must be an owner of the Blueprint.

NOTE: Adding a Blueprint to the group does NOT imply that access to the repository. Access is achieve via assignments in the group. 

An error will be returned under the following conditions:
1. The Group does not exist
1. The Blueprint does not exist
2. The Caller is not the owner of the Blueprint in the BP Meta Service
3. The caller does not have the enabling assignment to add resources to the group
4. Blueprint already assigned to the group.

```
add_blueprint_to_group(GroupFri='string', BlueprintName='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **BlueprintName** - The name of the blueprint being added to the group

#### Response
```
{ 'ResourceInfo' : {
    'GroupName' : 'string',
    'GroupFri' : 'string',
    'BlueprintName' : 'string'
    }
}
```
#### Details
- **GroupName** (string) - Name of the group
- **GroupFri** (string) - Group FRI
- **BlueprintName** - (string) - Name of the Blueprint added

#### Enabling Assignments
- OR
  - `fset:grp:assignment:*:fset-group-admin:*:o-*:*` 
  - `fset:grp:assignment:*:fset-group-resource-admin:*:o-*:*` 
  - `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` 
  - `fset:grp:assignment:*:fset-group-resource-admin:*:g-<groupId>:*` 

### Add AWS Product Account to a Group
Adds an AWS product account to the group. A product account can be added to more than one FSETGrp. Only an owner of the Product account can add the account to the group. 

An error will be returned under the following conditions:
1. The Group does not exist
2. The Product account does not exist
3. The Caller does not have the enabling assignments to add product accounts to the group.
4. The Caller is not an owner of the product account.

```
add_product_account_to_group(GroupFri='string', ProductAccountId='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **ProductAccountId** - The 12-digit AWS AccountId.

#### Response
```
{ 'ResourceInfo' : {
    'GroupName' : 'string',
    'GroupFri' : 'string',
    'AccountId' : 'string'
    }
}
```
#### Details
- **GroupName** (string) - Name of the group
- **GroupFri** (string) - Group FRI
- **AccountId** - (string) - 12-digit AWS AccountId of the added account.

#### Required Assignments
- OR
  - `fset:grp:assignment:*:fset-group-admin:*:o-*:*` 
  - `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` 


### Remove a Blueprint from a Group
Remove a Blueprint from the group. In order to remove a blueprint, the caller must be an owner of the Blueprint.

If the Blueprint has an associated source repository, permissions to the repository may be removed, including read access (other assignments may still allow access). The removal of permissions to the repository may take up to 10 minutes to be replicated into the FSET Identity provider.

An error will be returned under the following conditions:
1. The group does not exist
2. The blueprint isn't associated to the group
3. The Caller is not an owner of the Blueprint in the BP Metadata Service
4. The caller does not have an enabling assignment to remove resources

```
remove_blueprint_from_group(GroupFri='string', BlueprintName='string')
```
#### Parameters
- **GroupFri** - FRI of the group for which the repo will be removed.
- **BlueprintName** - The name of the blueprint

#### Response
```
{ }
```
#### Details
No response details

#### Enabling Assignments
- OR
  - `fset:grp:assignment:*:fset-group-admin:*:o-*:*` - (FSETGrp Admin)
  - `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` 


### Remove an AWS Product Account from a Group
Removes an AWS product account from the group. A product account must be associated to at least one FSETGrp. Only a product account owner can remove the account from the group.

An error will be returned under the following conditions:
1. The Group does not exist
2. The Product is not associated to the group
3. Removal would orphan the account (not in any other FSETGrp)
4. The Caller does not have the enabling assignments to remove product accounts from the group.

```
remove_product_account_to_group(GroupFri='string', ProductAccountId='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **ProductAccountId** - The 12-digit AWS AccountId.

#### Response
```
{}
```
No result details

#### Enabling Assignments
- OR
  - `fset:grp:assignment:*:fset-group-admin:*:o-*:*`
  - `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*`

### Add User to a Group
Add a user to a group and granting the user to one or more assignments. The user may already be part of the group in which case only assignmens will be added. Assignments are idempotent.

Granting a user an assignment does not guarentee the permission are approved.

An error will be returned under the following conditions:
1. Invalid group specified
2. Invalid user specified
3. The caller does not have an enabling assignment to add users
4. One or more specified assignments are not present in the group

```
add_user_to_group(GroupFri='string', UserId='string', Assignments=['string'])
```
#### Parameters
- **GroupFri** - FRI of the group to which the user will be added
- **UserId** - Id of the user being added
- **Assignments** - List of individual assignment FRIs.

#### Response
```
{ 'UserAssignmentInfo' : {
    'UserId' : 'string',
    'GroupFri' : 'string',
    'GroupName' : 'string',
    'Assignments' : [
        {'AssignmentFri' : 'string'}
        ]
    }
}
```
#### Details
- **UserId** - Id of the user
- **GroupFri** - FRI of the group
- **GroupName** - Group name
- **Assignments** - List of assignments associated to the user
  - **AssignmentFri** - Assignement FRI

#### Enabling Assignments
- OR
  - `fset:grp:assignment:*:fset-group-admin:*:o-*:*` 
  - `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` 

### Grant User Assignment
Grant one or more assignments to a user.

The assignment provisioning is asyncrounous and may be denied. In this case, the API return successfully without any indication of the provisioning process.

An error will be returned under the following conditions:
1. Invalid group specified
2. Invalid user specified
3. One or more specified assignments are not present in the group

```
grant_assignment(GroupFri='string', UserId='string', Assignments=['string'])
```
#### Parameters
- **GroupFri** - FRI of the group to which the user will be added
- **UserId** - Id of the user being added
- **Assignments** - List of individual assignment FRIs.

#### Response
```
{ 'UserAssignmentInfo' : {
    'UserId' : 'string',
    'GroupFri' : 'string',
    'GroupName' : 'string',
    'Assignments' : [
        {'AssignmentFri' : 'string'}
        ]
    }
}
```
#### Details
- **UserId** - Id of the user
- **GroupFri** - FRI of the group
- **GroupName** - Group name
- **Assignments** - List of assignments associated to the user
  - **AssignmentFri** - Assignement FRI

#### Enabling Assignments
- OR
  - `fset:grp:assignment:*:fset-group-admin:*:o-*:*` 
  - `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*` 
