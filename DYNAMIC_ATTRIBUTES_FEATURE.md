# Dynamic Guest Attributes Feature

## Overview

The guest list now supports **dynamic custom columns/attributes** that users can add, edit, delete, and reorder - just like managing functions! This provides unlimited flexibility without requiring database schema changes.

## Key Features

### 1. Custom Attribute Types Management
Users can:
- ✅ **Add** new custom attribute types (e.g., "Reception", "Haldi", "Mehendi", "Sangam", etc.)
- ✅ **Edit** existing attribute types (rename, change display name)
- ✅ **Delete** custom attribute types (with confirmation)
- ✅ **Reorder** attribute types to control column order in the guest list
- ✅ **Use Default Attributes** - System provides: Lunch, Dinner, Return Gift, Ceremony

### 2. Guest List Integration
- Custom attributes appear as **columns** in the guest list table
- Each attribute is a **checkbox** that can be toggled for each guest
- Attributes can be edited **inline** after guest is saved
- Export functionality works with any custom attribute

### 3. Database Design

#### GUEST_ATTRIBUTE_TYPES Table
Stores the attribute type definitions:
```sql
- id (Primary Key)
- name (unique identifier, e.g., "lunch", "reception")
- display_name (user-friendly name, e.g., "Lunch", "Reception")
- is_default (boolean - marks default system attributes)
- user_id (Foreign Key - for multi-user support)
- display_order (integer - controls column order)
- created_at, updated_at
```

#### GUEST_ATTRIBUTES Table
Stores the actual checkbox values:
```sql
- id (Primary Key)
- guest_id (Foreign Key to GUESTS)
- attribute_type_id (Foreign Key to GUEST_ATTRIBUTE_TYPES)
- value (boolean - checked/unchecked)
- created_at, updated_at
```

## User Experience Flow

### Adding a New Custom Attribute

1. User navigates to Guest List page
2. Clicks "Manage Attributes" or "Add Custom Column" button
3. Modal/form opens:
   - Enter attribute name (e.g., "Reception")
   - Enter display name (e.g., "Reception Ceremony")
   - Optionally set display order
4. Click "Save"
5. New column appears in guest list immediately
6. All existing guests show unchecked for new attribute
7. User can now check/uncheck this attribute for any guest

### Editing Guest Attributes

1. User views guest list with all custom columns
2. User clicks checkbox in any column for any guest
3. Checkbox toggles immediately (optimistic update)
4. Change is saved to database
5. If save fails, checkbox reverts to previous state

### Managing Attribute Types

1. User clicks "Manage Attributes" button
2. List of all attribute types appears:
   - Default attributes (marked, cannot be deleted)
   - Custom attributes (can be edited/deleted)
3. User can:
   - Click "Edit" to rename an attribute
   - Click "Delete" to remove an attribute (with confirmation)
   - Drag and drop to reorder attributes
4. Changes reflect immediately in guest list

## API Endpoints

### Guest Attribute Types
```
GET    /api/guest-attributes/types          - List all attribute types for user
POST   /api/guest-attributes/types          - Create new attribute type
GET    /api/guest-attributes/types/:id      - Get one attribute type
PUT    /api/guest-attributes/types/:id      - Update attribute type
DELETE /api/guest-attributes/types/:id      - Delete attribute type
PUT    /api/guest-attributes/types/reorder  - Reorder attribute types
GET    /api/guest-attributes/types/defaults - Get default attribute types
```

### Guest Attributes (Values)
```
PATCH  /api/guests/:id/attributes           - Update guest attributes (checkboxes)
GET    /api/guests/:id/attributes           - Get all attributes for a guest
```

## Default Attributes

When a new user signs up, the system automatically creates these default attribute types:

1. **Lunch** (`name: "lunch"`)
2. **Dinner** (`name: "dinner"`)
3. **Return Gift** (`name: "return_gift"`)
4. **Ceremony** (`name: "ceremony"`)

These are marked with `is_default: true` and cannot be deleted, but can be renamed or reordered.

## Example Use Cases

### Use Case 1: Traditional Indian Wedding
User creates custom attributes:
- Reception
- Haldi Ceremony
- Mehendi Ceremony
- Sangeet
- Baraat
- Vidai

### Use Case 2: Multi-Day Wedding
User creates custom attributes:
- Day 1 - Welcome Dinner
- Day 2 - Main Ceremony
- Day 3 - Brunch
- Day 4 - Farewell Party

### Use Case 3: Simple Wedding
User keeps default attributes:
- Lunch
- Dinner
- Return Gift
- Ceremony

## Export Functionality

Export works seamlessly with custom attributes:
- Export by any attribute: `GET /api/guests/export?filter=reception`
- Export full list with all custom columns
- CSV/Excel includes all custom attribute columns

## Benefits

1. **No Schema Changes**: Adding new columns doesn't require database migrations
2. **User Flexibility**: Each user can customize their guest list to match their wedding
3. **Scalability**: Unlimited number of custom attributes per user
4. **Easy Management**: Simple UI to add/edit/delete/reorder attributes
5. **Backward Compatible**: Default attributes ensure existing functionality works
6. **Multi-User Support**: Each user has their own set of custom attributes

## Implementation Notes

### Frontend
- Dynamic table columns based on attribute types
- Inline checkbox editing with optimistic updates
- Drag-and-drop reordering for attributes
- Modal/form for adding/editing attribute types

### Backend
- Validate attribute type names (unique per user)
- Prevent deletion of default attributes
- Cascade delete: When attribute type is deleted, delete all related guest_attributes
- Optimize queries: Load all attribute types once, then load guest attributes

### Database
- Index on `guest_attributes(guest_id, attribute_type_id)` for fast lookups
- Index on `guest_attribute_types(user_id, display_order)` for ordered queries
- Foreign key constraints ensure data integrity

## Migration Strategy

For existing installations:
1. Create default attribute types for existing users
2. Migrate existing boolean columns (reception, lunch, etc.) to guest_attributes table
3. Remove old boolean columns from GUESTS table
4. Update all queries to use new structure

---

This feature makes the guest list as flexible as the function list - users have complete control over their wedding management!

