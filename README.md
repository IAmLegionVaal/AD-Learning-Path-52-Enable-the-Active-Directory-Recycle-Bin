# AD Learning Path 52 — Enable the Active Directory Recycle Bin

## Objective
Enable AD Recycle Bin at the forest level and prove that a deleted test object can be restored with its key attributes and group memberships intact.

## Important
Enabling AD Recycle Bin is a forest-wide, irreversible change. Perform this activity only in the isolated lab after verifying forest health and functional level.

## Prerequisites
- Healthy forest and replication
- Supported forest functional level
- Disposable test user and group membership
- Recovery evidence captured before deletion

## Setup
1. Record the current Recycle Bin feature state.
2. Create `Recycle-Test`, populate attributes, and add it to a disposable group.
3. Record its distinguished name, GUID, SID, attributes, and memberships.
4. Enable the Recycle Bin through Active Directory Administrative Center or PowerShell.
5. Allow the feature state to replicate to all domain controllers.
6. Delete the test user, locate it under Deleted Objects, and restore it.
7. Compare the restored object with the pre-delete baseline.

```powershell
Get-ADOptionalFeature 'Recycle Bin Feature' -Properties EnabledScopes
Enable-ADOptionalFeature 'Recycle Bin Feature' `
    -Scope ForestOrConfigurationSet -Target 'corp.lab'
Get-ADOptionalFeature 'Recycle Bin Feature' -Properties EnabledScopes
```

## Validation
```powershell
$Deleted = Get-ADObject -Filter 'isDeleted -eq $true -and Name -like "Recycle-Test*"' `
    -IncludeDeletedObjects
Restore-ADObject -Identity $Deleted.ObjectGUID
Get-ADUser Recycle-Test -Properties *
Get-ADPrincipalGroupMembership Recycle-Test
```

## Evidence
Store feature state before/after, forest level, object baseline, deletion timestamp, deleted-object metadata, restored attributes and membership, replication check, errors/remediation, and final pass/fail status under `evidence/`.

## Troubleshooting
- Feature already enabled: continue with the restore exercise.
- Deleted object not found: verify the filter, deleted-object lifetime, and target domain controller.
- Parent OU was deleted: restore the parent before child objects.

## Security notes
Recycle Bin improves object recovery but does not replace protected backups. Restored objects may regain access through recovered memberships, so validate them before enabling use.

## Cleanup
Disable or remove the disposable restored user after evidence capture. The Recycle Bin feature itself cannot be disabled.

## References
- Microsoft Learn: Active Directory Recycle Bin
- Microsoft Learn: `Restore-ADObject`

## Next activity
`AD-Learning-Path-53-Back-Up-Domain-Controller-System-State`
