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
create_group(GroupName='string', GroupDescription='string', 'OwnerId='string')
```
#### Parameters
- **GroupName** - Name of the group. This must consist of `[a-zA-Z][a-zA-Z0-9]{3,}`
- **GroupDescription** (optional) - Arbitrary description.
- **OwnerId** - User ID of the group owner.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string',
    'GroupId' : 'string',
    'OwnerId' : 'string'
    }
}
```
#### Details
- **GroupName** - New name of the group
- **GroupId** - ID of the group
- **OwnerId** - User ID of the group owner


### Delete Group
Deletes an existing group. 

An error will be returned under the following conditions:
1. The group does not exists
2. The caller does not have permission to delete the group. Only a Group Admin has permissions to delete a group.
3. Group has active assignements that require deleting first.

```
delete_group(GroupId='string')
```
#### Parameters
- **GroupId** - Id of the group to be deleted.

#### Response
```
{ }
```

### Update Group Attributes
Update group attributes. 

An error will be returned under the following conditions:
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
    'GroupName' : 'string',
    'GroupId' : 'string',
    'OwnerId' : 'string'
    }
}
```
#### Details
- **GroupName** - New name of the group
- **GroupId** - ID of the group

### Update Group Owner
Updates the group owner. When the owner is changed, an FSETGrp Admin assignment is automatically added for the new owner. The old owner will retain the FSETGrp Admin assignment until it is explicitly removed.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have necessary permissions to update the owner
3. An invalid owner is specified. Likely due to non-existent user or wrong Organizational role.

```
update_group_owner(GroupId='string', OwnerId='string')
```
#### Parameters
- **GroupId** - Id of the group to be updated.
- **OwnerId** - User ID of the new group owner.

#### Response
```
{ 'GroupInfo' : {
    'GroupName' : 'string',
    'GroupId' : 'string',
    'OwnerId' : 'string'
    }
}
```
#### Details
- **GroupName** - New name of the group
- **GroupId** - ID of the group
- **OwnerId** - User ID of the group owner after the update is committed


### Add Rule to a Group
Adds a rule or set of rules to the group. The rules can then be used to make assignments within the group at a later time. If neither a rule or template is specified, the operation silently does nothing.

If a template is specified, only the individual rules contained in the template will be persisted on the group.

An error will be returned under the following conditions:
1. The group does not exist
2. The caller does not have necessary permissions to update the owner
3. Invalid rule due to mixed Jurisdictions
4. Invalid template
5. Unknown rule
6. Rule already exists
7. Maximum number of rules exceeded

```
add_rule(GroupId='string', RuleId='string', TemplateName='string')
```
#### Parameters
- **GroupId** - Id of the group to be updated.
- **RuleId** (optional) - Id of the rule being added to the group.
- **TemplateName** (optional)- The name of a template containing a set of rules, all of which will be added. Duplicates will be silenty ignored.

#### Response
```
{ 'GroupRuleInfo' : {
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
4. Rule can not be removed from the group. This mainly applies to the FSETGrp Admin rule
5. Assignments exist using the rule

```
remove_rule(GroupId='string', RuleId='string')
```
#### Parameters
- **GroupId** - Id of the group to be updated.
- **RuleId** - Id of the rule being removed.

#### Response
```
{ 'GroupRuleInfo' : {
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

### Create Rule Template
Create a Rule Template that can later be used to add rules to a group in an efficient manner.

An error will be returned under the following conditions:
1. The template already exists
2. The caller does not have necessary permissions to create templates
3. Invalid rule specified
4. Mixed Jurisdiction not allowed

```
create_rule_template(TemplateName='string', TemplateDescription ='string', Rules=[RuleId])
```
#### Parameters
- **TemplateName** - Arbitrary name of the new template.
- **TemplateDescription** (optional) - Arbitrary description of the template
- **Rules** - List of individual rules.
- **RuleId** - ID of an individual rule.

#### Response
```
{ 'TemplateInfo' : {
    'TemplateName' : 'string',
    'Rules' : [
        {'RuleId' : 'string'}
        ]
    }
}
```
#### Details
- **TemplateName** - Name of the template
- **Rules** - List of rules in the template
- **RuleId** - ID of a rule

### Delete Rule Template
Delete a Rule Template.

An error will be returned under the following conditions:
1. The template does not exist
2. The caller does not have necessary permissions to delete templates

```
delete_rule_template(TemplateName='string')
```
#### Parameters
- **TemplateName** - Arbitrary name of the new template.

#### Response
```
{ }
```

### List Rule Templates
List all Rule Templates.

An error will be returned under the following conditions:
1. The caller does not have necessary permissions to list templates

```
list_rule_templates()
```
#### Response
```
{ 'Templates' : 
    [
        {
            'TemplateName' : 'string',
            'TemplateDescription' : 'string'
        }
    ]
}
```