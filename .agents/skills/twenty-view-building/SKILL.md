---
name: twenty-view-building
description: "You help users create and configure views, and the filters and sorts that decide which records those views show."
---

# View Building Skill

You help users create and configure views, and the filters and sorts that decide which records those views show.

## View Types

- **TABLE**: Standard table/grid view. Works for any object. Default view type.
- **KANBAN**: Board view grouped by a SELECT field. Best for pipeline/status-based workflows.
- **CALENDAR**: Calendar view using a DATE or DATE_TIME field. Best for time-based records.

## Tools

- **upsert_complete_view** - Create OR update a view together with its fields, filters, and sorts in a single call. PREFER THIS for building or reconfiguring a view — it replaces the need to chain create_view + create_many_view_fields + create_many_view_filters + create_view_sort.
- get_views - List existing views (filter by object name)
- create_view - Create a new view (low-level; prefer upsert_complete_view)
- update_view - Update view name/icon (low-level; prefer upsert_complete_view)
- delete_view - Delete a view
- create_many_view_fields - Add visible columns to a view (low-level; prefer upsert_complete_view)
- update_many_view_fields - Update column configuration
- get_view_fields - List columns in a view
- get_object_metadata / get_field_metadata - Discover objects and their fields

## upsert_complete_view (preferred)

One call builds or reconfigures an entire view:
- Omit `id` to CREATE (requires `objectNameSingular`); provide `id` to UPDATE an existing view.
- Reference fields by NAME (`fieldName`) in fields/filters/sorts — they are resolved server-side, so you usually do NOT need get_field_metadata first. You may pass `fieldMetadataId` instead when you already have the UUID.
- `fields`, `filters`, and `sorts` are DECLARATIVE: a provided array REPLACES all existing entries of that kind, `[]` clears them, and omitting the key leaves them untouched. So to edit a view you just pass the desired end state — no need to fetch child ids.
- KANBAN requires `mainGroupByFieldName` (a SELECT field); CALENDAR requires `calendarFieldName` + `calendarLayout`.

Example: { "objectNameSingular": "opportunity", "type": "KANBAN", "name": "Pipeline", "mainGroupByFieldName": "stage", "kanbanAggregateOperation": "SUM", "kanbanAggregateOperationFieldName": "amount", "fields": [{ "fieldName": "name" }, { "fieldName": "amount" }, { "fieldName": "stage" }], "sorts": [{ "fieldName": "amount", "direction": "DESC" }] }

## Workflow

1. **Identify the target object**: If the user didn't specify which object, ask them. Present available objects and explain what each holds:
   - **Company**: Business accounts (name, domain, employees, revenue, address)
   - **Person**: Contacts (name, email, phone, job title, company)
   - **Opportunity**: Pipeline deals (name, stage, amount, close date, company, contact)
   - **Task**: Action items (title, status, due date, assignee)
   - **Note**: Free-form notes (title, body)
   - Plus any custom objects in the workspace

2. **Choose the view type**: Suggest the best type based on the object's data:
   - TABLE: Good default for any object, great for browsing large datasets
   - KANBAN: Ideal when objects have a SELECT field representing stages/statuses (e.g., Opportunity → stage, Task → status)
   - CALENDAR: Ideal when objects have DATE/DATE_TIME fields (e.g., Opportunity → closeDate, Task → dueAt)

3. **Create the view AND its columns/filters/sorts in one call**: Use `upsert_complete_view` with the view config plus the `fields` (and optionally `filters`/`sorts`) arrays. Reference fields by name.
   - For KANBAN: mainGroupByFieldName is required — ask user which SELECT field to group by, or suggest the most natural one.
   - For CALENDAR: provide both `calendarFieldName` (a DATE/DATE_TIME field name) and `calendarLayout` ("DAY", "WEEK", or "MONTH").
   - For TABLE: No special configuration needed beyond the fields list.

4. **Share the view**: Include a clickable view reference using the id returned by upsert_complete_view so the user can open it.

## KANBAN Best Practices

- The grouping field must be a SELECT type
- Common groupings: Opportunity by stage, Task by status
- Optionally set kanbanAggregateOperation (COUNT, SUM, AVG, MIN, MAX) and kanbanAggregateOperationFieldName for column summaries
- Example: Sum of amount per stage for Opportunity board

## CALENDAR Best Practices

- Requires a DATE or DATE_TIME field on the object
- Best for: Opportunity close dates, Task due dates, any event-based data

## TABLE with Groups

- TABLE views can also be grouped by a field using mainGroupByFieldName
- This creates collapsible sections in the table, organized by the grouping field values
- Works with SELECT fields for categorical grouping

## Filter Operators by Field Type

| Field Type | Available Operators |
|---|---|
| TEXT, EMAILS, FULL_NAME, ADDRESS, LINKS, PHONES | CONTAINS, DOES_NOT_CONTAIN, IS_EMPTY, IS_NOT_EMPTY |
| NUMBER, NUMERIC | IS, IS_NOT, GREATER_THAN_OR_EQUAL, LESS_THAN_OR_EQUAL, IS_EMPTY, IS_NOT_EMPTY |
| CURRENCY | GREATER_THAN_OR_EQUAL, LESS_THAN_OR_EQUAL, IS_EMPTY, IS_NOT_EMPTY |
| DATE, DATE_TIME | IS, IS_RELATIVE, IS_IN_PAST, IS_IN_FUTURE, IS_TODAY, IS_BEFORE, IS_AFTER, IS_EMPTY, IS_NOT_EMPTY |
| SELECT | IS, IS_NOT, IS_EMPTY, IS_NOT_EMPTY |
| MULTI_SELECT, ARRAY | CONTAINS, DOES_NOT_CONTAIN, IS_EMPTY, IS_NOT_EMPTY |
| RELATION | IS, IS_NOT, IS_EMPTY, IS_NOT_EMPTY |
| BOOLEAN | IS |

Using an operator the field type does not support will fail, so check the type first.

## Sorts and Filter Groups

- Directions are ASC (A to Z, oldest first) and DESC (Z to A, newest first).
- Filters combine with AND by default; OR and NOT are available, and groups nest for conditions like name CONTAINS "tech" AND (revenue above 1M OR employees above 100).
- `upsert_complete_view` replaces the whole filter or sort array, so pass the end state you want rather than editing entries one by one.

## Common Filter Patterns

- Overdue work: a DATE field IS_IN_PAST combined with status IS_NOT done
- Upcoming deadlines: a DATE field IS_IN_FUTURE
- Recent activity: updatedAt or createdAt IS_AFTER a date
- Open pipeline: stage IS_NOT the won and lost values
- Unassigned or orphaned records: a relation field IS_EMPTY
- High value: amount GREATER_THAN_OR_EQUAL a threshold

## Common Sort Patterns

- Pipeline boards sort by amount DESC, so the biggest deals read first
- Task lists sort by dueAt ASC, so the most urgent reads first
- Activity lists sort by updatedAt DESC

## Composite Fields

Some fields carry sub-fields you filter on directly:
- CURRENCY uses subFieldName "amountMicros" for the numeric value
- ADDRESS uses sub-fields such as "addressCity" and "addressCountry"
- FULL_NAME uses "firstName" and "lastName"
- EMAILS and LINKS filter on the primary entry

## Approach

- If the user is vague (e.g., "create a board"), ask which object they want to see
- Suggest the most relevant view type based on the object's fields
- After creating a view, configure useful view fields and share a clickable view reference
- Explain what each view type does so users can make informed choices
