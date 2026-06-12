# Shopify Order Status Bot - DETAILS Document

## SCREENS

### 1. Initial State
- **Trigger**: `/start` command or session reset
- **Message**:  
  > "Hi! I'm your Shopify order assistant. How can I help today?"  
  (Uses `welcome.message` i18n key)
- **Keyboard**:  
  ```
  [🚀 Check Order Status]
  ```
- **Transitions**:  
  - Button click → `Order Identifier Input` state

---

### 2. Order Identifier Input
- **Trigger**: Transition from Initial State or Error Screen
- **Message**:  
  > "To check your order status, I'll need either:  
  1️⃣ Your order number (e.g., #123456)  
  2️⃣ The email used to place the order"  
  (Uses `order.identifier.prompt` i18n key)
- **Keyboard**:  
  ```
  [1️⃣ By Order Number]  [2️⃣ By Email]
  ```
- **Transitions**:  
  - Button 1 → `Order Number Input` state  
  - Button 2 → `Email Input` state

---

### 3. Order Number Input
- **Trigger**: User enters numeric order number (e.g., `#123456`)  
- **Message**:  
  > "Looking up order #{{order_number}}..."  
- **Keyboard**: None  
- **Transitions**:  
  - Shopify API success → `Order Found` state  
  - Shopify API error → `API Error` state  
  - No order found → `Order Not Found` state  
  - Multiple orders found → `Multiple Orders Found` state

---

### 4. Email Input
- **Trigger**: User enters email (e.g., `customer@example.com`)  
- **Message**:  
  > "Looking up orders for {{email}}..."  
- **Keyboard**: None  
- **Transitions**:  
  - Shopify API success → `Order Found` state  
  - Shopify API error → `API Error` state  
  - No orders found → `Order Not Found` state  
  - Multiple orders found → `Multiple Orders Found` state

---

### 5. Order Found
- **Trigger**: Valid order retrieved from Shopify API
- **Message**:  
  > "📦 Order #{{order_number}} Status:  
  {{status}}  
  {{tracking_info}}  
  Need anything else?"  
  (Uses `order.status.template` i18n key)
- **Keyboard**:  
  ```
  [⬅️ Back to Menu]
  ```
- **Transitions**:  
  - Button click → `Initial State`  
  - Any other input → `Initial State`

---

### 6. Multiple Orders Found
- **Trigger**: Shopify API returns ≥1 orders for identifier
- **Message**:  
  > "We found multiple orders. Here are the 3 most recent:  
  {{order_list}}  
  Which order would you like to check?"  
- **Keyboard**:  
  ```
  [{{order_number_1}}]  [{{order_number_2}}]  [{{order_number_3}}]  
  [⬅️ Back to Menu]
  ```
- **Transitions**:  
  - Order number selection → `Order Found` state  
  - "More..." button (if applicable) → `Paginator` component  
  - Button click → `Initial State`

---

### 7. Order Not Found
- **Trigger**: No order matches identifier
- **Message**:  
  > "⚠️ We couldn't find that order. Please double-check the details or contact us at:  
  {{support_contact}}"  
  (Uses `error.not_found` and `support.contact_template` i18n keys)
- **Keyboard**:  
  ```
  [🔄 Try Again]  [📞 Contact Support]
  ```
- **Transitions**:  
  - "Try Again" → `Order Identifier Input`  
  - "Contact Support" → `Initial State` with support message

---

### 8. Invalid Input
- **Trigger**: User enters non-numeric order number or malformed email
- **Message**:  
  > "Hmm, that doesn't look valid. Please enter a numeric order number (e.g., #123456) or a valid email address."  
  (Uses `error.validation` i18n key)
- **Keyboard**:  
  ```
  [🔄 Try Again]
  ```
- **Transitions**:  
  - "Try Again" → `Order Identifier Input`  
  - Any other input → `Order Identifier Input`

---

### 9. API Error
- **Trigger**: Shopify API timeout/failure
- **Message**:  
  > "⏳ Let me check that for you..."  
  > (After 5s) "Connection timed out. Please try again or contact support"  
  (Uses `error.api_timeout` i18n key)
- **Keyboard**:  
  ```
  [🔄 Retry]  [📞 Contact Support]
  ```
- **Transitions**:  
  - "Retry" → Reattempt API call in current state  
  - "Contact Support" → `Initial State` with support message

---

## COMPONENTS

### 1. Main Menu Keyboard
- **Structure**:  
  ```
  [🚀 Check Order Status]
  ```
- **Behavior**: Always returns to `Initial State`

---

### 2. Order Identifier Prompt Keyboard
- **Structure**:  
  ```
  [1️⃣ By Order Number]  [2️⃣ By Email]
  ```
- **Behavior**:  
  - "By Order Number" → Activates numeric input validation  
  - "By Email" → Activates email regex validation

---

### 3. Paginator (for multiple orders)
- **Structure**:  
  ```
  [<< Previous]  [Next >>]  [⬅️ Back to Menu]
  ```
- **Behavior**:  
  - Shows 3 orders per page  
  - Loads next/previous orders from Shopify API  
  - "Back to Menu" → `Initial State`

---

### 4. Support Contact Card
- **Structure**:  
  > "📧 Email: {{email}}  
  > 🌐 Support Page: {{url}}"  
  (Uses `support.contact_template` i18n key)
- **Behavior**:  
  - Displays static support details from config  
  - "Contact Support" button → Sends this card to user

---

## TRANSITIONS

| Current State       | Input/Callback                     | Next State              | Side Effects                                  |
|---------------------|------------------------------------|-------------------------|-----------------------------------------------|
| Initial State       | `/start` or "Back to Menu"         | Initial State           | Clear session state                             |
| Initial State       | "🚀 Check Order Status"              | Order Identifier Input  | Store `state: "identifier_input"` in session  |
| Order Identifier Input | "1️⃣ By Order Number"              | Order Number Input      | Show input prompt with order number focus     |
| Order Identifier Input | "2️⃣ By Email"                      | Email Input             | Show input prompt with email focus            |
| Order Number Input  | Valid order number                 | Order Found             | Fetch order from Shopify API                  |
| Order Number Input  | Invalid order number               | Invalid Input           | Store invalid input in session                |
| Email Input         | Valid email                        | Order Found             | Fetch orders from Shopify API                 |
| Email Input         | Invalid email                      | Invalid Input           | Store invalid input in session                |
| Multiple Orders Found | Order number selection             | Order Found             | Fetch specific order from Shopify API         |
| Multiple Orders Found | "More..."                          | Paginator               | Load next page of orders                      |
| Order Not Found     | "🔄 Try Again"                      | Order Identifier Input  | Clear previous identifier                     |
| API Error           | "🔄 Retry"                          | Reattempt API call      | Resubmit lookup with same identifier          |

---

## DATA

### Entities
1. **Session**  
   - `session_id` (string, key)  
   - `state` (string: "initial", "identifier_input", "order_found", etc.)  
   - `temp_data` (JSON: {identifier_type, identifier_value, orders_list, current_page})

2. **Order**  
   - `shopify_order_id` (string, foreign key to Shopify)  
   - `order_number` (string)  
   - `status` (string: "fulfilled", "pending", etc.)  
   - `tracking_info` (string or null)  
   - `customer_email` (string)

---

## ACCEPTANCE NOTES

1. **Order Number Validation**  
   - Must accept input with/without `#` prefix  
   - Reject non-numeric characters (e.g., `#ABC123`) with `Invalid Input` screen

2. **Email Validation**  
   - Must match standard email regex  
   - Case-insensitive lookup via Shopify API

3. **Multiple Orders Handling**  
   - When ≥3 orders found, show first 3 with "More..." button  
   - When <3 orders found, show all with no paginator

4. **API Error Recovery**  
   - Retry logic limited to 3 attempts  
   - After 3 failures, lock session and show support contact

5. **Support Contact Display**  
   - Always includes both email and URL from config  
   - "Contact Support" button must appear on all error screens

6. **Session Expiry**  
   - Inactive sessions expire after 5 minutes  
   - Expired session → `Initial State` with warning message

7. **Internationalization**  
   - All messages must use i18n keys from UX spec  
   - Support for English, Spanish, and French initially