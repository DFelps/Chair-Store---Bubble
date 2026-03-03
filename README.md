# Chair Store - Functional Architecture (Bubble)

## 1. Project Overview

This project is a functional chair e-commerce built in Bubble.

The focus here is not UI or design polish. The goal is to build a
technically solid structure with:

-   clean data modeling
-   proper privacy rules
-   consistent workflows
-   safe order and stock handling

This is meant to simulate a real-world production structure.

### Roles

-   guest
-   customer
-   admin
-   staff

### V1 Scope

For V1, the app includes:

-   Product catalog
-   Product detail page with variants
-   Cart system
-   Checkout flow
-   Order creation
-   Basic admin panel
-   Stock control

------------------------------------------------------------------------

## 2. Option Sets

### UserRole

-   admin
-   staff
-   customer

### OrderStatus

-   pending_payment
-   paid
-   packing
-   shipped
-   delivered
-   canceled
-   refunded

### CouponType (optional)

-   percent
-   fixed

------------------------------------------------------------------------

## 3. Data Model

Core structure of the application.

### Product

  Field         Type            Notes
  ------------- --------------- ---------------------
  title         text            Product name
  slug          text            Unique identifier
  description   text            Basic description
  images        list of image   
  is_active     yes/no          Controls visibility
  featured      yes/no          Optional

### Variant

Each product can have multiple variants (SKU-level control).

  Field              Type      Notes
  ------------------ --------- ----------------------------------
  product            Product   Relation
  sku                text      Unique
  name               text      Example: Black / Wood / With arm
  price              number    
  compare_at_price   number    Optional
  stock_qty          number    Simple stock control (V1)
  is_active          yes/no    

### Cart

  Field   Type               Notes
  ------- ------------------ ----------------
  owner   User               Logged-in user
  items   list of CartItem   

### CartItem

  Field                 Type      Notes
  --------------------- --------- ---------------------------------
  cart                  Cart      
  variant               Variant   
  qty                   number    
  unit_price_snapshot   number    Locks price at add-to-cart time

### Address

  Field    Type   Notes
  -------- ------ -------------------
  owner    User   
  label    text   Home, Office, etc
  zip      text   
  street   text   
  number   text   
  city     text   
  state    text   

### Order

Orders use snapshot logic to avoid historical inconsistencies.

  Field                       Type                Notes
  --------------------------- ------------------- -----------------------
  order_number                text                Generated in workflow
  customer                    User                
  status                      OrderStatus         
  items                       list of OrderItem   
  shipping_address_snapshot   text                Address snapshot
  subtotal                    number              Snapshot
  discount                    number              Optional
  shipping                    number              Optional
  total                       number              Snapshot
  payment_provider            text                stripe / manual
  payment_intent_id           text                Optional
  created_at                  date                

### OrderItem

Order items store full snapshots to preserve pricing history.

  Field                 Type     Notes
  --------------------- -------- ---------------------
  order                 Order    
  variant_snapshot      text     SKU + name snapshot
  unit_price_snapshot   number   
  qty                   number   
  subtotal              number   

------------------------------------------------------------------------

## 4. Privacy Rules

Security is handled.

No sensitive data is searchable by Everyone.

  ------------------------------------------------------------------------
  Data Type                   Access             Condition
  --------------------------- ------------------ -------------------------
  Product                     Everyone           is_active = yes

  Variant                     Everyone           is_active = yes AND
                                                 product.is_active = yes

  Cart                        Owner only         owner = Current User

  CartItem                    Owner only         cart.owner = Current User

  Address                     Owner only         owner = Current User

  Order                       Owner +            customer = Current User
                              Admin/Staff        OR role is admin/staff

  OrderItem                   Owner +            order.customer = Current
                              Admin/Staff        User OR role is
                                                 admin/staff

  User                        Current User +     
                              Admin              
  ------------------------------------------------------------------------

Important: nothing sensitive is globally searchable.

------------------------------------------------------------------------

## 5. Pages Structure

UI is intentionally simple.

  Page             Route                 Purpose
  ---------------- --------------------- ------------------------------------
  Shop             /shop                 List active products
  Product          /p/:slug              Product detail + variant selection
  Cart             /cart                 Manage cart
  Checkout         /checkout             Create order
  Orders           /account/orders       User order history
  Order Detail     /account/orders/:id   Order details
  Admin Products   /admin/products       Manage products and stock
  Admin Orders     /admin/orders         Manage orders

------------------------------------------------------------------------

## 6. Core Workflows

### Cart - Add Item

1.  If user is not logged in → redirect to login\
2.  Find or create Cart\
3.  If CartItem already exists → increment qty\
4.  Otherwise create new CartItem\
5.  Validate qty \<= stock_qty

------------------------------------------------------------------------

### Checkout - Create Order

1.  Validate address\
2.  Revalidate stock for all items\
3.  Create Order with status = pending_payment\
4.  Create OrderItems with snapshots\
5.  Calculate totals

------------------------------------------------------------------------

### Payment Confirmation (V2)

1.  Confirm payment via webhook\
2.  Set Order.status = paid\
3.  Decrement stock_qty\
4.  Optional inventory log

Stock is only decremented after payment confirmation.

------------------------------------------------------------------------

## 7. Roadmap

### Phase 1 - MVP

-   Data model defined\
-   Privacy rules implemented\
-   Catalog working\
-   Cart working\
-   Order creation flow\
-   Basic admin panel

------------------------------------------------------------------------

### Phase 2 - More Robust

-   Stripe integration\
-   Webhooks\
-   Coupon system\
-   Inventory movement log\
-   Email notifications

------------------------------------------------------------------------

### Phase 3 - Growth

-   Guest cart\
-   Shipping calculation\
-   Reviews\
-   SEO improvements\
-   Analytics dashboard

------------------------------------------------------------------------

End of Documentation
