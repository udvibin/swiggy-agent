# swiggy-agent

**You're hungry. You open Swiggy. 20 minutes later you're still scrolling and nothing sounds good.**

Then you settle for the same butter chicken you've ordered 47 times, or worse, Domino's. Not because you wanted Domino's, but because you gave up.

This agent fixes that.

## The Problem

Swiggy has thousands of restaurants and nobody knows what they want to eat. The app gives you infinite choice, which sounds great in theory but in practice leads to:

- **"Nothing sounds good"** even though you're starving
- **"Too many things sound good"** and you can't pick
- **"I found what I want but it's too expensive without a coupon"** so you spiral into discount hunting
- **"I spent 15 minutes building a cart that doesn't qualify for the coupon"** so you start over

On top of this, some food just doesn't travel well. You pay ₹700 for a fancy pizza, it arrives lukewarm and soggy, and now you're broke AND unsatisfied. Domino's wins again.

## What This Agent Does

**It decides for you.** One recommendation. Not a list. Not 10 options. One.

The agent knows what you like, what you've had recently, what time it is, and most importantly, what's on offer right now. It opens with a single recommendation that's taste-matched, cost-optimized, and ready to order.

### The "Just Pick For Me" Flow

You open the agent. It already has a suggestion:

> "Szechuan noodles from Wok Street. You haven't had Chinese in 11 days, they've got ₹120 off above ₹399, and they're 18 mins away. Noodles + chilli chicken = ₹430, post-discount ₹310. Want this?"

Yes? Done. Order placed. Total time: 30 seconds.

No? It doesn't ask "then what do you want" (that restarts the paralysis). It makes another lateral suggestion in a different cuisine, still with a deal attached.

### The Cart Optimizer

This is where it saves you real money and time. Say a restaurant has "₹120 off above ₹499" and your items total ₹430. Instead of you manually scanning the menu for filler items, the agent:

1. Finds the cheapest item that crosses the threshold
2. Makes sure it actually complements your order (a drink, a side, not random dessert)
3. Shows you the math: "Add masala lemonade for ₹59, total ₹489... still short. Add garlic bread for ₹99, total ₹529, post-coupon ₹409. Three items for less than you'd pay for two."

It also compares across coupon types. General coupon gives ₹120 off but your credit card offer gives 20% up to ₹150? It picks whichever saves more.

### The Anti-Repeat Engine

Tracks your recent orders and actively steers you away from the rut. Not just "you ordered biryani a lot" but more like:

- You've only ordered from 4 restaurants in the last 3 weeks
- Your cuisine diversity is 3/10
- Here's a 4.3-rated Thai place you've never tried, with a BOGO, 15 mins away

Has two modes: **comfort** (familiar food, optimized for deals) and **explore** (new restaurants and cuisines, still deal-aware).

### Vibe-Based Ordering

Don't know what you want? Describe the vibe:

- "something heavy and comforting" → butter chicken + garlic naan from your top-rated nearby place
- "light, not too expensive" → poke bowl under ₹350
- "hungover" → you know what you need

The agent maps vibes to flavors to actual available dishes with active offers.

### The Delivery Reality Check

Some food travels great (biryani, curries, wraps). Some doesn't (pizza, fries, anything crispy). When you're about to drop ₹700 on something that'll arrive soggy from 40 mins away, the agent flags it and suggests alternatives that deliver better from closer. Not a hard block, just honest context.

## Future: Dineout Deal Nudger

A separate mode that monitors Swiggy Dineout for deals and available slots at restaurants matching your preferences. Instead of you actively searching for dine-in offers, it proactively surfaces them:

> "Saturday evening free? That BBQ place you liked has 30% off and a 7:30 slot for 4. Want me to book?"

Low-effort way to actually use Dineout instead of forgetting it exists.

---

## How It Works (Technical)

### Architecture

```
User (Telegram bot / React chat)
        │
        ▼
   LLM Layer
   - Parses natural language intent
   - Resolves cravings, constraints, budget
   - Chains MCP tool calls
        │
        ▼
   ┌────────────────────────────┐
   │  Swiggy MCP Servers        │
   │                            │
   │  Food:                     │
   │    search_restaurants      │
   │    search_menu             │
   │    get_restaurant_menu     │
   │    update_food_cart        │
   │    get_food_cart           │
   │    place_food_order        │
   │    track_food_order        │
   │                            │
   │  Dineout:                  │
   │    search_restaurants      │
   │    get_restaurant_details  │
   │    get_available_slots     │
   │    book_table              │
   │    get_booking_status      │
   └────────────────────────────┘
        │
        ▼
   Local Services (Python)
   - Taste Profile: order history + basic scoring
   - Cart Optimizer: coupon logic + threshold solver
   - Anti-Repeat: recency + diversity tracking
```

### Taste Profile (built passively, no onboarding)

Learned from order history over time:
- Cuisine frequency and recency
- Price comfort zone by day type (weekday vs weekend)
- Time-of-day patterns (weekday lunch is different from Saturday dinner)
- Reorder rate per restaurant (proxy for satisfaction)
- Order sequences (heavy meal followed by lighter next day)

### Cart Optimizer (pure logic, no AI needed)

- Input: selected items + all applicable coupons
- Checks every coupon's minimum threshold
- If short, scans menu for cheapest item to cross the threshold
- Compares general coupons vs credit card offers
- Output: best coupon + suggested add-on + net savings

### Conversational Layer (where AI earns its place)

The LLM handles things rule-based systems can't:
- "Something like last Friday but cheaper" → look up last Friday's order, find alternatives below that price with active offers
- "Cheesy but not pizza" → semantic understanding across menus
- "Not feeling Chinese" → negative signal for this time window, pivot to next best option
- Mapping vague vibes to concrete dish attributes

### Build Phases

| Phase | What | Complexity |
|-------|------|-----------|
| 1 | Cart Optimizer | Weekend project. Pure math, no AI. Immediate value. |
| 2 | Offer-First Discovery | Data aggregation + taste profile filtering. Light AI. |
| 3 | Conversational Agent | NL intent → MCP tool chains → single recommendation. This is the demo. |
| 4 | Passive Learning Loop | Accept/reject signals refine the taste profile over time. |

### Stack

- **LLM**: Any LLM api good enough for the application
- **Data Layer**: Swiggy MCP servers (Food + Dineout)
- **Backend**: Python (taste profile, cart optimizer, anti-repeat logic)
- **Frontend**: Telegram bot or React chat UI (TBD)

---
