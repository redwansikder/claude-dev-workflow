# Finding Your Phases: Example 3

A third worked example of the method in [Finding Your Phases](finding-phases-1.md). This one adds a new idea: **replace a phase with a manual workaround**, so the shop can start selling sooner.

---

## Example 3: Online clothing shop (Bangladesh)

### Step 1: List what users must be able to do

Write it all down, in any order:

**Customer**
- browse products by category (shirts, panjabi, saree)
- search products
- see a product page (photos, price, sizes)
- add to cart
- check out with an address and phone number
- pay with bKash / card
- pay cash on delivery (COD)
- track their order
- see past orders
- apply a discount coupon
- leave a product review
- add to wishlist
- sign up and log in

**Admin (shop owner)**
- add and edit products (photos, price, sizes, stock)
- see new orders
- change order status (confirmed → shipped → delivered)
- send orders to a courier (Pathao / Steadfast)
- create coupons
- see a sales report

### Step 2: Find the final thing

Ask: **what does the shop owner actually need?**

> A customer **places an order** that the owner can see and deliver.

Not reviews, not wishlists. An order coming in is what makes money.

### Step 3: Work backwards from it

```
Customer places an order
   ↑ needs: checkout (address, phone)
      ↑ needs: payment?                       ← check this
         ↑ needs: cart
            ↑ needs: product page
               ↑ needs: products exist
                  ↑ needs: admin adds products
                     ↑ needs: login? + database + image storage   ← check this too
```

### Step 3b: Check each link, including manual workarounds

[Example 2](finding-phases-2.md) asked "can this wait?" This time also ask **"can a person do this by hand for now?"**

| Link | Really needed first? | Workaround until it's built |
|---|---|---|
| Online payment (bKash/card) | **No** | **Cash on delivery.** Most Bangladeshi online shoppers use it anyway. |
| Customer login | **No** | **Guest checkout:** name, phone and address are enough to deliver. |
| Order status / tracking | **No** | The owner **calls or texts the customer** on WhatsApp. |
| Courier integration | **No** | The owner **books the courier by hand** in the Pathao/Steadfast app. |
| Admin order page | **Yes**, the owner must see orders | Even a simple list is enough. |
| Cart | Yes | Customers often buy 2–3 items. |

The short chain is now:

```
Customer places an order (COD, guest)
   ↑ checkout form (name, phone, address)
      ↑ cart
         ↑ product page
            ↑ admin adds products
               ↑ database + image storage + admin login
```

Customer login is gone. Only the admin needs to log in.

### Step 4: Read it in reverse and group it into phases

```
Phase 0: Setup              → database, image storage, admin login, deploy
Phase 1: Products           → admin adds/edits products, stock, sizes
Phase 2: Storefront         → category pages, product page, cart
Phase 3: Orders             → guest checkout (COD), admin sees orders   ← the final thing
```

After Phase 3 **the shop is open**. The owner can share product links on Facebook and take real COD orders, then handle delivery by hand.

### Step 5: Put the leftovers after the final thing

Look at what the owner finds most painful when doing things by hand, and automate that first:

| Leftover | Needs first | Pain if it's left manual |
|---|---|---|
| bKash / card payment | orders (Phase 3) | Medium: COD works, but some orders get refused at the door |
| Order status + SMS | orders | High: lots of "where is my order?" calls |
| Courier integration | order status | High at 20+ orders a day |
| Customer accounts + past orders | orders | Low |
| Search | products | Low with under 100 products |
| Coupons | checkout | Low, can wait for a sale |
| Reviews, wishlist | accounts | Low |
| Sales report | orders | Medium: the owner can count in Excel for now |

Order the later phases by **pain**, not by what's fun to build:

```
Phase 4: Order status + SMS updates
Phase 5: Online payment (bKash, card)
Phase 6: Courier integration
Phase 7: Customer accounts + order history
Phase 8: Search, coupons, sales report
Phase 9: Reviews, wishlist
```

### The final Roadmap

```md
## Roadmap
- [ ] Phase 0: Setup (DB, image storage, admin login, deploy)
- [ ] Phase 1: Admin adds products
- [ ] Phase 2: Storefront (categories, product page, cart)
- [ ] Phase 3: Guest checkout (COD) + admin order list   ← shop opens here
- [ ] Phase 4: Order status + SMS
- [ ] Phase 5: bKash / card payment
- [ ] Phase 6: Courier integration
- [ ] Phase 7: Customer accounts + order history
- [ ] Phase 8: Search, coupons, sales report
- [ ] Phase 9: Reviews, wishlist
```

---

## What this example adds

- **Ask "can a person do this by hand for now?"** COD replaces payments, WhatsApp replaces tracking, and the courier app replaces integration. The shop opens three phases earlier.
- **Not every user type needs login.** Customers check out as guests, and only the admin logs in.
- **Order the leftovers by pain.** Automate whatever costs the owner the most time each day. Wishlists come last because nobody's day is ruined without one.
- **Real usage changes the Roadmap.** If customers keep asking for bKash in Phase 4, move payments up. The Roadmap is a guess, and real orders correct it.

**Next:** [Example 4: Bus ticket booking](finding-phases-4.md) shows what to do when a link can't be removed.
