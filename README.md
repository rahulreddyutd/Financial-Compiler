# Obin AI Financial Compiler

I built this to show what Obin's Financial Compiler concept looks like as a working product.

If you've read Obin's founding story, you know the core argument: the gap between AI potential and production-grade deployment in finance isn't about model capability, it's about the validation layer underneath. Most AI vendors skip that layer. Obin didn't. They called it the Financial Compiler, a deterministic engine that checks agent outputs against accounting identities and covenant definitions before a human ever sees the result.

This is my attempt to build that.

---

## What it does

You pick a loan scenario, the AI agent reads the document, extracts every financial covenant, and computes compliance ratios. Then a second pass runs the compiler, which checks the agent's own work for definition errors, missing components, and edge cases that would produce a wrong answer in a real portfolio.

The output includes a full audit trail with timestamps, a per-covenant breakdown, and a portfolio-level verdict.

Three scenarios are included out of the box:

- **Direct Lending** (Apex Manufacturing): standard senior secured structure with net leverage, interest coverage, CapEx cap, and DSCR covenants
- **Unitranche / Software** (Northbridge Software): SaaS-specific covenants including ARR floor, Rule of 40, and an acquisition step-up provision that makes the leverage threshold dynamic
- **Mezzanine / Healthcare** (Summit Healthcare): PIK structure with a cross-reference covenant that depends on a separate agreement, a gross PIK accrual cap with no reset, and a DSCR tested on a semi-annual annualized basis

Each scenario was designed around a real failure mode. The mezzanine doc, for example, has a covenant whose threshold literally lives in another document. The compiler flags that as an external dependency and notes that the mezzanine lender has no visibility if the senior lender quietly amends their terms. That's not a hypothetical edge case. That's something that bites people.

---

## How it works

Two Claude API calls run sequentially.

The first sends the loan document to Claude acting as a financial analyst. It extracts covenant definitions, computes ratios using the financials in the document, and flags anything that looks like an edge case or requires data that isn't present.

The second sends Claude's own output back to Claude acting as the compiler. The compiler checks for definition mismatches (gross debt vs net debt, GAAP vs adjusted EBITDA, total revenue vs ARR), arithmetic errors, missing components like CapEx in a DSCR numerator, and dynamic threshold provisions like step-ups and carryforwards.

The two-pass structure is the point. The agent might get something wrong. The compiler is supposed to catch it before it reaches a human. If the compiler corrects a verdict, it shows up in the output with a clear explanation of what the agent got wrong and what the correct answer is.

---

## What I was thinking when I built this

I spent four years building AI systems inside regulated financial institutions, Capital One and Optum. The failure mode I kept running into wasn't model quality. It was that nobody had built a principled validation layer between the model output and the business decision.

A model tells you a borrower is compliant. The analyst trusts it. Nobody checks whether the model used gross debt instead of net debt, or forgot to net out the revolver availability from the liquidity covenant. Those aren't hypothetical mistakes. They happen, and in credit monitoring, they're the kind of mistake that surfaces three quarters too late.

Obin's framing of this as a "compiler" is exactly right. You don't ship code without a compiler. You shouldn't ship AI-generated financial outputs without one either.

This demo is my version of showing up with a working prototype rather than a cover letter.

---

## Technical notes

Built as a single self-contained HTML file with no dependencies or build step. Uses the Anthropic Messages API with assistant prefill to enforce structured JSON output from both the agent and compiler calls. The fallback layer means the UI degrades gracefully if either call returns something unparseable.

Total API cost per run is roughly $0.02 to $0.04 depending on document length.
