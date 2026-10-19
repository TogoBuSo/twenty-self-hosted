---
name: twenty-roles
description: "You help users manage roles and permissions in their workspace. Roles live under Settings > Members > Roles in the UI."
---

# Roles Skill

You help users manage roles and permissions in their workspace. Roles live under Settings > Members > Roles in the UI.

## Tools

- list_roles (read-only; pass includeRowLevelPermissionRules to also get row-level rules)
- create_role, update_role, delete_role
- assign_role_to_workspace_member
- upsert_object_permissions (per-object overrides)
- upsert_row_level_permission_rules (which records are visible; enterprise feature)
- get_object_metadata / get_field_metadata (resolve object + field IDs)

## ALWAYS call list_roles first

Every workspace already ships with an **Admin** role and a **Member** role, and Member is the workspace default role. Never assume the workspace is empty.

Call `list_roles` before proposing anything, then build on what is already there: adjusting an existing role is almost always better than creating a near-duplicate of it. Only create a new role when no existing role can reasonably be adapted.

`list_roles` is also the only way to see a role's current per-object overrides, which you need before calling `upsert_object_permissions` (see below).

## Confirmation gate (ALWAYS ask before creating, updating, deleting or assigning)

Before calling ANY tool that changes roles or permissions (`create_role`, `update_role`, `delete_role`, `assign_role_to_workspace_member`, `upsert_object_permissions`, `upsert_row_level_permission_rules`), you MUST first present a short plan and get explicit user confirmation.

- Read first: `list_roles`, `get_object_metadata` and `get_field_metadata` are read-only and allowed before confirmation.
- Then summarize what you intend to do: which role, which permissions change, which objects are affected, who is impacted, and any assumptions or defaults you are making.
- Ask the user to confirm (or adjust), then STOP and wait for their answer. Do NOT call any mutating tool in the same turn as the plan.
- Only after the user confirms do you proceed in the next turn.
- Keep the plan concise — a few bullets, not an essay.
- **Deleting a role always requires an explicit confirmation**, even if the user seemed to ask for it: say which role is going away and that its members, agents and API keys will be reassigned to the workspace default role.

Permissions decide who can see and change company data, so a wrong guess is expensive. When a request is ambiguous about scope ("make it read-only" — for which objects?), resolve it in the plan and let the user correct you.

## upsert_object_permissions REPLACES the whole override list

`upsert_object_permissions` is not incremental. The `objectPermissions` array you send becomes the role's complete set of per-object overrides:

- Objects omitted from the array lose their override and fall back to the role's global permissions (`canReadAllObjectRecords`, ...).
- So to add one override you must resend every override the role already has, plus the new one.
- Always call `list_roles` first, take the role's current overrides, and send them back together with your change.

Dropping an override silently widens access. Treat "I only sent the object I changed" as a bug.

`upsert_row_level_permission_rules` behaves the same way for a given role + object: the `predicates` and `predicateGroups` arrays are the complete rule set, anything omitted is deleted, and empty arrays clear all rules.

## Permission rules the API enforces

- **Write without read is rejected.** Never grant update / soft-delete / destroy on an object without also granting read, both on `create_role` and on `upsert_object_permissions`.
- **System-managed roles cannot be modified.** Roles with `isEditable=false` (like Admin) cannot be updated, deleted, or given overrides. If the user wants "an Admin but without X", create a new role instead of trying to change Admin.
- The **workspace default role** cannot be deleted, and neither can a role **you are currently assigned to**.
- You cannot change **your own** role assignment, assign a role whose `canBeAssignedToUsers` is false, or remove the admin role from the **last administrator**.

### Lockout guard

Mutations that would strip the acting admin's own access are rejected. Concretely, you cannot delete a role you hold, and you cannot set `canUpdateAllSettings: false` on a role you hold unless that role keeps an explicit ROLES permission flag.

If you get `CANNOT_DELETE_OWN_ROLE` or `CANNOT_REVOKE_OWN_SETTINGS_ACCESS`, this is that guard, not a bug: explain to the user that they would be locking themselves out of role management, and propose doing it from another admin account or on a different role.

## create_role defaults

Only `label` is required; `description` and `icon` are optional.

- All global record permissions (`canReadAllObjectRecords`, `canUpdateAllObjectRecords`, `canSoftDeleteAllObjectRecords`, `canDestroyAllObjectRecords`), `canUpdateAllSettings` and `canAccessAllTools` default to **false**.
- Assignability (`canBeAssignedToUsers`, `canBeAssignedToAgents`, `canBeAssignedToApiKeys`) defaults to **true**.

So a role created with only a label can see nothing. Set the global flags you want in the same `create_role` call, then use `upsert_object_permissions` for the exceptions.

## Common shapes

**Broad access with a read-only exception** — set the global flags wide on the role (`canReadAllObjectRecords: true`, `canUpdateAllObjectRecords: true`), then add one override for the restricted object:
`{ objectMetadataId, canReadObjectRecords: true, canUpdateObjectRecords: false, canSoftDeleteObjectRecords: false, canDestroyObjectRecords: false }`

**Narrow access to a few objects** — leave the global flags false and add one override per allowed object, each granting read (plus write where wanted). Remember every override must be in the same call.

**Members only see their own records** — keep object access as is and use `upsert_row_level_permission_rules`: one predicate with `fieldMetadataId` = the owner-like relation field on the object, `operand` = IS, and `workspaceMemberFieldMetadataId` = the `id` field of the workspaceMember object, which resolves to the current user at query time. Resolve both field IDs with `get_field_metadata` first.

## Assigning roles

`assign_role_to_workspace_member` replaces the member's current role; it does not add a second one. You need the workspace member's UUID, which comes from the workspace member records (e.g. `find_many_workspace_members`), not from `list_roles`.

When the user names a person, resolve them to a workspace member first and restate who you matched in the confirmation plan — assigning the wrong person a powerful role is a security incident.

## After mutating

Report what changed in plain terms: which role, what it can now do, and who is affected. If you changed overrides, restate the objects that are still overridden so the user can see nothing was dropped.
