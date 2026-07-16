# 💳 Payment Systems Architecture — How Payments Actually Work

> **"When you scan a QR code at a chai stall and ₹20 is deducted in 2 seconds — this is what happens behind the scenes."**

---

## 📑 Table of Contents
1. [How a Payment Works (End-to-End Flow)](#1-how-a-payment-works)
2. [Key Players in a Payment System](#2-key-players)
3. [UPI — How It Really Works (India's System)](#3-upi-how-it-works)
4. [QR Code Payment Flow (Step by Step)](#4-qr-code-payment-flow)
5. [Card Payment Flow (Visa/Mastercard)](#5-card-payment-flow)
6. [Ticket Booking — How BookMyShow/IRCTC Handles Payments](#6-ticket-booking-payment-flow)
7. [Payment Gateway Integration (Razorpay/Stripe in Spring Boot)](#7-payment-gateway-integration)
8. [Webhook — How Payment Confirmation Reaches Your Server](#8-webhooks)
9. [Idempotency — Preventing Double Payments](#9-idempotency)
10. [Refund Flow](#10-refund-flow)
11. [Payment Status & Reconciliation](#11-payment-status--reconciliation)
12. [Wallet System (Paytm Wallet / In-App Wallet)](#12-wallet-system)
13. [Subscription / Recurring Payments](#13-subscription-payments)
14. [Security in Payments](#14-security)
15. [Database Schema for a Payment System](#15-database-schema)
16. [Complete Spring Boot Payment Service](#16-complete-spring-boot-implementation)
17. [Interview Questions (20+)](#17-interview-questions)

---

## 1. How a Payment Works (End-to-End Flow)

Every payment, whether UPI, card, or net banking, follows this basic flow:

```
┌──────────┐     ┌──────────────┐     ┌─────────────────┐     ┌──────────────┐     ┌──────────┐
│  Customer │────▶│  Merchant's  │────▶│ Payment Gateway  │────▶│  Acquiring   │────▶│  Issuing │
│  (You)    │     │  App/Website │     │ (Razorpay/Stripe)│     │  Bank        │     │  Bank    │
│           │◀────│              │◀────│                  │◀────│  (Merchant's │◀────│  (Your   │
│           │     │              │     │                  │     │   Bank)      │     │   Bank)  │
└──────────┘     └──────────────┘     └─────────────────┘     └──────────────┘     └──────────┘
    Pays              Initiates            Routes &                Receives            Deducts
                     Payment              Processes              Money               Money

Step 1: You click "Pay ₹500" on BookMyShow
Step 2: BookMyShow sends payment request to Razorpay
Step 3: Razorpay shows you payment options (UPI, Card, NetBanking)
Step 4: You select UPI, enter PIN
Step 5: Razorpay routes to NPCI (UPI network)
Step 6: NPCI talks to your bank (Issuing Bank)
Step 7: Your bank verifies balance, deducts ₹500
Step 8: Money reaches merchant's bank (Acquiring Bank)
Step 9: Razorpay sends "SUCCESS" webhook to BookMyShow
Step 10: BookMyShow confirms your ticket!

Total time: 2-5 seconds ⚡
```

---

## 2. Key Players in a Payment System

```
┌─────────────────────────────────────────────────────────────────┐
│                    PAYMENT ECOSYSTEM                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CUSTOMER (You)                                                  │
│  └─ Has a bank account / card / UPI ID                          │
│                                                                  │
│  MERCHANT (Amazon, Zomato, BookMyShow)                           │
│  └─ The business accepting payment                               │
│                                                                  │
│  PAYMENT GATEWAY (Razorpay, Stripe, PayU, CCAvenue)              │
│  └─ Middleware that connects merchant to banks                   │
│  └─ Handles: card tokenization, retry, multiple payment modes    │
│  └─ Charges: 1.5-2.5% per transaction                           │
│                                                                  │
│  PAYMENT PROCESSOR / SWITCH                                      │
│  └─ Routes transactions to the right network                    │
│  └─ Examples: NPCI (UPI), Visa/Mastercard (cards)               │
│                                                                  │
│  ISSUING BANK (Your bank — SBI, HDFC, ICICI)                    │
│  └─ Verifies your balance, deducts money                        │
│                                                                  │
│  ACQUIRING BANK (Merchant's bank)                                │
│  └─ Receives the money on behalf of the merchant                │
│                                                                  │
│  NPCI (National Payments Corporation of India)                   │
│  └─ Operates UPI, IMPS, NEFT, RuPay                            │
│  └─ Central switch that connects 300+ banks                     │
│                                                                  │
│  RBI (Regulator)                                                 │
│  └─ Sets rules, mandates tokenization, controls policies        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. UPI — How It Really Works

### What is UPI?
**Unified Payments Interface** — India's real-time payment system. Built by NPCI on top of IMPS (Immediate Payment Service). Connects 300+ banks through a single API.

### UPI Architecture

```
┌───────────┐     ┌──────────────┐     ┌────────┐     ┌──────────────┐     ┌───────────┐
│  Google   │     │  PSP Server  │     │  NPCI  │     │  Remitter    │     │  Beneficiary│
│  Pay App  │────▶│  (GPay's     │────▶│  (UPI  │────▶│  Bank        │────▶│  Bank       │
│  (TPApp)  │     │   Server)    │     │ Switch)│     │  (Your Bank) │     │ (Merchant's)│
└───────────┘     └──────────────┘     └────────┘     └──────────────┘     └───────────┘
  Third-Party       Payment Service     Central          Sender's           Receiver's
  App (UI)          Provider            Switch           Bank               Bank

Key Terms:
- TPApp: Third Party App (GPay, PhonePe, Paytm — the app you see)
- PSP: Payment Service Provider (the bank behind the app)
  - GPay's PSP → Axis Bank, ICICI, SBI, HDFC
  - PhonePe's PSP → Yes Bank → ICICI
  - Paytm's PSP → Paytm Payments Bank
- NPCI: The central switch connecting all banks
- VPA: Virtual Payment Address (yourname@okaxis, yourname@ybl)
```

### UPI Payment Flow (What happens in those 2 seconds)

```
You send ₹500 to chai@paytm via Google Pay:

Step  Time     Action
────  ───────  ──────────────────────────────────────────
 1    0ms      You enter chai@paytm, amount ₹500, enter UPI PIN
 2    50ms     GPay app encrypts UPI PIN using device-bound key
 3    100ms    GPay sends payment request to its PSP (e.g., Axis Bank)
 4    200ms    PSP validates request, generates transaction ID
 5    300ms    PSP sends to NPCI UPI Switch
 6    400ms    NPCI resolves chai@paytm → finds it's on Paytm Payments Bank
 7    500ms    NPCI sends DEBIT request to YOUR bank (remitter)
 8    700ms    Your bank:
                 - Decrypts UPI PIN
                 - Verifies PIN is correct
                 - Checks balance ≥ ₹500
                 - Places HOLD on ₹500
                 - Sends "DEBIT SUCCESS" to NPCI
 9    900ms    NPCI sends CREDIT request to Paytm Payments Bank (beneficiary)
10    1100ms   Paytm Payments Bank:
                 - Credits ₹500 to chai's account
                 - Sends "CREDIT SUCCESS" to NPCI
11    1300ms   NPCI sends SUCCESS to your PSP (Axis Bank)
12    1500ms   PSP sends SUCCESS to GPay app
13    1700ms   GPay shows ✅ "Payment Successful"
14    1800ms   Both you and chai get SMS/notification

Total: ~2 seconds!
```

### UPI Collect Request (When merchant requests money)

```
Zomato wants ₹350 from you:

1. Zomato → sends "Collect Request" to your UPI ID
2. You receive notification: "Zomato is requesting ₹350"
3. You open GPay → see the request → enter UPI PIN to approve
4. Same flow as above (debit → credit)

This is how IRCTC, electricity bills, etc. work!
```

### UPI Intent Flow (Deep Link — Most Common Today)

```
Amazon checkout → Click "Pay via UPI":

1. Amazon generates a UPI intent URL:
   upi://pay?pa=amazon@ybl&pn=Amazon&am=1999&cu=INR&tr=TXN12345

2. Your phone opens GPay/PhonePe (based on your default UPI app)
3. App auto-fills: To=amazon@ybl, Amount=₹1999
4. You enter PIN → payment completes
5. Amazon's server gets webhook → confirms order

pa = payee address (VPA)
pn = payee name
am = amount
cu = currency
tr = transaction reference
```

---

## 4. QR Code Payment Flow (Step by Step)

### Static QR (Fixed — printed at shops)

```
The QR code at your local shop simply contains:

upi://pay?pa=shopkeeper@paytm&pn=Raj%20Kirana%20Store

That's it! It's just a UPI intent URL encoded as a QR.

When you scan it:
1. Camera reads QR → decodes to UPI URL
2. Your UPI app opens → payee is pre-filled
3. You enter amount (₹50) + UPI PIN
4. Payment flows through NPCI as described above
```

### Dynamic QR (Generated per transaction — like on payment machines)

```
When you order food at McDonald's:

1. Cashier enters ₹450 in POS machine
2. Machine generates a UNIQUE QR containing:
   upi://pay?pa=mcdonalds@icici&pn=McDonalds&am=450&tr=TXN_MC_98765&cu=INR

3. You scan → amount is PRE-FILLED (₹450) → enter PIN
4. Payment completes
5. POS machine immediately shows ✅ (it polls its server OR gets push notification)

Key difference: Dynamic QR has amount + unique transaction ID pre-filled
So the merchant's machine can map THIS specific QR scan to THIS specific order!
```

### How does the shop's machine know INSTANTLY?

```
Two mechanisms:

1. POLLING (Simple):
   Machine asks server every 1 second: "Is TXN_MC_98765 paid?"
   Server checks database → "YES" → machine shows ✅

2. PUSH NOTIFICATION / WEBHOOK (Better):
   Payment gateway sends POST request to merchant's server
   Server pushes update to machine via WebSocket
   Machine shows ✅ immediately

3. UPI CALLBACK (Direct):
   NPCI sends callback to PSP → PSP sends to merchant
   Sub-second notification
```

---

## 5. Card Payment Flow (Visa/Mastercard)

```
You buy shoes on Amazon with HDFC Credit Card:

┌───────────┐    ┌──────────┐    ┌──────────────┐    ┌──────────┐    ┌──────────┐
│  Amazon   │───▶│ Razorpay │───▶│ Card Network │───▶│ Issuing  │    │Acquiring │
│  Checkout │    │ (Gateway)│    │ (Visa/MC)    │    │ Bank     │    │ Bank     │
│           │◀───│          │◀───│              │◀───│ (HDFC)   │    │(Amazon's)│
└───────────┘    └──────────┘    └──────────────┘    └──────────┘    └──────────┘

Step 1: AUTHORIZATION (Does the customer have money?)
  - You enter card details on Amazon
  - Razorpay sends card info to Visa network
  - Visa routes to HDFC (your card issuer)
  - HDFC checks: Is card valid? Enough limit? No fraud?
  - HDFC places a HOLD on ₹3000 (not yet deducted!)
  - HDFC sends "AUTHORIZED" back through the chain
  - Amazon shows "Order Confirmed!" ✅

Step 2: CAPTURE (Actually take the money) — Hours/Days later
  - Amazon ships the shoes
  - Amazon tells Razorpay: "Capture this ₹3000"
  - Money moves from HDFC → Visa → Amazon's bank

Step 3: SETTLEMENT (Merchant gets money) — T+1 to T+3 days
  - Razorpay settles the amount to Amazon's bank account
  - Minus gateway fees (1.5-2%)

Why two steps (Auth + Capture)?
  - Hotels: Authorize ₹10,000 at check-in, capture actual bill (₹7,500) at checkout
  - E-commerce: Authorize at order, capture at shipping
  - If order cancelled before capture → just RELEASE the hold (no refund needed!)
```

### 3D Secure (OTP Flow)

```
Why you get OTP for online card payments in India:

1. You enter card number, expiry, CVV on Amazon
2. Razorpay sends to card network
3. Card network says: "This card requires 3D Secure"
4. Browser redirects to HDFC's page
5. HDFC sends OTP to your registered phone
6. You enter OTP on HDFC's page
7. HDFC verifies → sends "AUTHENTICATED" to Visa
8. Payment continues

3D Secure = Additional authentication layer
RBI mandates it for all online card transactions in India
International: "Verified by Visa" / "Mastercard SecureCode"
```

---

## 6. Ticket Booking — How BookMyShow/IRCTC Handles Payments

```
The HARDEST part: 1000 users trying to book the SAME 500 seats simultaneously!

═══ IRCTC Tatkal Booking Flow ═══

10:00:00 AM — Tatkal opens. 50,000 users hit the server.

Step 1: SEAT LOCK (Temporary Hold)
  - You select Train, Class, Date → click "Book"
  - Server runs: SELECT ... FOR UPDATE (database row lock)
  - If seats available:
    - Status: AVAILABLE → TEMPORARILY_HELD
    - Timer starts: 10 minutes to complete payment
    - Other users see these seats as "unavailable"
  - If not: "Seats not available" ❌

Step 2: PAYMENT INITIATION
  - You select UPI/Card → redirected to payment page
  - Payment gateway creates a payment session
  - Timer: 10 minutes (IRCTC) / 8 minutes (BookMyShow)

Step 3: PAYMENT PROCESSING
  - You complete payment (UPI PIN / OTP)
  - Gateway processes → sends result

Step 4a: PAYMENT SUCCESS
  - Gateway webhook hits IRCTC server: "Payment ₹1500 SUCCESS"
  - IRCTC: TEMPORARILY_HELD → CONFIRMED
  - PNR generated, ticket issued
  - SMS + Email sent

Step 4b: PAYMENT FAILURE / TIMEOUT
  - Gateway says FAILED or 10 minutes expire
  - IRCTC: TEMPORARILY_HELD → AVAILABLE (seats released back!)
  - Other users can now book those seats

Step 4c: PAYMENT STUCK (Neither success nor failure)
  - Most complex case!
  - IRCTC keeps seats in HELD state
  - Runs a background job every 2 minutes:
    - Calls gateway: "What's the status of TXN_12345?"
    - If SUCCESS → confirm booking
    - If FAILED → release seats
    - If STILL PENDING → wait more (up to 30 minutes)
  - If money debited but booking not confirmed → auto-refund after 24-48 hours
```

### BookMyShow Architecture for Payment

```
Why BookMyShow blocks your seats before payment:

┌───────────┐    ┌─────────────┐    ┌──────────┐    ┌──────────────┐
│  User     │───▶│  Seat Lock  │───▶│ Payment  │───▶│  Confirm     │
│  Selects  │    │  Service    │    │ Service  │    │  Booking     │
│  Seats    │    │ (Redis/DB)  │    │          │    │  Service     │
└───────────┘    └─────────────┘    └──────────┘    └──────────────┘
                      │                                    │
                 ┌────▼────┐                          ┌────▼────┐
                 │  Timer  │                          │ Generate│
                 │ 8 mins  │                          │ Ticket  │
                 │         │                          │ + QR    │
                 └─────────┘                          └─────────┘
                 (If expired,                    (Send via Email/SMS)
                  release seats)

Seat Locking Implementation:
```

```java
@Service
public class SeatLockService {
    
    @Autowired private StringRedisTemplate redis;
    
    // Lock seats for 8 minutes while user pays
    public boolean lockSeats(String showId, List<String> seatIds, String userId) {
        String lockValue = userId + ":" + System.currentTimeMillis();
        
        // Try to lock each seat atomically
        for (String seatId : seatIds) {
            String key = "seat_lock:" + showId + ":" + seatId;
            Boolean locked = redis.opsForValue()
                .setIfAbsent(key, lockValue, Duration.ofMinutes(8));
            
            if (!Boolean.TRUE.equals(locked)) {
                // Seat already locked by someone else → rollback
                releaseSeatsBefore(showId, seatIds, seatId, userId);
                return false; // ❌ Seats not available
            }
        }
        return true; // ✅ All seats locked for this user
    }
    
    // Release seats (on payment failure or timeout)
    public void releaseSeats(String showId, List<String> seatIds, String userId) {
        for (String seatId : seatIds) {
            String key = "seat_lock:" + showId + ":" + seatId;
            String owner = redis.opsForValue().get(key);
            
            // Only release if WE own the lock
            if (owner != null && owner.startsWith(userId + ":")) {
                redis.delete(key);
            }
        }
    }
    
    // Check if seats are available
    public boolean areSeatsAvailable(String showId, List<String> seatIds) {
        for (String seatId : seatIds) {
            String key = "seat_lock:" + showId + ":" + seatId;
            if (Boolean.TRUE.equals(redis.hasKey(key))) {
                return false; // At least one seat is locked
            }
        }
        return true;
    }
}
```

---

## 7. Payment Gateway Integration (Razorpay in Spring Boot)

### pom.xml
```xml
<dependency>
    <groupId>com.razorpay</groupId>
    <artifactId>razorpay-java</artifactId>
    <version>1.4.3</version>
</dependency>
```

### Step 1: Create Order on Server

```java
@RestController
@RequestMapping("/api/payments")
public class PaymentController {
    
    @Value("${razorpay.key.id}") private String keyId;
    @Value("${razorpay.key.secret}") private String keySecret;
    
    // Step 1: Create Razorpay Order (called when user clicks "Pay")
    @PostMapping("/create-order")
    public Map<String, Object> createOrder(@RequestBody PaymentRequest request) throws Exception {
        
        RazorpayClient client = new RazorpayClient(keyId, keySecret);
        
        JSONObject options = new JSONObject();
        options.put("amount", request.getAmount() * 100);  // Amount in PAISE (₹500 = 50000)
        options.put("currency", "INR");
        options.put("receipt", "order_" + System.currentTimeMillis());
        options.put("payment_capture", 1);  // Auto-capture
        
        Order order = client.orders.create(options);
        
        // Save to database
        PaymentOrder paymentOrder = new PaymentOrder();
        paymentOrder.setRazorpayOrderId(order.get("id"));
        paymentOrder.setAmount(request.getAmount());
        paymentOrder.setStatus("CREATED");
        paymentOrder.setUserId(request.getUserId());
        paymentOrderRepo.save(paymentOrder);
        
        // Return to frontend
        return Map.of(
            "orderId", order.get("id"),
            "amount", order.get("amount"),
            "currency", "INR",
            "keyId", keyId    // Frontend needs this to open Razorpay checkout
        );
    }
}
```

### Step 2: Frontend Opens Razorpay Checkout

```javascript
// Frontend (React/HTML)
var options = {
    key: response.keyId,
    amount: response.amount,
    currency: "INR",
    name: "BookMyShow",
    order_id: response.orderId,    // From Step 1
    
    handler: function(paymentResponse) {
        // Payment successful on Razorpay's end
        // Send to YOUR server for verification
        fetch('/api/payments/verify', {
            method: 'POST',
            body: JSON.stringify({
                razorpay_order_id: paymentResponse.razorpay_order_id,
                razorpay_payment_id: paymentResponse.razorpay_payment_id,
                razorpay_signature: paymentResponse.razorpay_signature
            })
        });
    },
    
    prefill: {
        name: "Dilip",
        email: "dilip@example.com",
        contact: "9999999999"
    },
    theme: { color: "#F37254" }
};

var rzp = new Razorpay(options);
rzp.open();
```

### Step 3: Verify Payment on Server (CRITICAL!)

```java
// ⚠️ NEVER trust the frontend! Always verify server-side!
@PostMapping("/verify")
public ResponseEntity<?> verifyPayment(@RequestBody PaymentVerification verification) {
    
    // Generate expected signature
    String data = verification.getRazorpayOrderId() + "|" + verification.getRazorpayPaymentId();
    String expectedSignature = HmacUtils.hmacSha256Hex(keySecret, data);
    
    if (!expectedSignature.equals(verification.getRazorpaySignature())) {
        // ❌ TAMPERED! Someone modified the payment on frontend
        return ResponseEntity.status(400).body("Payment verification failed!");
    }
    
    // ✅ Signature verified — payment is genuine
    PaymentOrder order = paymentOrderRepo.findByRazorpayOrderId(
        verification.getRazorpayOrderId());
    order.setStatus("PAID");
    order.setRazorpayPaymentId(verification.getRazorpayPaymentId());
    paymentOrderRepo.save(order);
    
    // Trigger post-payment actions (confirm booking, send email, etc.)
    bookingService.confirmBooking(order.getBookingId());
    
    return ResponseEntity.ok(Map.of("status", "success"));
}
```

---

## 8. Webhooks — How Payment Confirmation Reaches Your Server

```
Problem: What if the user's browser closes AFTER payment but BEFORE 
         the verify API is called? Payment is done but you don't know!

Solution: WEBHOOKS — Razorpay calls YOUR server directly.

Your Server ◀──── POST /api/webhooks/razorpay ──── Razorpay Server

Razorpay sends payment status to your webhook URL regardless of 
whether the user's browser is open or not.
```

```java
@RestController
@RequestMapping("/api/webhooks")
public class PaymentWebhookController {
    
    @Value("${razorpay.webhook.secret}") private String webhookSecret;
    
    @PostMapping("/razorpay")
    public ResponseEntity<String> handleRazorpayWebhook(
            @RequestBody String payload,
            @RequestHeader("X-Razorpay-Signature") String signature) {
        
        // Step 1: Verify webhook signature (prevent spoofing)
        String expectedSignature = HmacUtils.hmacSha256Hex(webhookSecret, payload);
        if (!expectedSignature.equals(signature)) {
            return ResponseEntity.status(400).body("Invalid signature");
        }
        
        // Step 2: Parse the event
        JSONObject event = new JSONObject(payload);
        String eventType = event.getString("event");
        
        switch (eventType) {
            case "payment.captured":
                handlePaymentCaptured(event.getJSONObject("payload")
                    .getJSONObject("payment").getJSONObject("entity"));
                break;
                
            case "payment.failed":
                handlePaymentFailed(event.getJSONObject("payload")
                    .getJSONObject("payment").getJSONObject("entity"));
                break;
                
            case "refund.processed":
                handleRefundProcessed(event);
                break;
        }
        
        // Step 3: ALWAYS return 200 OK (otherwise Razorpay will retry)
        return ResponseEntity.ok("Webhook received");
    }
    
    private void handlePaymentCaptured(JSONObject payment) {
        String orderId = payment.getString("order_id");
        String paymentId = payment.getString("id");
        int amount = payment.getInt("amount"); // In paise
        
        // Update database
        PaymentOrder order = paymentOrderRepo.findByRazorpayOrderId(orderId);
        
        // IDEMPOTENCY: Check if already processed
        if ("PAID".equals(order.getStatus())) return; // Already processed!
        
        order.setStatus("PAID");
        order.setRazorpayPaymentId(paymentId);
        paymentOrderRepo.save(order);
        
        // Trigger business logic
        bookingService.confirmBooking(order.getBookingId());
        emailService.sendConfirmation(order.getUserId());
    }
}
```

### Webhook vs Polling

```
Webhook (PUSH — Recommended):
  Razorpay calls YOUR server when payment status changes.
  ✅ Real-time, no unnecessary calls
  ❌ Must handle retries, idempotency, signature verification

Polling (PULL — Fallback):
  Your server calls Razorpay every N seconds: "Is TXN done?"
  ✅ Simple to implement
  ❌ Wasteful, not real-time

Best Practice: Use BOTH!
  - Webhook as primary notification
  - Polling as fallback (cron job every 5 mins to check pending payments)
```

---

## 9. Idempotency — Preventing Double Payments

```
Scenario: User clicks "Pay" → network timeout → user clicks "Pay" again
Without idempotency: ₹500 charged TWICE!
With idempotency: Second click returns cached result of first payment.
```

```java
@Service
public class PaymentService {
    
    @Autowired private StringRedisTemplate redis;
    @Autowired private PaymentOrderRepository repo;
    
    public PaymentResponse initiatePayment(String idempotencyKey, PaymentRequest request) {
        
        // Check if this request was already processed
        String cachedResult = redis.opsForValue().get("idem:" + idempotencyKey);
        if (cachedResult != null) {
            return objectMapper.readValue(cachedResult, PaymentResponse.class);
            // Return SAME result without charging again!
        }
        
        // Process payment...
        PaymentResponse response = processWithGateway(request);
        
        // Cache result for 24 hours
        redis.opsForValue().set("idem:" + idempotencyKey, 
            objectMapper.writeValueAsString(response), Duration.ofHours(24));
        
        return response;
    }
}

// Frontend sends:
// POST /api/payments
// Headers: Idempotency-Key: order_12345_attempt_1
```

---

## 10. Refund Flow

```
Customer requests refund for ₹500 order:

1. Merchant initiates refund via Razorpay API
2. Razorpay sends refund request to Acquiring Bank
3. Acquiring Bank → Card Network / NPCI → Issuing Bank
4. Issuing Bank credits ₹500 back to customer
5. Webhook: "refund.processed" sent to merchant

Timeline:
  UPI Refund:     Instant to 48 hours
  Card Refund:    5-7 business days (!)
  Net Banking:    3-5 business days
  Wallet:         Instant
```

```java
// Initiate Refund
@PostMapping("/refund")
public Map<String, Object> refund(@RequestBody RefundRequest request) throws Exception {
    
    RazorpayClient client = new RazorpayClient(keyId, keySecret);
    
    JSONObject options = new JSONObject();
    options.put("amount", request.getAmount() * 100); // Partial or full refund
    options.put("speed", "normal"); // "normal" (5-7 days) or "optimum" (instant for UPI)
    
    Refund refund = client.payments.refund(request.getPaymentId(), options);
    
    // Update database
    PaymentOrder order = paymentOrderRepo.findByRazorpayPaymentId(request.getPaymentId());
    order.setStatus("REFUND_INITIATED");
    order.setRefundId(refund.get("id"));
    paymentOrderRepo.save(order);
    
    return Map.of("refundId", refund.get("id"), "status", refund.get("status"));
}
```

---

## 11. Payment Status & Reconciliation

```
Payment can be in these states:

CREATED → AUTHORIZED → CAPTURED → SETTLED
                    ↘ FAILED
CAPTURED → REFUND_INITIATED → REFUNDED

State Machine:
  CREATED:    Order created, waiting for user to pay
  AUTHORIZED: Bank approved (card only — money on hold)
  CAPTURED:   Money deducted from customer
  SETTLED:    Money received in merchant's bank account (T+1 to T+3)
  FAILED:     Payment rejected (insufficient funds, wrong OTP, etc.)
  REFUNDED:   Money returned to customer
```

```java
// Daily Reconciliation Job (match your records with gateway records)
@Scheduled(cron = "0 0 6 * * *") // Run daily at 6 AM
public void dailyReconciliation() {
    
    RazorpayClient client = new RazorpayClient(keyId, keySecret);
    
    // Get all payments from yesterday
    LocalDate yesterday = LocalDate.now().minusDays(1);
    List<PaymentOrder> ourRecords = repo.findByDateAndStatus(yesterday, "CREATED");
    
    for (PaymentOrder order : ourRecords) {
        try {
            // Check with Razorpay
            Payment gatewayPayment = client.payments.fetch(order.getRazorpayPaymentId());
            String gatewayStatus = gatewayPayment.get("status");
            
            if ("captured".equals(gatewayStatus) && "CREATED".equals(order.getStatus())) {
                // MISMATCH: Razorpay says paid, we say unpaid
                // → Confirm the booking!
                order.setStatus("PAID");
                repo.save(order);
                bookingService.confirmBooking(order.getBookingId());
                alertService.sendMismatchAlert(order); // Alert the team
            }
            
        } catch (Exception e) {
            log.error("Reconciliation failed for order: " + order.getId(), e);
        }
    }
}
```

---

## 12. Wallet System (In-App Wallet)

```java
@Entity
public class Wallet {
    @Id @GeneratedValue
    private Long id;
    private Long userId;
    
    @Column(precision = 12, scale = 2)
    private BigDecimal balance;
    
    @Version  // Optimistic locking to prevent race conditions!
    private Long version;
}

@Entity
public class WalletTransaction {
    @Id @GeneratedValue
    private Long id;
    private Long walletId;
    
    @Enumerated(EnumType.STRING)
    private TransactionType type; // CREDIT, DEBIT
    
    private BigDecimal amount;
    private String description;
    private String referenceId; // For idempotency
    private LocalDateTime createdAt;
}

@Service
public class WalletService {
    
    @Transactional
    public void credit(Long userId, BigDecimal amount, String referenceId) {
        // Idempotency check
        if (txnRepo.existsByReferenceId(referenceId)) return;
        
        Wallet wallet = walletRepo.findByUserId(userId)
            .orElseThrow(() -> new RuntimeException("Wallet not found"));
        
        wallet.setBalance(wallet.getBalance().add(amount));
        walletRepo.save(wallet); // @Version handles concurrent updates
        
        // Record transaction
        WalletTransaction txn = new WalletTransaction();
        txn.setWalletId(wallet.getId());
        txn.setType(TransactionType.CREDIT);
        txn.setAmount(amount);
        txn.setReferenceId(referenceId);
        txn.setCreatedAt(LocalDateTime.now());
        txnRepo.save(txn);
    }
    
    @Transactional
    public void debit(Long userId, BigDecimal amount, String referenceId) {
        if (txnRepo.existsByReferenceId(referenceId)) return;
        
        Wallet wallet = walletRepo.findByUserId(userId)
            .orElseThrow(() -> new RuntimeException("Wallet not found"));
        
        if (wallet.getBalance().compareTo(amount) < 0) {
            throw new InsufficientBalanceException("Balance: " + wallet.getBalance());
        }
        
        wallet.setBalance(wallet.getBalance().subtract(amount));
        walletRepo.save(wallet);
        
        WalletTransaction txn = new WalletTransaction();
        txn.setWalletId(wallet.getId());
        txn.setType(TransactionType.DEBIT);
        txn.setAmount(amount);
        txn.setReferenceId(referenceId);
        txn.setCreatedAt(LocalDateTime.now());
        txnRepo.save(txn);
    }
    
    // Transfer between wallets (atomic!)
    @Transactional
    public void transfer(Long fromUserId, Long toUserId, BigDecimal amount) {
        String refId = UUID.randomUUID().toString();
        debit(fromUserId, amount, "debit_" + refId);
        credit(toUserId, amount, "credit_" + refId);
    }
}
```

---

## 13. Subscription / Recurring Payments

```java
@Entity
public class Subscription {
    @Id @GeneratedValue
    private Long id;
    private Long userId;
    private String planId;       // "monthly_premium", "annual_basic"
    private String razorpaySubscriptionId;
    
    @Enumerated(EnumType.STRING)
    private SubscriptionStatus status; // ACTIVE, PAUSED, CANCELLED, EXPIRED
    
    private LocalDateTime startDate;
    private LocalDateTime endDate;
    private LocalDateTime nextBillingDate;
}

// Razorpay Subscription API
@PostMapping("/subscribe")
public Map<String, Object> createSubscription(@RequestBody SubscriptionRequest request) throws Exception {
    
    RazorpayClient client = new RazorpayClient(keyId, keySecret);
    
    JSONObject options = new JSONObject();
    options.put("plan_id", request.getPlanId());       // Pre-created plan on Razorpay
    options.put("total_count", 12);                     // 12 billing cycles
    options.put("quantity", 1);
    
    JSONObject notifyInfo = new JSONObject();
    notifyInfo.put("notify_phone", request.getPhone());
    notifyInfo.put("notify_email", request.getEmail());
    options.put("notify_info", notifyInfo);
    
    Subscription sub = client.subscriptions.create(options);
    
    // Save to DB
    // Return short_url for user to authorize recurring payment
    return Map.of(
        "subscriptionId", sub.get("id"),
        "shortUrl", sub.get("short_url")    // User opens this to authorize
    );
}

// Webhook handles recurring charge events
// "subscription.charged" → renew access
// "subscription.cancelled" → revoke access
// "payment.failed" → retry or notify user
```

---

## 14. Security in Payments

```
┌─────────────────────────────────────────────────────────────┐
│                  PAYMENT SECURITY CHECKLIST                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. NEVER store full card numbers (PCI-DSS compliance)      │
│     → Use payment gateway's tokenization                    │
│                                                              │
│  2. ALWAYS verify payment server-side                       │
│     → Never trust frontend "payment success" callback       │
│     → Verify signature with gateway's secret key            │
│                                                              │
│  3. Use HTTPS everywhere                                     │
│     → All payment APIs must be over TLS/SSL                 │
│                                                              │
│  4. Webhook signature verification                           │
│     → Verify X-Razorpay-Signature header                    │
│     → Prevent fake webhook attacks                          │
│                                                              │
│  5. Idempotency keys for all payment APIs                   │
│     → Prevent double charging on retry                      │
│                                                              │
│  6. Amount validation on SERVER                              │
│     → Don't trust amount from frontend                      │
│     → Recalculate total from cart items server-side          │
│                                                              │
│  7. Rate limiting on payment endpoints                       │
│     → Prevent card testing attacks (brute force CVV)        │
│                                                              │
│  8. Audit logging for every payment event                   │
│     → Log: who, when, how much, status, IP address          │
│                                                              │
│  9. Environment-specific keys                                │
│     → Test keys for development, live keys for production   │
│     → NEVER commit API keys to Git!                         │
│                                                              │
│  10. PCI DSS Compliance                                      │
│      → Use hosted checkout (Razorpay/Stripe handles card UI)│
│      → Your server never sees raw card numbers              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 15. Database Schema for a Payment System

```sql
-- Core Tables

CREATE TABLE payment_orders (
    id              BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id        VARCHAR(50) UNIQUE NOT NULL,    -- Your internal order ID
    user_id         BIGINT NOT NULL,
    amount          DECIMAL(12,2) NOT NULL,
    currency        VARCHAR(3) DEFAULT 'INR',
    status          ENUM('CREATED','AUTHORIZED','CAPTURED','FAILED',
                         'REFUND_INITIATED','REFUNDED') DEFAULT 'CREATED',
    
    -- Gateway fields
    gateway_name    VARCHAR(20),                     -- 'razorpay', 'stripe'
    gateway_order_id VARCHAR(100),                   -- Razorpay's order ID
    gateway_payment_id VARCHAR(100),                 -- Razorpay's payment ID
    gateway_signature VARCHAR(255),
    
    -- Metadata
    payment_method  VARCHAR(20),                     -- 'upi', 'card', 'netbanking'
    description     VARCHAR(255),
    idempotency_key VARCHAR(100) UNIQUE,
    
    -- Timestamps
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    paid_at         TIMESTAMP NULL,
    
    INDEX idx_user_id (user_id),
    INDEX idx_status (status),
    INDEX idx_gateway_order (gateway_order_id)
);

CREATE TABLE payment_events (
    id              BIGINT PRIMARY KEY AUTO_INCREMENT,
    payment_order_id BIGINT NOT NULL,
    event_type      VARCHAR(50) NOT NULL,            -- 'CREATED','CAPTURED','FAILED','REFUNDED'
    event_data      JSON,                            -- Raw webhook payload
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (payment_order_id) REFERENCES payment_orders(id),
    INDEX idx_payment_order (payment_order_id)
);

CREATE TABLE refunds (
    id              BIGINT PRIMARY KEY AUTO_INCREMENT,
    payment_order_id BIGINT NOT NULL,
    refund_id       VARCHAR(100),                    -- Gateway's refund ID
    amount          DECIMAL(12,2) NOT NULL,
    reason          VARCHAR(255),
    status          ENUM('INITIATED','PROCESSED','FAILED') DEFAULT 'INITIATED',
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    processed_at    TIMESTAMP NULL,
    
    FOREIGN KEY (payment_order_id) REFERENCES payment_orders(id)
);
```

---

## 16. Complete Spring Boot Payment Service

```java
// ═══ Payment Service (Orchestrates Everything) ═══

@Service
@Transactional
public class PaymentService {
    
    @Autowired private PaymentOrderRepository orderRepo;
    @Autowired private PaymentEventRepository eventRepo;
    @Autowired private StringRedisTemplate redis;
    @Autowired private RabbitTemplate rabbitTemplate;
    
    private final RazorpayClient razorpay;
    
    public PaymentService(@Value("${razorpay.key.id}") String keyId,
                          @Value("${razorpay.key.secret}") String secret) throws Exception {
        this.razorpay = new RazorpayClient(keyId, secret);
    }
    
    // ═══ CREATE ═══
    public PaymentOrder createPayment(Long userId, BigDecimal amount, String description) {
        JSONObject options = new JSONObject();
        options.put("amount", amount.multiply(BigDecimal.valueOf(100)).intValue());
        options.put("currency", "INR");
        options.put("receipt", "rcpt_" + System.currentTimeMillis());
        
        try {
            Order rzpOrder = razorpay.orders.create(options);
            
            PaymentOrder order = new PaymentOrder();
            order.setOrderId("ORD_" + System.currentTimeMillis());
            order.setUserId(userId);
            order.setAmount(amount);
            order.setGatewayName("razorpay");
            order.setGatewayOrderId(rzpOrder.get("id"));
            order.setStatus(PaymentStatus.CREATED);
            order.setDescription(description);
            
            return orderRepo.save(order);
        } catch (RazorpayException e) {
            throw new PaymentException("Failed to create order", e);
        }
    }
    
    // ═══ VERIFY ═══
    public PaymentOrder verifyAndCapture(String gatewayOrderId, String paymentId, String signature) {
        // Verify signature
        String data = gatewayOrderId + "|" + paymentId;
        if (!verifySignature(data, signature)) {
            throw new PaymentException("Invalid payment signature!");
        }
        
        PaymentOrder order = orderRepo.findByGatewayOrderId(gatewayOrderId)
            .orElseThrow(() -> new PaymentException("Order not found"));
        
        // Idempotency: already processed?
        if (order.getStatus() == PaymentStatus.CAPTURED) return order;
        
        order.setStatus(PaymentStatus.CAPTURED);
        order.setGatewayPaymentId(paymentId);
        order.setPaidAt(LocalDateTime.now());
        orderRepo.save(order);
        
        // Record event
        saveEvent(order.getId(), "PAYMENT_CAPTURED", paymentId);
        
        // Notify other services
        rabbitTemplate.convertAndSend("payment-events", "payment.captured",
            Map.of("orderId", order.getOrderId(), "userId", order.getUserId(), 
                   "amount", order.getAmount()));
        
        return order;
    }
    
    // ═══ STATUS CHECK (for reconciliation) ═══
    public String checkGatewayStatus(String paymentId) {
        try {
            Payment payment = razorpay.payments.fetch(paymentId);
            return payment.get("status"); // "created", "authorized", "captured", "failed"
        } catch (RazorpayException e) {
            return "unknown";
        }
    }
    
    // ═══ REFUND ═══
    public void initiateRefund(String orderId, BigDecimal amount, String reason) {
        PaymentOrder order = orderRepo.findByOrderId(orderId)
            .orElseThrow(() -> new PaymentException("Order not found"));
        
        JSONObject options = new JSONObject();
        options.put("amount", amount.multiply(BigDecimal.valueOf(100)).intValue());
        
        try {
            Refund refund = razorpay.payments.refund(order.getGatewayPaymentId(), options);
            
            order.setStatus(PaymentStatus.REFUND_INITIATED);
            orderRepo.save(order);
            
            saveEvent(order.getId(), "REFUND_INITIATED", refund.get("id"));
        } catch (RazorpayException e) {
            throw new PaymentException("Refund failed", e);
        }
    }
}
```

---

## 17. Interview Questions (20+)

| # | Question | Key Answer |
|:---:|:---|:---|
| 1 | How does UPI work internally? | App → PSP → NPCI → Debit Bank → Credit Bank. VPA resolves via NPCI. |
| 2 | How does QR payment work? | QR = encoded UPI intent URL. Static QR (no amount), Dynamic QR (pre-filled amount + txn ID). |
| 3 | What happens when payment succeeds but your server crashes? | Webhook from gateway confirms payment. Reconciliation cron as backup. |
| 4 | How to prevent double payment? | Idempotency key in header + Redis cache of processed requests. |
| 5 | Auth vs Capture? | Auth = place hold on money. Capture = actually deduct. Used for hotels, e-commerce. |
| 6 | How BookMyShow locks seats? | Redis SETNX with 8-min TTL. Seats auto-release on timeout. |
| 7 | What is PCI-DSS? | Security standard for card data. Use hosted checkout to avoid handling raw card numbers. |
| 8 | Webhook vs Polling? | Webhook: gateway pushes to you (real-time). Polling: you pull status (fallback). |
| 9 | How to handle partial refund? | Send refund amount < original to gateway API. Track partial amounts in DB. |
| 10 | What is 3D Secure / OTP? | Additional auth layer for online card payments. Bank sends OTP. RBI mandated in India. |
| 11 | How does wallet system work? | DB table with balance + @Version (optimistic lock). Every credit/debit = transaction record. |
| 12 | How to reconcile payments? | Daily cron: compare your DB status with gateway API. Alert on mismatches. |
| 13 | What is a payment gateway vs processor? | Gateway: merchant-facing API (Razorpay). Processor: routes to card networks (Visa/NPCI). |
| 14 | How to handle payment timeout? | Keep in PENDING. Background job checks gateway. Confirm or release after 30 mins. |
| 15 | What are subscription payments? | Razorpay stores mandate. Auto-charges on billing cycle. Webhooks for success/failure. |
| 16 | How does Paytm Wallet work? | Pre-loaded balance in Paytm's own bank. Debit = internal ledger update (instant). |
| 17 | What is settlement? | When gateway transfers captured money to merchant's bank. Usually T+1 to T+3. |
| 18 | How to design split payments? | Marketplace: customer pays Flipkart, Flipkart splits to seller + commission. Route API on gateway. |
| 19 | What is a dead letter queue in payments? | Failed payment events go to DLQ. Manual review → retry or refund. |
| 20 | How to handle concurrent booking? | `SELECT FOR UPDATE` (pessimistic lock) OR Redis SETNX (distributed lock) for seat/inventory. |
