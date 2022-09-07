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
    "message": <string>
    "uri" : <link>
}
```
### Details
- **statusCode** - Status code
- **message** - 
- **uri** - 

----
* * * 

## APIs
From here down are all the api calls for the service.

### Create Group
Creates a new group. An error will be returned under the following conditions:
1. A group already exists with the same name
2. The caller does not have necessary permissions to create groups
3. An invalid owner is specified
4. The group name does not meet the nameing requirements

```
create_group(GroupName='string', GroupDescription='string', 'OwnerId='string')
```
#### Parameters
- **GroupName** - Name of the group. This must consist of `[a-zA-Z]([a-zA-Z0-9])+`
- **GroupDescription** (optional) - Arbitrary description.
- **OwnerId** - Id of the group owner. The group owner.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string'
    'GroupId' : 'string'
    'OwnerId' : 'string'
    }
}
```
#### Details
- **GroupName** - New name of the group
- **GroupId** - ID of the group
- **OwnerId** - ID of the group owner


### Update Group Attributes
Update group attributes. An error will be returned under the following conditions:
1. A group does not exist
2. The caller does not have necessary permissions to update group attributes
4. The group name does not meet the nameing requirements

```
update_group_attributes(GroupId='string', GroupName='string',
        GroupDescription='string')
```
#### Parameters
- **GroupId** - Id of the group to be updated.
- **GroupName** (optional) - Name of the group. This must consist of `[a-zA-Z]([a-zA-Z0-9])+`
- **GroupDescription** (optional) - Arbitrary description.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string'
    'GroupId' : 'string'
    'OwnerId' : 'string'
    }
}
```
#### Details
- **GroupName** - New name of the group
- **GroupId** - ID of the group
- **OwnerId** - ID of the group owner

### Update Group Owner
Updates the group owner. An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have necessary permissions to update the owner
3. An invalid owner is specified

```
update_group_owner(GroupId='string', OwnerId='string')
```
#### Parameters
- **GroupId** - Id of the group to be updated.
- **OwnerId** - Id of the group owner.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string'
    'GroupId' : 'string'
    'OwnerId' : 'string'
    }
}
```
#### Details
- **GroupName** - New name of the group
- **GroupId** - ID of the group
- **OwnerId** - ID of the new group owner


### Add Rule to a Group
Adds a rule to the group. The rule can then be used to make assignments within the group.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have necessary permissions to update the owner
3. Invalid rule due to mixed Jurisdictions
4. Unknown rule
5. Maximum number of rules exceeded

```
add_rule(GroupId='string', RuleId='string')
```
#### Parameters
- **GroupId** - Id of the group to be updated.
- **RuleId** - Id of the rule being added to the group.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string'
    'GroupId' : 'string'
    'Rules' : [
        {'RuleId' : 'string'}
        ]
    }
}
```
#### Details
- **GroupName** - Name of the group
- **GroupId** - ID of the group
- **Rules** - List of rules assocated to the group
- **RuleId** - ID of a rule


### Remove Rule from a Group
Remove a rule from the group.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have necessary permissions to update the owner
3. Rule is not in the group
4. Assignments exist using the rule

```
remove_rule(GroupId='string', RuleId='string')
```
#### Parameters
- **GroupId** - Id of the group to be updated.
- **RuleId** - Id of the rule being removed.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string'
    'GroupId' : 'string'
    'Rules' : [
        {'RuleId' : 'string'}
        ]
    }
}
```
#### Details
- **GroupName** - Name of the group
- **GroupId** - ID of the group
- **Rules** - List of remaining rules assocated to the group
- **RuleId** - ID of a rule

