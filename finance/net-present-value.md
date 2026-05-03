# A Beginner's Guide to NPV & Discounting

*The one financial concept that explains how stocks, startups, and investments are really valued.*


There's a question that sits at the heart of almost every financial decision ever made:

> **"Is what I'm paying today worth what I'll get in the future?"**

Whether you're buying stocks, evaluating a business, or deciding whether to fund a startup, this question comes up every single time. And the tools that answer it — **discounting** and **Net Present Value (NPV)** — are surprisingly simple once you strip away the jargon.

Let's break it down from scratch.

> **📝 Note:** This post was written with the assistance of AI. The content reflects my personal learning and understanding and should not be taken as professional advice. Please do your own research and due diligence before acting on anything written here.

## 🍫 Start With Chocolate

Forget money for a moment. Imagine I offer you a choice:

- **Option A:** 1 chocolate *right now*
- **Option B:** 1 chocolate *next year*

You'd pick Option A. Obviously.

Not because you're impatient (well, maybe a little), but because:
- You can enjoy it *now*
- You don't have to wait and wonder if I'll actually deliver
- You could even trade it for something else today

This intuition — that things today are worth more than the same things later — is the entire foundation of discounting.


## 💡 So What Is Discounting?

Discounting is just a formal way of asking:

> *"What is a future amount of money worth to me today?"*

If someone promises you ₹100 a year from now, you'd naturally value it at *less* than ₹100 today. How much less? That depends on two things:

1. **How long you have to wait** — the longer the wait, the less it's worth today
2. **How risky it is** — the less certain the payment, the more you discount it

The math looks like this:

$$PV = \frac{FV}{(1+r)^t}$$

Where:
- **PV** = Present Value (what it's worth today)
- **FV** = Future Value (what you'll receive later)
- **r** = your discount rate (required return)
- **t** = time in years

So if someone promises you ₹110 next year and you want a 10% return:

$$PV = \frac{110}{(1.10)^1} = ₹100$$

You'd be willing to pay up to ₹100 today for that promise.


## 🔁 Discounting vs. Compounding

You've probably heard of compounding — the idea that money grows over time. Discounting is just the reverse.

| Direction | Process | Example |
|---|---|---|
| Forward in time | Compounding | ₹80 × 1.25 = ₹100 |
| Backward in time | Discounting | ₹100 ÷ 1.25 = ₹80 |

Same relationship. Opposite direction.

## 🧮 Now, What Is NPV?

Net Present Value takes discounting and applies it to an *entire project or investment* — one with multiple cash flows over several years.

The formula:

$$NPV = \sum_{t=0}^{n} \frac{CF_t}{(1+r)^t}$$

Translation: discount every cash flow back to today's value, then add them all up.

### The Decision Rule Is Simple

| Result | Meaning |
|---|---|
| NPV > 0 ✅ | Invest — it creates value |
| NPV < 0 ❌ | Reject — it destroys value |
| NPV = 0 ⚖️ | Break even — you're indifferent |


## 🍎 A Real-World Example: Buying Apple Stock

Let's say you believe Apple will deliver value equivalent to **$100 per share** over the next year (through growth or dividends). The stock is currently trading at $85.

You want a **10% annual return**. So you discount:

$$PV = \frac{100}{1.10} = \$91$$

The stock is worth $91 to you today. It's trading at $85.

**That's a good deal.** You're getting $91 worth of future value for $85.

If it were trading at $110? Walk away.

This is exactly how professional investors think — not "is this a good company?" but "are the future cash flows worth more than the price I'm paying today?"


## 🚀 And a Startup Example

A founder approaches you:

> "Invest ₹1,00,000 today. I'll return ₹2,00,000 in 5 years."

Sounds appealing. But you factor in the risk and decide you need a **25% return**.

$$PV = \frac{2{,}00{,}000}{(1.25)^5} \approx ₹65{,}000$$

The future ₹2 lakhs is only worth about ₹65,000 to you *today*. They're asking for ₹1 lakh.

**The deal doesn't work.** You'd be overpaying for the future cash flow.


## 🎯 The Discount Rate — Where Does It Come From?

Your discount rate is simply your **required return**: the minimum reward you demand for taking on risk and waiting.

Think of it as three layers stacked together:

### 1. Risk-Free Rate (~6–7% in India)
What you'd earn by doing nothing risky — like putting money in government bonds.

### 2. Equity Risk Premium (~7–9%)
Extra return demanded for investing in stocks instead of safe assets.

### 3. Company-Specific Risk
Added on top based on the investment's uncertainty:

- Stable blue-chip company → **+0–2%**
- Growing tech company → **+2–5%**
- Early-stage startup → **+5–15%**

### In Practice

| Investment Type | Typical Discount Rate |
|---|---|
| Stable large-cap (e.g., Infosys, Apple) | 8–10% |
| Growing mid-cap | 10–15% |
| Risky startup | 15–25%+ |

## 🧠 The Key Insight Most People Miss

Your **discount rate** and your **required return** are the same number, just viewed from different directions.

- Required return: "I want 12% from this investment going forward"
- Discount rate: "I'll value future cash flows by dividing by 1.12 each year"

They meet in the middle. When you say "I use a 12% discount rate," you're really saying: *"I only invest if I can earn at least 12% per year."*

## 🔥 The Pro Mindset Shift

Most beginners ask: *"Is this a good company?"*

Investors who consistently beat the market ask: *"Are this company's future cash flows worth more than its current price — after accounting for risk?"*

That's it. That's the whole game.

Every stock price, every startup valuation, every bond price — they're all just the market's best guess at the present value of future cash flows. When you think a company's future is being undervalued by the market, you buy. When it's overvalued, you don't.

NPV and discounting give you the framework to make that call with logic, not gut feeling.

## 🔍 But Wait — How Do You Actually Estimate Future Cash Flows?

This is the question most guides skip. And it's a fair one, because all of this math only works if you have cash flow numbers to plug in. Where do those come from?

The honest answer: **you're always estimating.** There's no formula that magically produces future cash flows. But you're not guessing blindly either. Analysts build estimates from three sources:

### 1. 📊 Historical Performance

Start with what the company has *actually done*. Look at:
- Revenue growth over the last 5–10 years
- Profit margins (are they stable? expanding? shrinking?)
- Free cash flow trends

If a company has grown revenue consistently at 12% a year for a decade, that's a reasonable baseline to project forward — unless something fundamental has changed.

### 2. 🏢 Business Fundamentals

Numbers alone don't tell the full story. You also need to understand the *business itself*:

- **Pricing power** — Can it raise prices without losing customers? (Think: a utility vs. a commodity seller)
- **Recurring revenue** — Subscription businesses are far more predictable than one-time sales
- **Industry trajectory** — Is the market growing, shrinking, or being disrupted?
- **Competitive moat** — Does it have something competitors can't easily copy?

This is why Warren Buffett only invests in businesses he can *understand*. A company selling fizzy drinks has more predictable cash flows than a biotech startup waiting on FDA approval. Simpler business = more reliable estimate.

### 3. 🔭 Forward-Looking Assumptions

Finally, you layer in judgment about the future:

- **Management guidance** — What does the company itself say about next year?
- **Analyst reports** — What are professional researchers projecting?
- **Macro factors** — Interest rates, regulation, consumer trends
- **Industry growth rates** — If the market is growing at 8%, can this company keep up?

### The Key Mindset

Don't aim for precision — aim for **reasonable ranges**. Instead of projecting exactly ₹500 crore in cash flow next year, ask: "Is it likely to be between ₹400–600 crore?" Then stress-test your NPV at both ends.

| Company Type | Forecast Confidence | Why |
|---|---|---|
| Utility / FMCG | High ✅ | Stable demand, predictable pricing |
| Established tech | Medium 🟡 | Growth visible but competitive |
| Startup / Biotech | Low ⚠️ | Binary outcomes, high uncertainty |

The harder a business is to forecast, the higher your discount rate should be — which is really just another way of saying: *"I'm less certain, so I need a bigger margin of safety."*

## ✅ The Takeaways

1. **Money today > money tomorrow** — always, because of time, risk, and opportunity cost
2. **Discounting** converts future money into today's equivalent value
3. **NPV** sums all discounted cash flows to tell you if an investment is worth it
4. **Your discount rate = your required return** — set it based on the risk you're taking
5. Every investment decision boils down to one question: *"Is the present value of future cash flows greater than what I'm paying today?"*
6. **Future cash flows are estimated, not known** — use history, business fundamentals, and judgment; the simpler the business, the more reliable your estimate

*Once you internalize these ideas, you'll never look at a price tag — on a stock, a startup, or any asset — the same way again.*