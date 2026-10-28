# Signal catalogue

> **Update 28 Oct (council review 1):** keystroke timing is dropped for mobile web (Android keyboards report key code 229); new server-side signals (probation payees, 24 h/30 d new-payee outflow, bait-profit, payee fan-in) and real web signals (paste via `inputType`, dwell, app switches) are in `../03-design/risk-engine.md` v2 §1, with a REAL / SIMULATED column. Digital arrests run on WhatsApp/Skype video calls, so a call signal must include VoIP (Android audio mode), not only cellular call state.

> Priority: **P0** = MVP must-have, **P1** = nice to have, **P2** = stretch.
> **Risk type** says which kind of threat the signal points to. This decides the *kind* of challenge (see `../03-design/risk-engine.md`):
> **IMP** = impostor (account takeover, stolen credentials, SIM swap) · **SCAM** = genuine user being manipulated (APP fraud) · **BOT** = automation/card-testing · **MULE** = risky receiver.
> Updated 2026-10-15 after research: added the scam-in-progress and beneficiary families and the risk-type column. The original rows are kept.

## Behaviour (session)
| Signal | Feature (how we compute it) | Why it matters | Type | Priority |
|---|---|---|---|---|
| Time from login to payment | `login_to_pay_ms`; flag if < 5th percentile of user | Bots/ATO move unnaturally fast | BOT/IMP | P0 |
| Transaction velocity | count of txns in 1h / 24h vs user baseline | Card-testing, account takeover | BOT/IMP | P0 |
| Failed auth attempts recently | `failed_pin_24h` | Credential stuffing / guessing | IMP | P0 |
| Typing / click cadence | z-score of mean inter-key interval vs the user's own profile | Human vs bot; user's own rhythm. Best simple detector EER ≈ 9.6% on the CMU benchmark, so use it as a *contributing* signal, never alone | IMP/BOT | P1 |
| Pasted payee / amount | `payee_pasted` boolean | Scammers send the VPA/account to paste | SCAM | P1 |
| Hesitation / long dwell on confirm | `confirm_dwell_ms` vs user baseline | Coached users pause and are talked through steps | SCAM | P1 |
| Navigation path (skipped steps) | missing expected page events | Scripted flows | BOT | P2 |

## Scam-in-progress context (mobile SDK signals; simulated in the web prototype)
| Signal | Feature | Why it matters | Type | Priority |
|---|---|---|---|---|
| **Active phone call during payment** | `on_call` | On a call in **up to 70% of remote-access-scam sessions vs < 1% of genuine sessions** (BioCatch's own claim) | SCAM | **P0** |
| Screen-sharing / remote-access app active | `screen_share` / `remote_access_app` | How OTPs and PINs get stolen; Google is piloting alerts for this in GPay/Paytm | SCAM/IMP | P0 |
| Session opened from an SMS/WhatsApp link | `entry_source` | Phishing / "pay to unblock" flows | SCAM | P2 |

## Device
| Signal | Feature | Why it matters | Type | Priority |
|---|---|---|---|---|
| New device for this user | `device_id ∉ user.known_devices` | Classic ATO indicator | IMP | P0 |
| Recent SIM change / device re-binding | `sim_change_hours` | SIM swap defeats OTP (Karnataka HC case, ₹87.7 lakh) | IMP | P0 |
| IP geo vs usual location / impossible travel | km/h between last and current geo | Stolen credentials | IMP | P0 |
| VPN / proxy / TOR | `is_vpn` | Hiding origin | IMP/BOT | P1 |
| Emulator / rooted / headless browser | `is_emulator` | Fraud tooling | BOT | P1 |
| Many users on one device | distinct users per device in 7d | Fraud farm / mule handler (BioCatch: ~35 accounts per mule device) | MULE/BOT | P1 |

## Transaction
| Signal | Feature | Why it matters | Type | Priority |
|---|---|---|---|---|
| Amount vs user's median | `amount / user.median_amount` and robust z-score | Out-of-pattern spend; 98.5% of fraud value is in txns > ₹10k | ALL | P0 |
| New merchant / category for user | `merchant ∉ history` | Novelty | IMP/SCAM | P0 |
| **New beneficiary + high amount** (transfers) | `first_time_payee && amount > k × median` | Mule / scam payments: the core APP pattern | SCAM | **P0** (was P1) |
| Unusual hour for this user | hour outside the user's 95% interval | Behavioural mismatch | IMP | P1 |
| Round amounts / just-under-limit amounts | `amount % 1000 == 0`, near the daily limit | Limit probing | SCAM/BOT | P2 |
| Draining the balance | `amount / balance` | Scams empty accounts | SCAM | P1 |

## Beneficiary (receiver) reputation
| Signal | Feature | Why it matters | Type | Priority |
|---|---|---|---|---|
| Payee risk flag | lookup in a mock list modelled on DoT FRI (Medium/High/Very-High) and MuleHunter-style scores | Reuse ecosystem intelligence | MULE | P0 (mocked) |
| Payee account age & inbound fan-in | days since first seen; distinct senders in 24h | New accounts collecting from many strangers are mules | MULE | P1 |

## User context
| Signal | Feature | Why | Type | Priority |
|---|---|---|---|---|
| Vulnerable-user flag | age ≥ 70 (RBI's proposed trusted-person cohort) | Seniors lost ₹4,005 cr in 2025; lower thresholds + trusted-contact challenge | SCAM | P1 |

Sources: see `problem-evidence.md` and `prior-art.md`. Keystroke EER: Killourhy & Maxion 2009, https://www.cs.cmu.edu/~keystroke/KillourhyMaxion09.pdf
