● Only the migration history files remain (which is correct — they're the
  historical record of what was added and removed). No live code or         
  templates reference Razorpay or INR anymore.                       
                                                                            
  Here's a summary of everything changed:                                   
                                                                            
  File: accounts/models.py                                                  
  Change: Removed 3 Razorpay fields from Cart                               
  ────────────────────────────────────────                                
  File: accounts/views.py                                                 
  Change: Removed import razorpay, rewrote cart view, replaced success view,

    added place_order view, fixed create_order (UUID order_id,
    payment_mode=โอนเงิน)
  ────────────────────────────────────────
  File: accounts/urls.py
  Change: Added place-order/ URL
  ────────────────────────────────────────
  File: templates/accounts/cart.html
  Change: Replaced Razorpay JS checkout with a plain POST form; button now
    says "ยืนยันคำสั่งซื้อ"
  ────────────────────────────────────────
  File: ecomm/settings.py
  Change: Removed RAZORPAY_KEY_ID / RAZORPAY_SECRET_KEY
  ────────────────────────────────────────
  File: .env + .env.example
  Change: Removed Razorpay env vars
  ────────────────────────────────────────
  File: requirements.txt
  Change: Removed razorpay==1.4.2
  ────────────────────────────────────────
  File: accounts/migrations/0017_remove_razorpay_fields.py
  Change: New migration applied
  ────────────────────────────────────────
  File: CLAUDE.md
  Change: Updated description and cart lifecycle docs

  The checkout flow is now: cart → click "ยืนยันคำสั่งซื้อ" → POST to place_order
  → cart marked paid, order created with a ORD-XXXXXXXX UUID id → redirect
  to success page. If you later want to integrate a Thai payment gateway
  (e.g. Omise/Opn or 2C2P), the place_order view is the right place to hook
  it in.