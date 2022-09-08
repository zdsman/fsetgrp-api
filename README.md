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
- `fset:grp:assignment:<classification>:<persona>:<scope>:<jurisdiction>:<function>` - FSET Assignment 
- `fset:user:user:<userId>` - FSET User

* * * 

## APIs
From here down are all the api calls for the service.

### Create Group
Creates a new group. When a new group is created, an FSETGrp Admin assignement is automatically added to the group for the Group owner. Additional FSETGrp Admin assignments may be added at a later time.

An error will be returned under the following conditions:
1. A group already exists with the same name
2. The caller does not have necessary permissions to create groups
3. An invalid group owner specified. Likely due to non-existent user or wrong Organizational Role for the implied FSETGrp Admin assignement.
4. The group name does not meet the nameing requirements

```
create_group(GroupName='string', GroupDescription='string', OwnerFri='string')
```
#### Parameters
- **GroupName** - Name of the group. This must consist of `[a-zA-Z][a-zA-Z0-9]{3,}`
- **GroupDescription** (optional) - Arbitrary description.
- **OwnerFri** - User FRI to be the group owner.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string',
    'GroupFri' : 'string',
    'OwnerFri' : 'string'
    }
}
```
#### Details
- **GroupName** (string) - Name of the group
- **GroupFri** (string) - Group FRI
- **OwnerFri** - (string) - User FRI of the group owner


### List Groups
List all groups.

An error will be returned under the following conditions:
1. The caller does not have necessary permissions to list groups

```
list_groups(Filters=[{'Name' : 'string', 'Values' : ['string']}])
```

#### Parameters
- **Filters** (optional) - One or more filters
  - `Owner` - The FRI of an owner to filter the results
  - `Assignments` - The FRI of one or more assignments to filter
  - `Users` - The FRI of one or more users

#### Response
```
{
    'Groups': {
        'GroupFri': 'string',
        'GroupName': 'string',
        'OwnerFri': 'string'
    }
}
```
#### Details
- **GroupName** - New name of the group
- **GroupFri** - FRI of the group
- **OwnerFri** - User FRI of the group owner

### Describe Group
Get information about a single FSET Group.

An error will be returned under the following conditions:
1. The caller does not have necessary permissions to describe groups
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
    'OwnerFri' : 'string',
    'AvailableAssignments': [
        'string',
    ],
    'Blueprints': [
        'string',
    ],
    'UserAssignments': [
        {
            'UserFri': 'string',
            'AssignmentFris' : ['string',]
        }
    ]
}
```
#### Details
- **GroupName** - Name of the group
- **GroupFri** - FRI of the group
- **OwnerFri** - FRI of the owner
- **AvailableAssignments** - List of available assignement FRIs associated to the group
- **Blueprints** - List of Blueprint FRIs associated to the group
- **UserAssignments** - List of all user assignments in the group
  - **UserFri** - FRI of the User
  - **AssignmentFris** - List of all assignment FRIs associated with the User

### Delete Group
Deletes an existing group. 

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have permission to delete the group. Only a Group Owner has permissions to delete a group.
3. Group has active assignements that require deleting first.

```
delete_group(GroupFri='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be deleted.

#### Response
```
{ }
```

### Update Group Attributes
Update group attributes. 

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have necessary permissions to update group attributes
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
    'GroupFri' : 'string'
    }
}
```
#### Details
- **GroupName** - New name of the group
- **GroupFri** - FRI of the group

### Update Group Owner
Updates the group owner. When the owner is changed, an FSETGrp Admin assignment is automatically added for the new owner. The previous owner will retain the FSETGrp Admin assignment until it is explicitly removed.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have necessary permissions to update the owner
3. An invalid owner is specified. Likely due to non-existent user or wrong Organizational role.

```
update_group_owner(GroupFri='string', OwnerFri='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **OwnerFri** - User FRI of the new group owner.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string',
    'GroupFri' : 'string',
    'OwnerFri' : 'string'
    }
}
```
#### Details
- **GroupName** (string) - Name of the group
- **GroupFri** (string) - Group FRI
- **OwnerFri** - (string) - User FRI of the group owner


### Add Assignments to a Group
Adds an assignment or set of assignments to the group. The assignments can then be used to make assignments to a user at a later time. If neither an assignment or template is specified, the operation silently does nothing.

If a template is specified, only the individual assignments contained in the template will be persisted on the group.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have necessary permissions to add assignments
3. Invalid assignment due to mixed Jurisdictions
4. Invalid assignment due to Organization Jurisdiction restriction. Assignment with Organization jurisdiction can only be in one group.
5. Invalid template
6. Unknown assignment
7. Assignment already exists


```
add_assignment(GroupFri='string', AssignmentFri='string', TemplateFri='string')
```
#### Parameters
- **GroupFri** - Id of the group to be updated.
- **AssignmentFri** (optional) - FRI of the rule being added to the group.
- **TemplateFri** (optional)- FRI of a template containing a set of assignments, all of which will be added. Duplicates will be silenty ignored.

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


### Remove Assignment from a Group
Remove an assignment from the group. Before an Assignment can be removed all users associated to the Assignement must be disassociated.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have necessary permissions to remove the assignment
3. Assignment is not in the group
4. Assignment can not be removed from the group. This mainly applies to the FSETGrp Admin assignments. There must always be a FSETGrp Admin assigned.
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

### Create Assignment Template
Create an Assignment Template that can later be used to add rules to a group in an efficient manner.

An error will be returned under the following conditions:
1. The template already exists
2. The caller does not have necessary permissions to create templates
3. Invalid assignment specified
4. Mixed Jurisdiction assignments not allowed

```
create_assignment_template(TemplateName='string', TemplateDescription ='string', Assignments=['string'])
```
#### Parameters
- **TemplateName** - Arbitrary name of the new template.
- **TemplateDescription** (optional) - Arbitrary description of the template
- **Assignments** - List of individual assignment FRIs.

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

### Delete Assignment Template
Delete an Assignment Template.

An error will be returned under the following conditions:
1. The template does not exist
2. The caller does not have necessary permissions to delete templates

```
delete_assignment_template(TemplateFri='string')
```
#### Parameters
- **TemplateFri** - FRI of the template to be deleted.

#### Response
```
{ }
```

### List Assignment Templates
List all Templates.

An error will be returned under the following conditions:
1. The caller does not have necessary permissions to list templates

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

