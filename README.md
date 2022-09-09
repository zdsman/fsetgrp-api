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
List all groups. If the caller has an assignment in a group, the results will always include those groups regardless of their assignments. Otherwise, an enabling assignment is necessary.

An error will be returned under the following conditions:
1. The caller does not have an enabling assignment to list groups AND has no assignments in any group.

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

#### Enabling Assignments
- `fset:grp:assignment:*:architect:*:o-*:observer`
- `fset:grp:assignment:*:architect:*:g-<groupId>:observer`
- `fset:grp:assignment:*:bi-admin:*:o-*:limitedObserver`
- `fset:grp:assignment:*:dba-manager:*:o-*:expertContributor`
- `fset:grp:assignment:*:dba:*:o-*:expertContributor`
- `fset:grp:assignment:*:db-manager:*:g-<groupId>:expertContributor`
- `fset:grp:assignment:*:data-engineer:*:g-<groupId>:expertContributor`
- `fset:grp:assignment:*:dl-admin:*:g-<groupId>:expertContributor`
- `fset:grp:assignment:*:data-scientist:*:g-<groupId>:observer`
- `fset:grp:assignment:*:developer:*:[s]g-<groupId>:*`
- `fset:grp:assignment:*:expense-manager:*:o-*>:observer`
- `fset:grp:assignment:*:experience-manager:*:g-<groupId>:*Observer`
- `fset:grp:assignment:*:fset-group-admin:*:g-<groupId>:*`
- `fset:grp:assignment:*:fset-group-admin:*:o-*>:*`
- `fset:grp:assignment:*:operator:*:[s]g-<groupId>:operations`
- `fset:grp:assignment:*:operator:*:[s]g-<groupId>:*Observer`
- `fset:grp:assignment:*:perf-test-admin:*:o-*:Observer`
- `fset:grp:assignment:*:platform-sys-admin:*:o-*:expertContributor`
- `fset:grp:assignment:*:product-account-owner:*:o-*:expertContributor` - look bogus jurisdiction
- `fset:grp:assignment:*:program-manager:*:o-*:*Observer`
- `fset:grp:assignment:*:program-manager:*:o-*:*Observer`
- `fset:grp:assignment:*:security-admin:*:o-*:limitedObserver`
- `fset:grp:assignment:*:site-reliability-engineer:*:o-*:operations`
- `fset:grp:assignment:*:user-account-admin:*:o-*:expertContributor`


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
2. The caller is not the owner of the group. Only a group owner can delete a group.
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
2. The caller does not have an enabling assignment to update the owner
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
2. The caller does not have an enabling assignment to add assignments
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
2. The caller does not have an enabling assignment to remove the assignment
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
2. The caller does not have an enabling assignment to create templates
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

### Add Source Repository to a Group
Adds a source repository (ie. github) to the group. A source repository can only be associated to one group at a time. The permissions to access the repository may take up to 10 minutes to replicate into the FSET Identity provider.

A repository with or without a Blueprint can be added to the group.

An error will be returned under the following conditions:
1. The repository does not exist
2. The caller does not have an enabling assignment to add repositories
3. Invalid repository
4. Repository allready assigned to a group.

```
add_source_repository(GroupFri='string', RepoUrl='string')
```
#### Parameters
- **GroupFri** - FRI of the group to be updated.
- **RepoUrl** - The URL of the repository

#### Response
```
{ }
```
#### Details
Nothing is response at this time.

### List Source Repositories in a Group
List source repositories (ie. github) in the group. 

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have an enabling assignment to list repositories

```
list_source_repositories(GroupFri='string')
```
#### Parameters
- **GroupFri** - FRI of the group for which repos will be listed.

#### Response
```
{ 
    'Repositories' : {
        'GroupName' : 'string',
        'GroupFri' : 'string',
        'UrlRepos' : [
            {
                'RepoUrl' : 'string',
                'BlueprintName' : 'string'
            }
        ],
        'ArnRepos' : [
            {
                'RepoArn' : 'string',
                'BlueprintName' : 'string'
            }
        ]
    }
}
```
#### Details
- **GroupFri** - FRI of the FSET Group
- **GroupName** - Name of the FSET Group
- **UrlRepos** - List of the source repositories that use a URL for the identifier
  - **RepoUrl** - The URL of the source repository
  - **BlueprintName** - The Blueprint name found in the repository. This will not be present if the repo doesn't contain a valid blueprint.yml file.
- **ArnRepos** - List of the source repositories that use an AWS ARN for the identifier
  - **RepoArn** - The AWS ARN of the source repository
  - **BlueprintName** - The Blueprint name found in the repository. This will not be present if the repo doesn't contain a valid blueprint.yml file.

### Remove a Source Repository from a Group
Remove a source repository (ie. github) from the group. The removal of permissions for the repository may take up to 10 minutes to be replicated into the FSET Identity provider.

An error will be returned under the following conditions:
1. The group does not exist
2. The repository isn't associated to the group
3. The caller does not have an enabling assignment to list repositories

```
remove_source_repository(GroupFri='string', RepoUrl='string')
```
#### Parameters
- **GroupFri** - FRI of the group for which the repo will be removed.
- **RepoUrl** - The URL of the repository

#### Response
```
{ }
```
#### Details
No response details

### Add User to a Group
Add a user to a group and associate the user to one or more assignments. The user may already be part of the group in which case only assignmens will be added. Duplicate assignments will be silently ignored.

An error will be returned under the following conditions:
1. Invalid group specified
2. The caller does not have an enabling assignment to add users
3. One or more specified assignments are not present in the group

```
add_user_to_group(GroupFri='string', UserFri='string', Assignments=['string'])
```
#### Parameters
- **GroupFri** - FRI of the group to which the user will be added
- **UserFri** - FRI of the user being added
- **Assignments** - List of individual assignment FRIs.

#### Response
```
{ 'UserAssignmentInfo' : {
    'UserFri' : 'string',
    'GroupFri' : 'string',
    'GroupName' : 'string',
    'Assignments' : [
        {'AssignmentFri' : 'string'}
        ]
    }
}
```
#### Details
- **UserFri** - FRI of the user
- **GroupFri** - FRI of the group
- **GroupName** - Group name
- **Assignments** - List of assignments associated to the user
  - **AssignmentFri** - Assignement FRI
