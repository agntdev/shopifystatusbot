# Shopify Order Status Bot - UX SPEC Document

## COMMAND TREE
```
/start
  - Initiates conversation, shows welcome message and main menu

/check_order
  - Primary command for order status lookup
  - Can be triggered via command or inline button
  - Triggers order identifier input prompt
```

## DIALOG STATE MACHINE
**States:**
1. **Initial State**  
   - Displays welcome message with /check_order button  
   - Transitions to "Order Identifier Input" on /check_order

2. **Order Identifier Input**  
   - Awaits user input (order number/email)  
   - Transitions to:  
     - "Order Found" if valid order retrieved  
     - "Order Not Found" if lookup fails  
     - "Invalid Input" for malformed input

3. **Order Found**  
   - Displays order status and tracking info  
   - Transitions back to Initial State on /start or back button

4. **Order Not Found**  
   - Shows error message with support contact  
   - Transitions to:  
     - "Order Identifier Input" on retry  
     - Initial State on /start

5. **Invalid Input**  
   - Shows validation error with examples  
   - Transitions to "Order Identifier Input" on retry

## INLINE-KEYBOARD LAYOUT
**Main Menu (Initial State):**
```
[🚀 Check Order Status]
```

**Order Identifier Input:**
```
[1️⃣ By Order Number]  [2️⃣ By Email]
```

**Order Found Screen:**
```
[⬅️ Back to Menu]
```

**Error Screen:**
```
[🔄 Try Again]  [📞 Contact Support]
```

## MESSAGE COPY & TONE
**Welcome Message:**
> "Hi! I'm your Shopify order assistant. How can I help today?"  
> *(with check order button)*

**Order Identifier Prompt:**
> "To check your order status, I'll need either:  
> 1️⃣ Your order number (e.g., #123456)  
> 2️⃣ The email used to place the order"

**Order Found Message:**
> "📦 Order #{{order_number}} Status:  
> {{status}}  
> {{tracking_info}}  
> Need anything else?"  

**Order Not Found Message:**
> "⚠️ We couldn't find that order. Please double-check the details or contact us at:  
> 📧 {{support_email}}  
> 🌐 {{support_url}}"

**Invalid Input Message:**
> "Hmm, that doesn't look valid. Please enter a numeric order number (e.g., #123456) or a valid email address."

## EDGE CASES
1. **Invalid Order Number**  
   - Example: "Order #ABC"  
   - Response: Show validation error with numeric example

2. **Multiple Orders Found**  
   - Response: Show first 3 most recent orders with numbers  
   - Follow-up: "Which order would you like to check?"

3. **Shopify API Timeout**  
   - Response: "⏳ Let me check that for you..."  
   - After 5s: "Connection timed out. Please try again or contact support"

4. **Empty State (No Orders)**  
   - Response: "No orders found for {{email}}. Please check the email or contact support"

5. **Unknown Commands**  
   - Response: "I can only help with order status checks. Use /check_order to get started"

## i18n STRINGS
**Translatable Keys:**
- `welcome.message`  
- `order.identifier.prompt`  
- `order.status.template`  
- `error.not_found`  
- `error.validation`  
- `error.api_timeout`  
- `support.contact_template`  
- `button.check_status`  
- `button.retry`  
- `button.support`

**Support Contact Template:**
> "📧 Email: {{email}}  
> 🌐 Support Page: {{url}}"