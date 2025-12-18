# Architecture Changes Summary

This document summarizes the modifications made to the architecture based on your requirements.

## 1. Payment-Budget Integration ✅

### Changes Made:
- **Payment table** now includes `budget_item_id` foreign key to link payments to specific budget items
- **Payment categories** are now populated dynamically from existing budget categories (not manually entered)
- **Automatic deduction**: When a payment is made, it automatically:
  - Updates `amount_paid` in the corresponding budget item
  - Recalculates `remaining_amount` = `planned_amount` - `amount_paid`
- **New API endpoint**: `GET /api/payments/categories` - Returns all categories from budget items

### Data Flow:
```
User creates payment → Selects category from budget dropdown → 
Payment saved → Budget item's amount_paid updated → 
Remaining amount recalculated automatically
```

## 2. Guest List - Edit Checkboxes After Saving ✅

### Changes Made:
- Added ability to **edit guest checkboxes inline** after the guest has been saved
- New API endpoint: `PATCH /api/guests/:id/checkboxes` - Updates only the checkbox fields
- Existing `PUT /api/guests/:id` endpoint can also update checkboxes along with other fields
- Guest list view now supports inline editing of checkboxes (Reception, Lunch, Return Gift, etc.)

### User Experience:
- User can add a guest with initial checkboxes
- Later, user can click on any checkbox in the guest list to toggle it
- Changes are saved immediately or via a save button

## 3. Function Table - Removed Timestamps ✅

### Changes Made:
- Removed `created_at` and `updated_at` fields from `FUNCTIONS` table
- Function table now only contains:
  - `id` (Primary Key)
  - `name`
  - `date`
  - `time`
  - `user_id` (Foreign Key)

## 4. User ID Field Explanation ✅

### Purpose:
The `user_id` field in all tables (FUNCTIONS, GUESTS, BUDGET_ITEMS, PAYMENTS) serves the following purpose:

- **Multi-user support**: Allows multiple users to use the same application
- **Data isolation**: Each user only sees and manages their own wedding data
- **Security**: Ensures users cannot access or modify other users' data
- **Scalability**: Enables the app to serve multiple weddings simultaneously

### Example:
- User A (user_id = 1) creates functions, guests, budget for their wedding
- User B (user_id = 2) creates functions, guests, budget for their separate wedding
- When User A logs in, they only see their data (filtered by user_id = 1)
- When User B logs in, they only see their data (filtered by user_id = 2)

## 5. Guest Functions Table - Additional Columns ✅

### Changes Made:
The `GUEST_FUNCTIONS` table now includes additional columns:

- `id` (Primary Key)
- `guest_id` (Foreign Key to GUESTS)
- `function_id` (Foreign Key to FUNCTIONS)
- **`number_of_persons`** (NEW) - Stores how many people are attending for this guest-function combination
- **`notes`** (NEW) - Additional notes or data specific to this guest-function relationship
- `created_at` - Timestamp when record was created
- `updated_at` - Timestamp when record was last updated

### Use Cases:
- Track how many people a guest is bringing to a specific function
- Add notes like "VIP guest", "Special dietary requirements", etc.
- Store any other function-specific information about a guest

## Updated API Endpoints

### Payments
```
GET  /api/payments/categories  - Get all categories from budget items
POST /api/payments              - Create payment (auto-deducts from budget)
PUT  /api/payments/:id          - Update payment (recalculates budget)
DELETE /api/payments/:id        - Delete payment (reverses budget deduction)
```

### Guests
```
PATCH /api/guests/:id/checkboxes - Update only checkbox fields
PUT  /api/guests/:id             - Update guest (including checkboxes)
```

## Updated Database Relationships

```
BUDGET_ITEMS ||--o{ PAYMENTS : "has payments"
```
- One budget item can have multiple payments
- Each payment is linked to a specific budget item via `budget_item_id`
- This ensures payments are properly tracked and deducted from the correct budget category

## Implementation Notes

### Payment Creation Flow:
1. Frontend calls `GET /api/payments/categories` to get available categories
2. User selects a category from the dropdown (populated from budget)
3. User enters payment details
4. Frontend calls `POST /api/payments` with `budget_item_id`
5. Backend:
   - Creates payment record
   - Updates budget item: `amount_paid += payment.amount`
   - Recalculates: `remaining_amount = planned_amount - amount_paid`
6. Returns updated budget information

### Guest Checkbox Editing:
1. User views guest list with checkboxes displayed
2. User clicks/toggles a checkbox
3. Frontend calls `PATCH /api/guests/:id/checkboxes` with updated checkbox values
4. Backend updates only the checkbox fields
5. Frontend refreshes to show updated state

### Budget Remaining Calculation:
- **Initial**: `remaining_amount = planned_amount`
- **After Payment**: `remaining_amount = planned_amount - amount_paid`
- **After Multiple Payments**: `amount_paid = SUM(all payments for this budget item)`
- **After Payment Update**: Recalculate `amount_paid` and `remaining_amount`
- **After Payment Delete**: Subtract deleted payment amount from `amount_paid` and recalculate

---

All changes have been reflected in both `ARCHITECTURE.md` and `ARCHITECTURE_VIEWER.html` files.

