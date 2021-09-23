 # Azure AD
The following KQL queries are useful for analysing Azure AD logs like Sign-in logs, Audit logs or Activity logs. You can simply use them in a Log Analytics Workspace log query or for further analytics and alerting, i.e. Azure Monitor Workbooks, Alerts or Azure Sentinel.

 ## 1. Get latest changes on Conditional Access policies
This query outputs the latest changes on Conditional Access policies, including the initiator, affected policy and old and new values. It uses the Azure AD Sign-in log to find the matching UserPrincipalName from the users ObjectId.

```kusto
AuditLogs
| where AdditionalDetails[0].value == "Conditional Access" 
| project ActivityDateTime, InitiatedBy.user.id, ActivityDisplayName, Type, TargetResources[0].displayName, TargetResources[0].modifiedProperties[0].oldValue, TargetResources[0].modifiedProperties[0].newValue
| extend UserId = tostring(InitiatedBy_user_id) 
| join kind=leftouter (SigninLogs 
    | project UserId, UserPrincipalName 
    | extend UserId = tostring(UserId))
    on UserId
| project-away InitiatedBy_user_id, UserId, UserId1
| project-rename Timestamp = ActivityDateTime, Action = ActivityDisplayName, InitiatedBy = UserPrincipalName, Source = Type, ConditionalAccessPolicy = TargetResources_0_displayName, OldValue = TargetResources_0_modifiedProperties_0_oldValue, NewValue = TargetResources_0_modifiedProperties_0_newValue
| project Timestamp, ConditionalAccessPolicy, Action, InitiatedBy, Source, OldValue, NewValue
| order by Timestamp desc
```

![Get latest changes on Conditional Access policies](https://raw.githubusercontent.com/gerbermarco/AzureKQL/main/AzureAD/includes/img/1.jpg)