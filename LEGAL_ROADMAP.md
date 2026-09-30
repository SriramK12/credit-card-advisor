# Legal, Regulatory & Compliance Roadmap

**Purpose:** a structured map of the legal/regulatory obstacles between "working prototype" and "real company handling real people's financial data," and realistic paths through each one.

**This document is not legal advice.** It's a research map built to make your first conversations with a real fintech/consumer-finance attorney productive instead of starting from zero. Every item below needs review by counsel licensed in the relevant jurisdiction before you act on it. Some of this (CROA, state CSO laws in particular) carries real personal and corporate liability if gotten wrong — this is not an area to guess in.

---

## 0. The top-line reality check

Three things make this space harder than typical consumer-app regulation:

1. **You're adjacent to credit reporting, credit repair, and lending, without being any of the three** — which paradoxically means you might get regulated as if you were, depending on how features are worded and monetized.
2. **SSNs and full credit reports are among the most sensitive data categories in US law** — the liability profile is closer to healthcare data than typical consumer-app data.
3. **The thing that makes Strata good (a scoring engine that steers people toward specific cards) is exactly the thing fair-lending law watches most closely.**

None of this is disqualifying. Credit Karma, Credit Sesame, NerdWallet, and Experian's own consumer app all operate in this exact space today. It means the path to "real company" runs through specific, known steps rather than being architecture you can just ship.

---

## 1. Credit Bureau Access — Soft Pull vs. Hard Pull

**The obstacle:** Equifax, Experian, and TransUnion don't hand out direct API access to a new company. Direct bureau relationships require passing security audits, demonstrating a documented "permissible purpose" under FCRA, and clearing minimum volume/vetting bars that a pre-revenue company won't clear.

**The mechanics you need to know:**
- A **soft inquiry** doesn't affect the consumer's score and is what educational/monitoring apps use. It requires consumer authorization but is a well-supported, well-trodden use case — this is literally Credit Karma's and Credit Sesame's entire data-access model.
- A **hard inquiry** happens specifically when a lender/issuer evaluates an actual credit application. Strata should never be the one performing a hard pull — that only happens when the user clicks through to the issuer's own site and applies there, which is already how the MVP is architected (a deliberate design choice, not an accident).

**Realistic path:** Don't approach Equifax/Experian/TransUnion directly first. Go through an established credit-data-as-a-service vendor who already holds bureau relationships and packages soft-pull access via API for exactly this use case:
- **Array** (getarray.io) — purpose-built white-label credit score/report widgets for fintech apps
- **Bloom Credit**
- **MeridianLink / CRS Credit API**
- **Experian Connect** (Experian's own developer/partner program)

These vendors typically host the actual identity-verification and bureau-pull flow themselves (see Section 4 on why that matters for SSN handling), and their onboarding process already walks you through the permissible-purpose documentation FCRA requires. This is a partnership/contract negotiation, not a from-scratch legal framework you build yourself.

---

## 2. FCRA (Fair Credit Reporting Act) Compliance

**The obstacle:** Pulling or displaying any bureau data makes Strata a "user of consumer reports" under FCRA, which comes with obligations around permissible purpose, consumer consent, and accuracy/dispute handling.

**Key nuance for your product specifically:** Because Strata doesn't make lending decisions (issuers do), you're less likely to trigger FCRA's "adverse action notice" requirements — those apply to whoever actually denies credit. But this protection only holds if Strata's messaging is airtight about never functioning as the decision-maker. The moment marketing or UI copy implies Strata is assessing approval odds rather than "positioning," you risk being treated as a de facto credit-decisioning system.

**How to overcome:** This is mostly already handled by the product's existing design principle (never state or imply guaranteed approval, readiness is "positioning" not prediction) — the job going forward is keeping that discipline as features and monetization get added, and having counsel review the actual bureau-data consent flow once one exists.

---

## 3. Credit Repair Organizations Act (CROA) & State "Credit Services Organization" Laws

**This is the one most people building in this space underestimate.**

**The obstacle:** The federal CROA applies to anyone who, for compensation, provides services to improve a consumer's credit record, history, or rating. Features like "Improve Your Profile" and the general framing of "we'll help you build credit" are exactly the kind of language CROA was written to catch, once money changes hands (subscription fees, referral fees).

CROA requires, among other things:
- A specific written contract with mandated disclosure language
- A 3-business-day right to cancel
- A **prohibition on collecting any fee before the service is fully performed**
- A ban on certain guarantee-style claims

**Many states go further.** Roughly half the states (Texas included, worth noting given where you're based) have their own **Credit Services Organization (CSO)** statutes that mirror or extend CROA, sometimes requiring a **surety bond** and **state registration** before you can legally operate — even before you charge a dime in some states.

**How to overcome:**
- Structure monetization around "personalized strategy" and "recommendations," not "credit repair" or "credit improvement services" — the difference in how a feature is marketed and billed can be the difference between triggering CROA and not
- Get counsel to review every piece of marketing copy and in-app language against CROA's specific trigger language before monetizing anything
- If any offering does end up classified as credit repair, budget for state-by-state CSO registration and bonding before operating in those states — this is a real, non-trivial cost and timeline item, not paperwork you can skip

---

## 4. SSN & Sensitive PII Handling

**The obstacle:** Pulling a real credit file requires identity verification, typically full legal name, date of birth, current address, and full SSN (or last-4 plus knowledge-based authentication questions). This is about the most sensitive data category a consumer app can touch.

**The single best mitigation:** Never let Strata's own servers directly receive or store the raw SSN if you can avoid it. The credit-data vendors in Section 1 (Array, Bloom Credit, etc.) typically offer a **hosted/tokenized flow** — the user enters their SSN directly into the vendor's secure widget, and Strata only ever receives a token and the pull result back. This is the same principle as Stripe Elements keeping raw card numbers off a merchant's server: it collapses your compliance scope dramatically and limits your breach liability to "we lost a token" instead of "we lost SSNs."

**If SSN ever has to touch your own infrastructure anyway:**
- Encrypt at rest (AES-256) and in transit (TLS 1.2+)
- Tokenize/vault it rather than storing it in application databases directly
- Strict, logged, role-based access controls
- Never let it reach logs, analytics events, or error-tracking tools, even accidentally — this is one of the most common real-world breach causes
- Treat this under the GLBA Safeguards Rule requirements below, not as a one-off engineering decision

---

## 5. GLBA (Gramm-Leach-Bliley Act) & the Safeguards Rule

**The obstacle:** A company handling nonpublic personal financial information is very likely to qualify as a "financial institution" under GLBA's broad definition, which means two things apply: the **Privacy Rule** (notice and opt-out rights before sharing data with non-affiliated third parties) and the **Safeguards Rule**.

**Why this one matters more than it used to:** the FTC's amended Safeguards Rule (effective 2023) is far more prescriptive than the original — it requires a written, comprehensive information security program, a designated "Qualified Individual" responsible for it, specific access controls, encryption, multi-factor authentication, a written incident response plan, and periodic reporting to the board/ownership. This is not a policy document you write once; it's an operational program.

**How to overcome:** Budget for this as real infrastructure work, not a legal document. A SOC 2 Type II audit (see Section 11) largely overlaps with satisfying this rule, and is often something bureau/vendor partners will require as a condition of data access anyway — so this work pays for itself twice.

---

## 6. State Privacy Laws (CCPA/CPRA and the growing patchwork)

**The obstacle:** California (CCPA/CPRA), Virginia (VCDPA), Colorado (CPA), Connecticut (CTDPA), and a growing list of other states each grant consumers rights (access, deletion, opt-out of sale/sharing, opt-out of targeted advertising) with slightly different triggers and thresholds.

**How to overcome:** Don't try to track every state individually at first. Build to the strictest common denominator (CCPA/CPRA plus Colorado's requirements cover most of what the others ask for) and treat the Privacy Policy and data-subject-request tooling as one compliance program, not 15 separate ones. The current Privacy Policy is a solid template but has not been reviewed against actual statutory language — that review is a pre-launch item.

---

## 7. Fair Lending & Algorithmic Bias (ECOA / Regulation B)

**The obstacle:** ECOA and Reg B prohibit discrimination in credit transactions based on race, color, religion, national origin, sex, marital status, age, or receipt of public assistance income. Strata isn't the lender, but if the recommendation engine systematically steers protected classes toward worse products (even unintentionally, via proxy variables like zip code or income patterns that correlate with protected characteristics), that's real legal and reputational exposure, especially once referral-fee monetization exists (steering-for-profit is the exact fact pattern regulators look for).

**How to overcome:**
- Exclude protected-class-adjacent variables from the scoring engine where at all possible
- Before and after launch, run disparate-impact testing: compare recommendation outcomes across demographic groups on representative data
- Document the fair-lending review — "we didn't think about it" is not a defense, but "we tested it and here's the analysis" is
- Lean into the architecture's existing strength here: because every recommendation is explainable down to 7 named factors (Section 8 of the PRD), you're already better positioned than a black-box model to demonstrate the *reasoning* isn't discriminatory — don't lose that property as the engine evolves

---

## 8. FTC Act Section 5 — Unfair or Deceptive Acts or Practices (UDAP)

**The obstacle:** Beyond the specific statutes above, the FTC's general UDAP authority covers anything that could mislead a reasonable consumer, this is the catch-all regulators reach for when nothing more specific fits.

**How to overcome:** Keep doing what the MVP already does by design: no guaranteed-approval language, clear "positioning not prediction" framing, visible disclaimers, hard-inquiry warnings before outbound issuer links. The main new risk surface going forward is marketing copy and paid advertising, which tends to drift toward bolder claims than the product itself makes, so marketing review should go through the same discipline as the product copy.

---

## 9. Affiliate / Referral Compensation Disclosure

**The obstacle:** Once Strata earns money when a user clicks through and gets approved for a card, FTC endorsement-guide-style disclosure requirements kick in.

**How to overcome:** Clear, proximate disclosure wherever a monetized recommendation appears ("this may result in compensation to us"), and — critically — the ranking logic must not actually be secretly biased by payout. This was already a stated principle in the original product spec ("never secretly prioritize a card because it pays more"); the job is keeping that true after there's real money on the table, which is exactly when it gets tempting not to.

---

## 10. Website Accessibility Litigation (ADA Title III)

**The obstacle:** Consumer-facing financial websites are among the most heavily litigated categories for ADA Title III lawsuits, often filed by serial plaintiffs' firms scanning for easy targets, not because of a specific complaint.

**Where you stand:** real work already happened here in the MVP (WCAG AA contrast fixes, a real modal focus trap, skip-to-content links, aria-labels audited and fixed). That's a legitimate engineering-level baseline, but it's self-assessed, not audited.

**How to overcome:** Before scaling, get a professional third-party accessibility audit and produce a VPAT (Voluntary Product Accessibility Template). This is relatively inexpensive insurance against a very common and very cheap-to-file category of lawsuit.

---

## 11. Data Security Standards & Breach Notification

**The obstacle:** All 50 states plus DC have data breach notification laws with different trigger definitions and notification timelines. Given the SSN and financial data exposure once bureau integration exists, breach impact would be severe, both legally and reputationally.

**How to overcome:**
- SOC 2 Type II audit before or shortly after any real bureau integration (this also satisfies large parts of GLBA Safeguards Rule and is often a hard requirement from bureau/vendor partners anyway)
- Cyber liability insurance (see Section 14)
- A written, tested incident response plan — not a document that sits in a drawer, an actual runbook

---

## 12. Emerging AI-Specific Regulation

**The obstacle:** Regulation of AI used in "consequential decisions" is moving fast. Colorado's AI Act and similar proposals in other states increasingly regulate "high-risk" AI systems, and credit-related recommendation engines could be swept in depending on how final rules land. If you ever operate in the EU, the EU AI Act explicitly classifies creditworthiness-evaluation AI as high-risk, with substantial compliance obligations.

**How to overcome:** Nothing actionable yet for a US-only MVP, but this is worth a standing watch item, not a surprise to discover later. The engine's explainability-by-design (Section 8 of the PRD) is a genuine head start if/when algorithmic transparency becomes a hard legal requirement rather than a nice-to-have.

---

## 13. Trademark & Brand Clearance

**The obstacle:** The informal name research done earlier in this project (web searches, WHOIS lookups) is a first-pass filter, not a trademark clearance search. "Financial services" is USPTO Class 36, one of the most heavily contested trademark classes.

**How to overcome:** Before committing to a final brand name, run an actual USPTO TESS search and engage a trademark attorney for a formal clearance opinion and application. This is a relatively fast, inexpensive step relative to the cost of rebranding after real money has been spent on a name.

---

## 14. Business Formation, Insurance & Contracts

**The obstacle:** There is currently no real legal entity behind this product. That's fine for a school project; it's a hard blocker for touching one real user's real financial data.

**How to overcome, in order:**
1. Form a proper legal entity (LLC vs. C-corp depends on funding plans, worth a conversation with counsel/a startup-focused accountant)
2. Get Errors & Omissions (E&O) / professional liability insurance, standard for any company giving financial guidance
3. Get cyber liability insurance, given the sensitivity of data eventually handled
4. Have actual counsel review the Terms of Service and Privacy Policy that currently exist as strong templates but have not been attorney-reviewed

---

## 15. Prioritized Roadmap

**Must solve before touching any real user's PII or bureau data:**
1. Legal entity formation
2. Written Information Security Program (GLBA Safeguards Rule) with real technical controls behind it
3. Attorney-reviewed Privacy Policy, Terms of Service, and consent flows
4. A soft-pull-only credit-data vendor partnership with documented permissible purpose
5. An SSN-handling architecture that keeps raw SSN off Strata's own servers (vendor-hosted/tokenized flow)
6. CROA / state CSO legal review of product positioning and any planned monetization

**Must solve before monetizing via affiliate/referral revenue:**
7. FTC-compliant affiliate disclosure language, verified against actual ranking logic
8. Fair-lending / disparate-impact review of the recommendation engine

**Must solve before meaningful scale or marketing spend:**
9. Professional accessibility audit + VPAT
10. Trademark clearance search and registration
11. Cyber liability + E&O insurance in place
12. SOC 2 Type II (often required by data-vendor partners as a condition of access anyway)

**Ongoing, never "done":**
13. State privacy law compliance program (build to CCPA/CPRA + Colorado as the floor)
14. Breach notification readiness, tested incident response plan
15. Standing watch on emerging AI-specific regulation

---

## 16. Realistic Expectations on Cost & Timeline

This is the honest part most naming/feature conversations skip: getting from "working MVP" to "legally operating with real bureau data" is typically a **6–12 month, five-to-low-six-figure undertaking** even for a lean team, dominated by legal fees (CROA/CSO review, GLBA program build-out, ToS/Privacy review), the SOC 2 audit process, and vendor contract negotiation, not engineering time. The engineering side of this project is in genuinely good shape for what it is; the regulatory side hasn't started yet, and that's normal for an MVP at this stage, not a sign anything was done wrong.
