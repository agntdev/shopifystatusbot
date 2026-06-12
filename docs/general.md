# Shopify Order Status Bot - GENERAL Design Document

## Summary
This Telegram bot enables Shopify store customers to check their order status by integrating with the Shopify API. It provides real-time order details like status, tracking information, and items, while handling ambiguous cases by offering support contact details. Designed for end-users who need quick access to order information without requiring technical knowledge.

## Core Entities
- **Order**: Contains status, tracking information, items, and Shopify order ID.
- **Session**: Tracks user interaction state (e.g., awaiting order number input).
- **Customer**: Implicitly represented via order lookup (no direct storage of customer data).

## External Dependencies
- **Telegram Bot API**: 
  - Text message handling
  - Command-based interactions
- **Shopify API**: 
  - Order lookup by order number/email
  - Order status and tracking data retrieval
- **Persistence**: 
  - Session state storage (e.g., PostgreSQL or Redis)
  - No long-term customer/order data storage

## Full Feature List
- Check order status via order number lookup
- Check order status via customer email lookup
- Display tracking information if available
- Handle ambiguous order lookups (multiple orders, no orders)
- Provide pre-defined support contact details when orders are not found
- Maintain session state for multi-step interactions
- Handle Shopify API errors gracefully (e.g., API downtime)
- Support internationalization for order status messages

## Non-Goals
- Order modification (cancellations, changes)
- Product catalog information
- Payment processing or refund handling
- Return authorization workflows
- Shipping policy explanations
- User account management features