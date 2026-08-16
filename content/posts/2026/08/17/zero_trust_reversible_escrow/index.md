---
title: Zero-Trust Reversible Escrow for combating scams
tags: ["systemdesign", "architecture", "payments", "fintech", "india", "fraud", "cybersecurity"]
toc: true
---

## Introduction

In 2025, a 57-year-old woman in Bengaluru was kept under constant video surveillance for 6 months by scammers claiming to be CBI/RBI/Cybercrime officials. In those 6 months, she was terrorized into making 187 separate bank transfers, draining ≈ ₹32 crore of her life savings.[^da-6m]

[^da-6m]: [Bengaluru Woman Loses Rs 32 Crore In Digital Arrest That Lasted 6 Months](https://www.ndtv.com/bangalore-news/bengaluru-woman-loses-rs-32-crore-in-digital-arrest-that-lasted-6-months-9647994)

There is a clear uptick in cases. So RBI put out a discussion paper. Reading about it on news motivated this post.

**Reference:** [Discussion Paper on Exploring Safeguards in Digital Payments to Curb Frauds (April 2026)](https://rbidocs.rbi.org.in/rdocs/Publications/PDFs/DP090420261ED5D6E68D724A6EA870B7E68E45F80F.PDF)

TLDR:

> The discussion paper sets out the following four options, namely,
> 1) Lagged credit for authorised push payments other than low value;
> 2) Additional authentication by trusted person for high-value digital transactions by vulnerable sections of society;
> 3) Only accounts with satisfactory additional review to receive large credits; and
> 4) Customer-induced controls

The main proposals which were reported visibly in the news were 1 and 4. Basically, RBI proposed a 1-hour delay for sending > ₹10,000 to a new person / beneficiary, and a kill-switch for disabling transactions.

As you can imagine, this is a stupid, feel-good idea.

- The ruthless "digital arrest" criminals manipulate people into isolating/locking themselves up for multiple hours, days[^da-15d], or even months[^da-6m] (like above). They don't care if the victim kills themselves the next moment on camera. A 1-hour delay is meaningless, the attacker will simply tell the victim the system is "processing" or forces them to break the payment into smaller chunks to bypass the threshold. A one-hour delay is not the same thing as a one-hour interruption of the attack.

- Disabling transactions after the money has left your account achieves nothing. The act of disabling would happen when the victim is no longer panicked, i.e., after the damage is already done.

[^da-15d]: [90-year-old Jaipur ex-judge held under 15-day digital arrest, loses Rs 2.5 crore](https://www.indiatoday.in/cities/jaipur/story/digital-arrest-scam-jaipur-former-judge-loses-crore-to-fake-cbi-rbi-officials-ptag-2970183-2026-08-13)

We are asking the wrong question. The question isn't how to delay the *victim's* payment. It is how to choke the *attacker's* monetization. 

To systematically neutralize these threats, I propose a **"Zero Trust Reversible Escrow"** architecture. This model shifts the friction from the sender's outbound ledger to the receiver's inbound liquidity, attacking the crux of the scam's modus operandi.

Disclosure: AI assistance was used to get a quick polished draft from my own speak.

---

## 1. Unbundling the Transaction

The fundamental mistake in the RBI's proposal is treating the *time before payment* as the primary security boundary. A conventional instant payment collapses several distinct events into one split-second action. A safer architecture explicitly separates them:

- **Authorization:** The sender successfully authenticates and instructs the payment.

- **Visibility:** The sender and beneficiary can clearly see that the transaction exists on the ledger.

- **Customer Credit Availability (Liquidity):** The beneficiary can actually withdraw, spend, or transfer the funds.

- **Finality:** The transaction passes the defined protection period and can no longer be reversed.

These events do not have to be coupled. A payment can be *visible* without being *spendable*.

This gives the banking system a period where the payment visibly exists on the screen to pacify the scammer, but the fraudster's liquidity does not.

---

## 2. The Danger of Sender-Side Friction

If the RBI implements a 1-hour delay on the sender's side, it will likely take one of two flawed forms:

- **The Hard Block (Threshold Friction):** If the RBI outright blocks transactions over ₹10,000 to new beneficiaries, it simply forces the scammer to adapt. Because "low value" transfers are exempt, the scammer forces the victim into structuring: *"The high-value limit was blocked. Break the payment into chunks of ₹9,900 and send them using different accounts or one-by-one".*

- **The Pending Void:** What if the RBI holds the transaction in a "pending" void where the sender's money is debited but the receiver sees ₹0? This escalates psychological terror. The fake police officer assumes the victim is trying to trick them, starts screaming threats, and forces the panicked victim to drain a secondary bank account.

Thus, sender-side friction makes the issue worse as it forces the attacker to adapt and/or escalate the coercion.

---

## 3. The Core Model: 24-Hour Inbound Liquidity Escrow

Instead of creating sender-side friction, the transaction must process instantly and visually reflect on the destination ledger.

This immediate visual confirmation of "Payment Successful" pacifies the immediate crisis. Because the scammer sees the funds have successfully landed in their ledger, they recognize the victim has complied. This stops the scammer from screaming at the victim to empty a secondary bank account out of panic.

However, the destination account cannot withdraw, transfer, or monetize these funds for a baseline protection window of 24 hours. Note that the underlying interbank settlement does not need to remain pending for 24 hours.

Sophisticated scammers will know this lock exists, but it forces them into a far more expensive logistical problem. Instead of a fast "smash-and-grab", the scammer must now attempt to keep the victim awake, isolated, and under acute psychological control for 24 continuous hours to ensure they don't hit the "revert" button.

To be precise on the core-banking abstraction, the ledger state looks like this:

- **Sender Ledger:** -₹10,00,000
- **Receiver Ledger Balance:** +₹10,00,000 *(Status: Restricted Credit)*
- **Receiver Available Balance:** ₹0 *(From the restricted amount)*

A 24-hour baseline lock ensures a massive intervention window. It creates enough time for phone batteries to die, internet connections to drop, family members to intervene, and a biological sleep cycle to physically interrupt the victim's acute fight-or-flight compliance.

---

## 4. Crippling the Mule Network Economics

This architecture fundamentally alters the operational economics of money laundering. Mule networks rely entirely on rapid dispersion. When a socially engineered payment lands in a compromised account, fraudsters instantly split it and wire it to 50 other accounts or buy cryptocurrency before the victim realizes they've been scammed.

By trapping the inbound liquidity for 24 hours, the conversion of fraudulent authorization into irreversible liquidity is interrupted. This completely destroys the rapid-dispersion model, granting the bank's automated fraud-detection systems the time to flag the anomaly and freeze the mule account before the money escapes.

---

## 5. Handling Prolonged Attacks: Velocity-Compounded Escrows

Now you would say this won't help the Bengaluru woman. And you would be correct. A critical scenario to address is the prolonged attack where scammers hold a victim hostage across multiple days to attempt to run out a single 24-hour clock.

To neutralize this, lock times must scale with transaction velocity. If an account initiates back-to-back high-value transfers to new beneficiaries within a rolling 7-day window, the escrow duration **compounds dynamically**. For example, Transfer 1 = 24h lock, Transfer 2 = 48h lock, Transfer 3+ = 72h+ lock.

Simultaneously, consecutive high-value drain-attempts must automatically trigger {asynchronous, high-priority anomaly tickets} to the bank's fraud operations desk for human intervention. The scammer cannot simply wait out the clock if repeated transfers keep pushing the liquidity horizon further away.

---

## 6. Countering "Buyer Fraud" (Protecting Genuine Receivers)

A critical vulnerability in any reversible payment system is first-party abuse. If a sender can unilaterally pull funds back with a single button at hour 23, legitimate P2P trade (like buying a used laptop via UPI after meeting via OLX) becomes impossible. The buyer could simply take the laptop, walk away, and hit the "revert" button.

To prevent the receiver from becoming the victim, the system must not allow reversal to happen instantly on sender's request. The reversal is a fraud protection mechanism, so it must go through an appropriate fraud handling procedure.

That is, the system must distinguish between a *customer-requested cancellation* and a *fraud-triggered reversal*:

- **No Silent, Instant Clawbacks:** Hitting "revert" does not execute an immediate, unverified refund to the sender.

- **Transition to Dispute Freeze (`FRAUD_HOLD`):** Triggering a reversal immediately freezes the provisional funds on the receiver's end (preventing withdrawal) and locks the claim on the sender's end.

- **Mandatory Legal Routing:** Instead of an informal rollback, the dispute is formally routed into existing interbank fraud reporting and cybercrime investigation frameworks. Following through with the reversal should require the sender to submit a formal digital fraud declaration under penalty of law, introducing massive legal friction for bad-faith buyers trying to abuse the revert policy.

---

## 7. Emergency Bypass: Out-of-Band Mutual De-Anonymization

If a user genuinely needs to instantly send money for immediate use (like a sudden hospital bill for a family member), a bypass mechanism must be provided to ensure system liquidity is not choked during genuine emergencies. However, since it will allow one to bypass the timelock, it must have friction as a deliberate security feature.

To prevent bloating the core payment rails (UPI/IMPS) with unnecessary things, the verification can occur **out-of-band** via the banks' security overlay. The system must initiate an automated, synchronous **Dual-Node Video KYC** session through a strict, step-by-step sequence, such as the following:

1. **Ephemeral Consent & Warning:** The receiver is presented with a clear prompt: *"To accept these emergency funds instantly, you must consent to a live video session and reveal your registered bank account number, branch, full legal name, residence address city, and current live geo-location to the sender".*

2. **Mutual Visual De-anonymization:** Both the sender and the receiver must be present on the same video feed simultaneously. The sender sees the receiver's live face, legal name, bank account number, registered KYC city, and their real-time location (for e.g. *"Live from Jamtara, Jharkhand"*).

3. **Automated Integrity Checks:** It's 2026. We can use fully automated modern tools (without a human-in-the-loop, potentially employing AI) to enforce environmental integrity. The system can try to verify strict liveness (try to block deepfakes and screen-sharing), ask contextual security questions (e.g. forcing the sender to verbalize the purpose of the transfer), and match the physical face and ID on camera against the government KYC on file (Aadhaar and PAN records), to structurally defeat spoofed or stolen IDs.

4. **Cryptographic Release:** If all the checks pass, the verification layer issues an "Unlock Token" to the core ledger, bypassing the 24-hour escrow.

This high-friction bypass is intentional. It provides a necessary, forced pause for the user to think through the procedure before committing. Simultaneously, it gives the bank's automated systems the required time and data to accurately assess the transaction's risk.

**Why this breaks the scam:**

- **Shattering the Authority Illusion:** Attackers impersonating law enforcement rely 100% on unilateral anonymity. By forcing the banking app to display the stark reality: that the money is landing in a private citizen's account, accompanied by a live video feed of an unrelated individual broadcasting from a completely different state than they claim to be in, the psychological manipulation shatters.

- **The KYC Mismatch (Spoof Defeat):** Fraud syndicates often use stolen or spoofed ID documents to open shell accounts. Even if the KYC documents on file are forged, the physical person sitting in front of the live camera will not match the face on the KYC file. The automated verification detects the biometric mismatch and fails.

- **Mule Legal Exposure:** Money mules will be incentivized to refuse to participate in a live, recorded, two-way video feed directly opposite the victim they are extorting out of fear of immediate, undeniable legal exposure.

Because bad actors cannot satisfy this mutual de-anonymization condition without burning their operation, the bypass fails, and the 24-hour escrow remains firmly engaged.

---

## 8. Progressive Trust Architecture (Graded Beneficiary Status)

Defining a "first-time beneficiary" via a binary status is highly vulnerable to "penny testing". An attacker will simply ask the victim to transfer ₹10 to ensure the account works, before demanding a ₹10,00,000 transfer. If the system simply registers the ₹10 transfer and marks the beneficiary as permanently trusted, the escrow is bypassed.

Trust must scale in a graded manner. For example, transferring a small amount clears the base tier. Transferring a larger amount triggers the escrow mechanism. For even larger amounts surpassing subsequent thresholds, the escrow triggers again. The system could utilize a maximum of 5 (just an example) scaling triggers based on transaction value before an account is granted frictionless P2P capabilities.

Historical transaction familiarity can reduce friction, but no single successful small payment should ever be capable of magically laundering a subsequent ₹10,00,000 payment into the "permanently trusted" category.

---

## 9. State Machine Diagram: Transaction & Dispute Flow

```
[Initiate High-Value P2P Transfer]
                |
   [Check Velocity & Risk Tier]
                |
+---------------+---------------+
|                               |
[Known / Cleared Tier]          [New / High-Risk Tier]
|                               |
[Instant Settlement]            [Ledger Updated (Debited / Credited)]
                                |
                                [Liquidity Locked in Escrow]
                                (24H Base / Compounding on Velocity)
                                |
        +-----------------------+-----------------------+
        |                       |                       |
        v                       v                       v
[Timeout Expired]       [Reversal Initiated]    [Emergency Bypass]
        |                       |                       |
[Settlement Finalized]  [State: FRAUD_HOLD]     [Out-of-Band Sync Video]
(Liquidity Available)   (Funds Frozen on Both)  [  De-Anonymization    ]
                        |                       |
                        [Mandatory Fraud]       [If Passed: Issue Unlock]
                        [    Triage     ]       [Token to Core Ledger   ]
                        |
        +---------------+---------------+
        |                               |
[Fraud Confirmed]               [Dispute / Legit Trade]
        |                               |
[Ledger Compensated]            [Escrow Maintained /]
(Funds Returned)                [  Arbitration Hold ]

```

---

## 10. System Exceptions & Policy Rules

- **Strict P2M Exemption:** Unilateral reversal workflows and provisional escrows are strictly disabled for verified P2M (Person-to-Merchant) transactions. Commercial commerce must route through standard payment gateway dispute and chargeback frameworks. Different payment relationships require different models of finality.

- **Extended Escrows for Vulnerable Demographics:** Data shows that prolonged, multi-week digital arrests disproportionately target high-net-worth senior citizens. By default, the fast-track bypass feature should be disabled for high-risk demographics, and their escrow lock should be extended from 24 hours to **72 hours or more**. This ensures the escrow cannot be socially engineered out of them under any circumstances, unless they have explicitly opted-in to the bypass feature via an in-person branch mandate.

---

## Conclusion

This architecture physically breaks the ROI loop of fraudsters by choking their instant liquidity while protecting the transactional integrity of genuine users.

Delaying the sender's transfer merely tests how long a scammer can scream at a victim on the phone.
