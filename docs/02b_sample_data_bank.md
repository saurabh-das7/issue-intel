# 02b — Sample Data Bank

*llm-issue-categorizer · Last updated: April 2026*

---

## Purpose

This document contains three complete sample datasets — one per built-in 
demo tile in the app. Each dataset is 80–100 rows of realistic operational 
issue data with deliberately vague vendor-assigned labels. Expected 
categorisation outputs are documented here for use during pre-launch 
validation against the success metric of ≥80% accuracy.

Each dataset is designed to include:
- Clean rows that map to a single category without ambiguity
- Ambiguous rows that require the full description + resolution notes to classify correctly
- Rows that legitimately belong to multiple categories (multi-tag view test cases)
- Rows that should end up Uncategorised due to low confidence or insufficient signal
- At least one pair of rows likely to trigger a merge flag in the consolidation pass

Data lives in `app/samples.py` as Python dictionaries. The CSV format 
below is the source of truth for that file.

---

## Dataset 1 — UPI Customer Care

**Context description (pre-filled in app):**
"Customer support tickets for a UPI payments app — issues raised by end 
users via in-app chat, email, and call centre."

**Row count:** 90

**Expected taxonomy (6–7 categories after consolidation):**
1. Payment Failure / Transaction Decline
2. Refund / Dispute Resolution
3. KYC / Account Verification
4. Fraud / Unauthorized Transaction
5. App Crash / Technical Error
6. Beneficiary & Limit Management
7. Onboarding / Registration

**Merge flag expected:** "Payment Failure / Transaction Decline" and 
"Refund / Dispute Resolution" may be flagged as overlapping — they are 
related but distinct. The one-liners should disambiguate: payment failure 
= transaction did not go through; refund/dispute = transaction went through 
but user wants money back.

---

### UPI Full Dataset

```
ticket_id,issue_description,current_label,resolution_notes
UPI-001,Transaction of Rs 5000 to a merchant failed but amount was debited from account,Issue resolved,Reversal initiated within 3 business days
UPI-002,UPI PIN reset not working — app shows error after 3 attempts,Technical issue - fixed,PIN reset flow was broken due to OTP gateway issue; fixed in next release
UPI-003,Received a payment of Rs 1200 but it does not show in bank account,Escalated to bank,Bank confirmed credit; delay was on bank processing side
UPI-004,Account linked but UPI ID not getting created,Technical issue - fixed,Backend provisioning delay; resolved after manual trigger
UPI-005,Money sent to wrong UPI ID — user wants refund,Escalated to L2,Informed user that reversal requires recipient consent; case escalated
UPI-006,App crashes every time I try to open the payment history screen,Technical issue - fixed,Crash traced to null pointer in payment history API response; patched
UPI-007,KYC verification pending for 5 days — no update,Escalated to L2,KYC partner queue delay; expedited manually
UPI-008,Suspicious transaction of Rs 12000 that user did not initiate,Under investigation,Transaction flagged; card blocked and investigation opened
UPI-009,Cannot add a new bank account — says account already exists,Issue resolved,Duplicate account check bug; user advised to unlink and re-add
UPI-010,Transaction failed but amount deducted — happened twice in same day,Issue resolved,Double debit confirmed; both reversals processed
UPI-011,UPI transaction limit showing as Rs 0 even after KYC completion,Technical issue - fixed,Limit not refreshed post-KYC; fixed with manual limit reset
UPI-012,Received OTP for a transaction I did not initiate — possible fraud,Under investigation,OTP generation logged; account temporarily suspended pending review
UPI-013,App not loading after latest update on Android 12,Technical issue - fixed,Compatibility issue with Android 12 SDK; hotfix released
UPI-014,Request money feature not working — no notification sent to payer,Issue resolved,Push notification service outage at time of request; user asked to retry
UPI-015,Name on UPI ID does not match bank account name — transactions being rejected,Escalated to bank,Name mismatch due to bank record discrepancy; user directed to bank branch
UPI-016,Cannot complete video KYC — call keeps dropping at step 3,Technical issue - fixed,Video KYC SDK timeout issue; resolved with session extension patch
UPI-017,Rs 50000 transaction rejected — says daily limit exceeded but it is first transaction today,Escalated to L2,Limit reset timing issue at midnight; confirmed as bug; patch in progress
UPI-018,Linked bank account showing wrong balance,Issue resolved,Balance fetch API caching issue; cache cleared; resolved
UPI-019,Merchant QR code payment failed — money debited but merchant did not receive,Escalated to bank,Interbank settlement delay; merchant credit confirmed after 48 hours
UPI-020,App asks for UPI PIN every 30 seconds — very frustrating,Technical issue - fixed,Session timeout misconfiguration; fixed in v3.2.1
UPI-021,International transaction blocked — I am travelling abroad,User educated and closed,UPI international payments not supported; user informed
UPI-022,Refund for cancelled order not received after 10 days,Escalated to L2,Merchant refund not initiated; escalated to merchant ops team
UPI-023,KYC documents uploaded but status still shows pending,Escalated to L2,OCR failure on document scan; manual review triggered
UPI-024,Two factor authentication not working — cannot receive OTP,Technical issue - fixed,SIM binding issue post-SIM swap; re-binding initiated
UPI-025,Money transferred to beneficiary but shows as failed on my end,Escalated to bank,Bank confirmed credit; UI status not updated due to webhook failure
UPI-026,Fraudulent merchant charged me twice for same order,Under investigation,Merchant flagged; second charge reversal initiated; investigation ongoing
UPI-027,Cannot scan QR code — camera permission granted but scanner does not open,Technical issue - fixed,Camera SDK crash on specific device models; patch released
UPI-028,Autopay mandate setup failing for electricity bill,Issue resolved,NPCI mandate API intermittent failure; retried and succeeded
UPI-029,UPI registration failed — says mobile number already registered,User educated and closed,Number registered on another app; user advised to deregister first
UPI-030,Daily transaction limit not resetting at midnight,Technical issue - fixed,Timezone offset bug in limit reset scheduler; fixed
UPI-031,Cannot delete old bank account — getting error 403,Technical issue - fixed,Authorization bug in account deletion API; fixed in v3.3.0
UPI-032,Transaction successful on my end but recipient says not received,Escalated to bank,Interbank delay; funds credited to recipient within 24 hours
UPI-033,App showing duplicate transactions in history,Technical issue - fixed,Duplicate entry due to retry storm on payment API; deduplication fix applied
UPI-034,Biometric login stopped working after phone software update,Technical issue - fixed,Biometric SDK incompatibility with OS update; resolved in patch
UPI-035,Rs 2000 debited for a failed recharge — no reversal after 7 days,Escalated to L2,Biller not responding; reversal initiated from app side
UPI-036,Getting error code UPI-404 when trying to pay at petrol pump,Issue resolved,Specific terminal compatibility issue; user advised to use QR code instead
UPI-037,New phone — cannot re-register on UPI with same number,User educated and closed,Guided user through device migration flow
UPI-038,Transaction declined with message insufficient funds — account has sufficient balance,Escalated to bank,Bank-side decline due to technical error; bank confirmed and resolved
UPI-039,Received request to pay Rs 85000 from unknown ID — looks like fraud,User educated and closed,Advised user to decline and block the UPI ID; no transaction occurred
UPI-040,KYC expired — cannot transact until renewed but renewal flow is broken,Technical issue - fixed,KYC renewal API returning 500 error; fixed; user re-initiated successfully
UPI-041,App language changed automatically to a regional language I cannot read,Technical issue - fixed,Locale detection bug; fixed; user restored to English
UPI-042,Split bill feature — my share was deducted but others show as pending for 3 weeks,Issue resolved,Reminder sent to pending payers; user informed of limitation
UPI-043,Cannot link credit card — says feature not available in my region,User educated and closed,Credit card UPI feature not available in user's state; informed
UPI-044,Passbook not loading — spinning indefinitely,Technical issue - fixed,Passbook API timeout under high load; resolved
UPI-045,Rs 500 cashback not credited after qualifying transaction,Escalated to L2,Cashback engine delay; credited manually after verification
UPI-046,Wrong name showing for beneficiary before confirming payment — very confusing,Issue resolved,Name fetch API returning stale cache data; cache invalidation fix applied
UPI-047,Account blocked after 3 failed PIN attempts — unable to unblock,Technical issue - fixed,Unblock flow not triggering correctly; manual unblock + guided reset
UPI-048,Merchant collected payment but order was cancelled — refund not initiated,Escalated to L2,Merchant notified; refund SLA set at 5 business days
UPI-049,App size increased massively after update — phone storage full,User educated and closed,Advised user to clear app cache; no bug confirmed
UPI-050,UPI ID search showing wrong person name,Escalated to bank,Name resolution pulling from incorrect NPCI record; escalated to NPCI
UPI-051,Transaction history showing transactions from 2 years ago as today,Technical issue - fixed,Timestamp display bug in passbook; fixed
UPI-052,Cannot raise a dispute for a transaction older than 45 days,User educated and closed,Dispute window is 45 days per RBI guidelines; user informed
UPI-053,Rs 15000 sent to correct ID but recipient is in different bank — not received,Escalated to bank,Interbank IMPS failure; funds returned to sender within 48 hours
UPI-054,Notification for successful payment received but passbook shows failed,Technical issue - fixed,Notification and passbook on different event streams; sync fix applied
UPI-055,My UPI ID was used in someone else's fraudulent transaction,Under investigation,UPI ID spoofing attempt logged; account secured; investigation opened
UPI-056,Scheduled payment did not execute at the set time,Technical issue - fixed,Scheduler job missed due to maintenance window; compensation payment sent
UPI-057,App not working on tablet — layout broken,Technical issue - fixed,Tablet layout not supported; tablet mode added in v3.4.0
UPI-058,Cannot find option to change UPI PIN — interface confusing,User educated and closed,User navigated to correct screen with support guidance
UPI-059,Payment to government portal failing consistently — works on other apps,Escalated to L2,Government portal VPA compatibility issue; workaround via BHIM advised
UPI-060,Refund received in wrong bank account — I have two accounts linked,Escalated to L2,Refund routing logic defaulted to primary account; setting added
UPI-061,Rs 1000 transaction showing as pending for 4 days,Escalated to bank,Bank processing hold; funds released after bank intervention
UPI-062,App does not support my regional language for voice commands,User educated and closed,Feature roadmap item; user informed of current language support
UPI-063,Merchant QR expired — app does not show clear error,Technical issue - fixed,Expired QR error message improved in v3.3.2
UPI-064,Cannot see beneficiary account details after adding — just shows UPI ID,User educated and closed,By design for security; user informed
UPI-065,Multiple failed transactions in one day — account not blocked but transactions keep failing,Escalated to bank,Bank-side risk engine blocking; bank requested to whitelist
UPI-066,Received payment notification but amount not in account after 2 days,Escalated to bank,Credit confirmed by bank after investigation; passbook updated
UPI-067,App crashing on login after I changed my phone number,Technical issue - fixed,Phone number change flow not invalidating old session token; fixed
UPI-068,Sent Rs 10000 to correct number but recipient says ID is not active,Escalated to L2,Recipient's UPI ID deregistered; return initiated
UPI-069,QR code payment charged Rs 100 more than displayed price,Under investigation,Merchant terminal rounding issue under investigation; refund of excess initiated
UPI-070,Cannot set up autopay for OTT subscription — keeps timing out,Technical issue - fixed,Mandate API timeout; resolved after infrastructure scaling
UPI-071,Account shows as frozen — no explanation given,Escalated to L2,Account flagged by fraud system; unfrozen after manual KYC re-verification
UPI-072,Two different people with same name in beneficiary list — cannot tell them apart,User educated and closed,Advised to add nickname to beneficiaries for differentiation
UPI-073,Transaction declined due to network error but amount deducted,Issue resolved,Amount returned within 24 hours; standard reversal process
UPI-074,App not working on VPN — is this expected,User educated and closed,VPN usage blocks UPI transactions by design; user informed
UPI-075,New device registration taking more than 30 minutes,Technical issue - fixed,Device registration service latency spike; resolved
UPI-076,Cannot raise complaint for international transaction,User educated and closed,International transactions not supported; user informed
UPI-077,Getting error when trying to view transaction receipt,Technical issue - fixed,Receipt generation API returning 503; fixed
UPI-078,Autopay deducted amount but service provider says not received,Escalated to bank,Settlement timing mismatch; confirmed credit to provider after 24 hours
UPI-079,Account verification asking for Aadhaar but I only have PAN,Escalated to L2,PAN-based KYC path enabled for user; verification completed
UPI-080,App showing negative balance — clearly an error,Technical issue - fixed,Display bug in balance calculation; fixed; actual balance unaffected
UPI-081,Payment to friend failed with vague error — no reason given,Issue resolved,Error message improvement added; underlying bank timeout was cause
UPI-082,Receiving payments fine but sending always fails,Escalated to bank,Outward payment block on account; bank resolved after identity confirmation
UPI-083,Multiple cashback offers applied to same transaction — is this allowed,User educated and closed,Only one cashback offer applicable per transaction; excess reversed
UPI-084,Registration successful but cannot make any transactions — all fail,Escalated to L2,Account in probation period post-registration; limits applied as per RBI norms
UPI-085,App showing maintenance screen for 3 hours during business hours,Technical issue - fixed,Scheduled maintenance overran; resolved; compensation offered
UPI-086,Cannot close my account — option not available in settings,User educated and closed,Account closure requires branch visit per RBI guidelines; user informed
UPI-087,UPI transaction history not matching bank statement,Escalated to bank,Bank statement reconciliation discrepancy; bank confirmed all credits; display lag
UPI-088,Amount entered as Rs 1000 but Rs 10000 deducted — typo not caught,Under investigation,Amount confirmation screen bypassed due to UI bug; refund initiated; bug fixed
UPI-089,App showing someone else's name on my profile page,Technical issue - fixed,User profile cache collision; cleared; user data isolated
UPI-090,Cannot use UPI on a rooted phone — getting blocked,User educated and closed,Rooted device policy explained; security requirement confirmed
```

---

### UPI Test Case Notes

| Test purpose | Row IDs |
|-------------|---------|
| Clean single-category (unambiguous) | UPI-001, UPI-006, UPI-013, UPI-022, UPI-027 |
| Ambiguous — needs resolution notes to classify | UPI-025, UPI-032, UPI-054, UPI-066, UPI-073 |
| Multi-tag candidates (payment failure + fraud) | UPI-008, UPI-026, UPI-055, UPI-088 |
| Multi-tag candidates (KYC + onboarding) | UPI-029, UPI-040, UPI-079 |
| Uncategorised candidates (low signal) | UPI-049, UPI-062, UPI-074 |
| Merge flag trigger (payment failure ↔ refund) | UPI-005, UPI-022, UPI-035, UPI-048 |

---
---

## Dataset 2 — Ads System Monitoring

**Context description (pre-filled in app):**
"Tickets created by an automated monitoring system tracking the health 
of an Ads platform — covering bid auctions, ad delivery, creative 
quality, budgets, and measurement pipelines."

**Row count:** 85

**Expected taxonomy (6–7 categories after consolidation):**
1. Bid / Auction Anomaly
2. Ad Delivery / Serving Failure
3. Creative / Quality Rejection
4. Budget / Pacing Issue
5. Measurement / Tracking Gap
6. Policy / Compliance Flag
7. Infrastructure / Latency Spike

**Merge flag expected:** "Ad Delivery / Serving Failure" and 
"Infrastructure / Latency Spike" may be flagged — infrastructure 
issues often cause delivery failures. One-liners should distinguish: 
delivery failure = ad not showing to users; latency spike = system 
response time degradation regardless of delivery impact.

---

### Ads Monitoring Full Dataset

```
ticket_id,issue_description,current_label,resolution_notes
ADS-001,Win rate for brand keyword campaigns dropped 34% in last 2 hours — no bid changes made,Performance issue - resolved,Competitor surge detected; no action required
ADS-002,CPM spiking 3x on video inventory in EMEA — pacing burning budget 2x faster than planned,Revenue impact - root cause found,Floor price adjustment by publisher; flagged to account team
ADS-003,Ad creative rejected for alcohol brand — policy reason not specified in rejection notice,Quality problem - escalated,Policy team confirmed brand falls under restricted category; clearer rejection message added
ADS-004,Conversion tracking showing 0 events for last 6 hours — pixel not firing,System alert - investigated,Pixel wrapper update broke event firing; rollback initiated
ADS-005,Click-through rate dropped 60% on mobile placements — no creative changes,Performance issue - resolved,Ad slot position change by publisher; not a platform issue
ADS-006,Budget depleted in 2 hours for a campaign set to run all day — pacing failed,Revenue impact - root cause found,Pacing algorithm bug during high-traffic period; fix deployed
ADS-007,Frequency cap not working — same user seeing the same ad 20+ times per hour,Quality problem - escalated,Frequency cap enforcement bug in real-time bidder; patch applied
ADS-008,Impression data in dashboard not matching third-party measurement tool — 18% discrepancy,System alert - investigated,Discrepancy within normal range for viewability filtering; documented
ADS-009,Ad serving latency increased to 800ms — SLA is 200ms,System alert - investigated,Database query bottleneck during peak load; query optimised
ADS-010,Campaign paused automatically — no manual action taken,System alert - investigated,Budget limit reached earlier than expected due to pacing issue; resumption guide sent
ADS-011,Geo-targeting not working — ads showing outside specified target region,Performance issue - resolved,IP geolocation database stale update; refreshed
ADS-012,Video ad skipped tracking event not firing — completion rate seems inflated,System alert - investigated,Tracking event mismatch in VAST tag; corrected
ADS-013,Brand safety keyword block list not being applied to new ad placements,Quality problem - escalated,Keyword block list sync delay; force-sync applied
ADS-014,Bid floor increased by exchange but platform still bidding at old floor — losing all auctions,Performance issue - resolved,Floor price cache not refreshing in real time; cache TTL reduced
ADS-015,Ad image creative flagged for blurry quality — image resolution is 1080p,Quality problem - escalated,Compression artifact introduced during creative processing; processing pipeline fixed
ADS-016,ROI reporting showing negative values for profitable campaigns,System alert - investigated,Currency conversion bug in reporting layer; fixed; historical data reprocessed
ADS-017,Same ad appearing in ad slot and organic content feed simultaneously — duplication,Quality problem - escalated,Publisher feed integration returning duplicate placements; publisher notified
ADS-018,Budget utilisation at 3% after 8 hours for a high-budget campaign,Performance issue - resolved,Overly restrictive audience targeting; account team advised on expansion
ADS-019,Attribution model switched from last-click to data-driven without change request,System alert - investigated,Auto-optimisation feature changed model; feature disabled for this account
ADS-020,High impression count but zero clicks — possible bot traffic,Quality problem - escalated,Invalid traffic filter flagged 98% of impressions as non-human; campaign suspended
ADS-021,Ad disapproved for missing AdChoices icon — icon is present in creative,Quality problem - escalated,Detection algorithm false positive; manual review confirmed icon present; approved
ADS-022,ROAS dropped from 4.2 to 1.1 overnight — no campaign changes,Performance issue - resolved,Audience segment data feed delay caused targeting to revert to broad match
ADS-023,CPC bidding not working — all campaigns defaulted to CPM,System alert - investigated,Bid type migration bug; CPC restored for affected campaigns
ADS-024,Ad delivery stopped completely for one advertiser across all campaigns,System alert - investigated,Advertiser account flagged for payment hold; finance team notified
ADS-025,Viewability rate dropped from 72% to 31% in one hour,System alert - investigated,Above-fold placement removed by publisher without notice; account team notified
ADS-026,Dynamic ad creative not pulling product feed data — showing blank fields,Quality problem - escalated,Product feed API authentication token expired; refreshed
ADS-027,Auction timeout rate increased to 15% — normally below 2%,System alert - investigated,Downstream bidder service latency increase; auto-scaled
ADS-028,Measurement pixel firing twice per page load — inflating conversion counts,System alert - investigated,Pixel fired on both page load and DOM ready events; deduplicated
ADS-029,Campaign targeting 25-34 age group showing to 55+ users,Performance issue - resolved,Age targeting segment definition error in audience builder; corrected
ADS-030,Ad spend for yesterday not appearing in today's report,System alert - investigated,ETL pipeline delay; data backfilled within 6 hours
ADS-031,Click fraud detected — same IP clicking on ads 300 times in one hour,Quality problem - escalated,IP blocked; clicks invalidated; credit applied to advertiser
ADS-032,Ad copy contains competitor brand name — approved despite policy violation,Quality problem - escalated,Competitive keyword detection missed; ad paused; policy team informed
ADS-033,Reporting API returning 504 timeout for large date range queries,System alert - investigated,Query optimisation needed for reports spanning more than 90 days; fix in progress
ADS-034,Mobile app install campaign not tracking installs — iOS only,System alert - investigated,ATT framework changes breaking SKAdNetwork attribution; workaround documented
ADS-035,Bid adjustments for device type not being applied,System alert - investigated,Bid modifier bug introduced in last deploy; rollback applied
ADS-036,Ad rotation set to even rotation but one creative getting 95% of impressions,Performance issue - resolved,Rotation algorithm defaulting to optimised mode; setting corrected
ADS-037,Landing page URL in ad is returning 404 — ad still serving,Quality problem - escalated,Dead link detection not running on this campaign type; fix deployed
ADS-038,Cross-device attribution not working — desktop conversions not linked to mobile clicks,System alert - investigated,Identity graph sync delay; resolved within 24 hours
ADS-039,Ad serving rate dropping progressively over 3 hours — no clear cause,System alert - investigated,Memory leak in ad server causing gradual performance degradation; server restarted
ADS-040,Keyword match type changed from exact to broad automatically,System alert - investigated,Smart bidding optimisation overriding match types; setting corrected
ADS-041,Impression share data missing from competitive intelligence report,System alert - investigated,Data provider contract renewal delay; data restored after contract signed
ADS-042,Remarketing list size dropped from 500K to 2K overnight,System alert - investigated,Cookie consent policy change reduced trackable audience; expected behaviour documented
ADS-043,Ad creative approved yesterday flagged and disapproved today with same policy,Quality problem - escalated,Policy model update changed classification threshold; appealed and reinstated
ADS-044,Campaign budget set in USD but charging in EUR — advertiser in Germany,System alert - investigated,Currency setting defaulting to account region; corrected; credit issued for difference
ADS-045,Server-side bidding integration latency averaging 1.2 seconds — SLA is 100ms,System alert - investigated,Header bidding wrapper conflict; resolved after publisher-side config change
ADS-046,Audience segment overlap analysis showing 0% overlap for all segments,System alert - investigated,Segment overlap calculation bug; fix deployed; historical analysis rerun
ADS-047,Ad shown on brand-unsafe content despite brand safety controls,Quality problem - escalated,Brand safety list not applied to new content category added by publisher; escalated
ADS-048,Third-party tag firing in non-consent markets — GDPR violation risk,System alert - investigated,Consent management platform integration gap; tag blocked in affected markets
ADS-049,Bid landscape data not updating in optimisation dashboard,System alert - investigated,Bid landscape job failing silently; alerting added; job restarted
ADS-050,CPA target being ignored — spending 3x target CPA consistently,Performance issue - resolved,Smart bidding model cold-start period; manual CPA floor set as guardrail
ADS-051,Ad delivery for a new campaign at 0% after 24 hours — campaign approved,Performance issue - resolved,Budget too low for competitive auction entry; account team advised
ADS-052,Advertiser reporting dashboard loading time exceeds 45 seconds,System alert - investigated,Query plan regression after database upgrade; optimised
ADS-053,Negative keyword list not preventing ads from showing for excluded terms,Performance issue - resolved,Negative keyword propagation delay after list update; propagated correctly after 2 hours
ADS-054,Ad impression count showing correctly but revenue reporting at zero,System alert - investigated,Revenue attribution model missing for new ad format; added
ADS-055,Display ad rendering broken on Safari 17 — creative showing as broken image,Quality problem - escalated,WebP format not supported on older Safari; JPEG fallback added
ADS-056,Search term report missing 40% of search queries — shows as other,System alert - investigated,Privacy threshold changes masking low-volume search terms; expected behaviour documented
ADS-057,Automated rules not triggering at scheduled time,System alert - investigated,Rules engine timezone bug; corrected
ADS-058,Ad format not rendering correctly on connected TV inventory,Quality problem - escalated,CTV ad spec mismatch; creative team notified to reformat
ADS-059,Traffic quality score dropped below threshold — account flagged for review,Quality problem - escalated,Invalid traffic percentage exceeded 5% threshold; account under review
ADS-060,Budget pacing showing on track but campaign exhausted budget 4 hours early,Revenue impact - root cause found,Pacing model not accounting for auction price volatility; model updated
ADS-061,Impression data for yesterday not finalised 36 hours later,System alert - investigated,Data finalisation pipeline SLA breach; root cause: upstream partner delay
ADS-062,Ad targeting excludes a geography but ads still serving there,Performance issue - resolved,Exclusion not applied to one sub-exchange; corrected
ADS-063,Conversion value reporting showing 100x actual revenue,System alert - investigated,Currency decimal handling bug; fixed; historical data corrected
ADS-064,Ad disapproved for counterfeit goods — product is legitimate,Quality problem - escalated,Brand name in title triggered counterfeit classifier falsely; appealed; reinstated
ADS-065,Click data showing in analytics but not in ads platform — discrepancy,System alert - investigated,Click deduplication logic difference between platforms; documented
ADS-066,New ad format approved by policy team but disapproved by automated system,Quality problem - escalated,Policy approval not reflected in automated classifier; override applied
ADS-067,Serving rate for one specific placement dropped to zero overnight,System alert - investigated,Publisher decommissioned the placement; removed from targeting
ADS-068,Bid strategy switching between manual and smart bidding randomly,System alert - investigated,A/B test framework accidentally enrolling production campaigns; test isolated
ADS-069,Advertiser receiving duplicate billing for same impressions,Revenue impact - root cause found,Billing pipeline processing same events twice; deduplication fix applied; credit issued
ADS-070,Audience targeting using first-party data not scaling beyond 10K users,Performance issue - resolved,First-party data match rate low due to hashed email format mismatch; corrected
ADS-071,Real-time bidding response time degraded after infrastructure migration,System alert - investigated,DNS resolution latency post-migration; resolved after DNS TTL propagation
ADS-072,Creative performance data not available for first 48 hours after launch,System alert - investigated,Creative performance dashboard has 48-hour data lag by design; documentation updated
ADS-073,Ad serving suspended for entire advertiser account without notice,System alert - investigated,Automated fraud detection triggered; account reinstated after manual review
ADS-074,Keyword bidding data not visible in shared library across campaigns,System alert - investigated,Shared library sync bug; fixed after cache clear
ADS-075,Privacy-safe audience modelling not activating for opted-in users,System alert - investigated,Consent signal not flowing through to modelling pipeline; pipeline fix applied
ADS-076,HTML5 ad creative not animating on mobile — works on desktop,Quality problem - escalated,Mobile animation library version conflict; resolved with creative team
ADS-077,Auction win rate for remarketing campaigns declined 45% in one week,Performance issue - resolved,Cookie deprecation reducing remarketing audience match rate; expected trend documented
ADS-078,Custom conversion event not being tracked after app update,System alert - investigated,SDK event name changed in app update without updating ads platform config; corrected
ADS-079,Campaign serving normally but zero data in real-time reporting dashboard,System alert - investigated,Real-time reporting pipeline delayed due to Kafka consumer lag; resolved
ADS-080,Ad rejected for misleading claims — copy is factual with cited sources,Quality problem - escalated,Policy classifier flagging superlatives; appeal submitted with supporting evidence
ADS-081,Impression frequency across devices much higher than frequency cap set,System alert - investigated,Cross-device frequency capping not enabled; feature activated for account
ADS-082,Automated bidding algorithm increasing bids during low-conversion hours,Performance issue - resolved,Bidding model not incorporating time-of-day conversion patterns; model retrained
ADS-083,Budget allocation between campaigns in a shared budget not working as expected,Performance issue - resolved,Shared budget priority settings not saved correctly; corrected
ADS-084,Video completion rate data missing for campaigns running on new publisher network,System alert - investigated,New publisher not sending completion events; publisher integration team engaged
ADS-085,Ad targeting a specific device model not applying correctly — showing on all devices,Performance issue - resolved,Device targeting parameter not passed correctly to bidder; corrected
```

---

### Ads Monitoring Test Case Notes

| Test purpose | Row IDs |
|-------------|---------|
| Clean single-category | ADS-001, ADS-006, ADS-009, ADS-013, ADS-031 |
| Ambiguous — needs resolution notes | ADS-010, ADS-022, ADS-042, ADS-056, ADS-077 |
| Multi-tag candidates (delivery + infrastructure) | ADS-009, ADS-027, ADS-039, ADS-045, ADS-071 |
| Multi-tag candidates (measurement + policy) | ADS-034, ADS-048, ADS-075 |
| Uncategorised candidates | ADS-072 (by-design behaviour, not an issue) |
| Merge flag trigger (delivery ↔ infrastructure) | ADS-009, ADS-027, ADS-039, ADS-045 |

---
---

## Dataset 3 — Copilot Support

**Context description (pre-filled in app):**
"Support tickets for a Copilot-style AI assistant product — issues 
raised by enterprise users experiencing problems with AI-generated 
responses, integrations, and assistant behaviour."

**Row count:** 80

**Expected taxonomy (6–7 categories after consolidation):**
1. Hallucination / Factual Error
2. Context Loss / Memory Issue
3. Response Quality / Formatting
4. Slow Response / Latency
5. Integration / Access Issue
6. Capability Misunderstanding
7. Privacy / Data Concern

**Merge flag expected:** "Hallucination / Factual Error" and 
"Response Quality / Formatting" may be flagged as overlapping — 
both relate to output quality. One-liners should distinguish: 
hallucination = factually incorrect content; response quality = 
correct content but poorly structured or presented.

---

### Copilot Support Full Dataset

```
ticket_id,issue_description,current_label,resolution_notes
COP-001,Copilot cited a research paper that does not exist — gave me a completely fabricated reference,Bug reported - escalated,Known hallucination pattern on citation requests; guardrail being developed
COP-002,Response taking more than 90 seconds for simple questions — used to be instant,Performance issue - resolved,Infrastructure scaling event; resolved after auto-scaling triggered
COP-003,Copilot lost context of our conversation after 10 messages — started answering generic questions,Bug reported - escalated,Context window limit reached; user not notified; notification added
COP-004,Formatting of code blocks completely broken — code showing as plain text,Bug reported - escalated,Markdown renderer update introduced regression; fixed
COP-005,Copilot gave me a step-by-step guide for a process that our company discontinued 2 years ago,Bug reported - escalated,Training data cutoff limitation; user advised on knowledge cutoff date
COP-006,Cannot access Copilot from work laptop — says account not licensed,User complaint - resolved,License provisioning delay for new employee; resolved after IT ticket
COP-007,Asked for a summary of last meeting — Copilot summarised a completely different meeting,Bug reported - escalated,Meeting context injection pulling wrong meeting ID; bug fixed
COP-008,Copilot response in French even though my interface is set to English,Bug reported - escalated,Language detection fallback bug; locale enforcement added
COP-009,Copilot keeps repeating the same sentence 3-4 times in every response,Bug reported - escalated,Repetition penalty parameter misconfigured in latest model update; fixed
COP-010,Asked Copilot to summarise a confidential document — worried it will be used for training,User complaint - resolved,Data handling and training opt-out policy explained to user
COP-011,Response quality much worse after yesterday's update — answers are vague and unhelpful,User complaint - resolved,Model update feedback logged; quality regression being investigated
COP-012,Copilot cannot access my calendar even though integration is enabled,Bug reported - escalated,Calendar OAuth token expired; re-authentication resolved the issue
COP-013,Got a confident wrong answer about our internal product roadmap — Copilot invented details,Bug reported - escalated,Grounding failure on internal document retrieval; RAG pipeline investigated
COP-014,Copilot response cut off mid-sentence — happened 5 times today,Bug reported - escalated,Max token limit hit without graceful truncation; fix applied
COP-015,Cannot share a Copilot conversation thread with a colleague,User complaint - resolved,Share feature requires both users to have active licenses; explained
COP-016,Copilot using competitor product name when I asked about our product — very confusing,Bug reported - escalated,Named entity substitution error in response generation; logged for model fix
COP-017,Previous conversation history not loading — shows empty on reopen,Bug reported - escalated,Conversation persistence service outage; restored from backup
COP-018,Copilot answered a question about HR policy incorrectly — employee acted on wrong information,Bug reported - escalated,HR policy document not in retrieval index; document added; guardrail added for sensitive HR queries
COP-019,Response time fine but responses are only 1-2 sentences — not enough detail,User complaint - resolved,Response length setting on minimum; user guided to adjust verbosity setting
COP-020,Copilot sharing my conversation context with other users in my organisation — data leak concern,User complaint - resolved,Tenant isolation architecture explained; no cross-user data sharing confirmed
COP-021,Integration with project management tool not syncing task updates,Bug reported - escalated,Webhook endpoint for task sync returning 401; token refresh fixed
COP-022,Copilot hallucinating employee names when asked about team members,Bug reported - escalated,Directory integration not connected; user directed to IT for setup
COP-023,Bullet point formatting renders incorrectly in exported PDF,Bug reported - escalated,PDF export renderer not handling nested bullets; fix in v2.3.1
COP-024,Copilot session timing out after 5 minutes of inactivity — losing all context,User complaint - resolved,Session timeout configurable in admin settings; IT admin notified
COP-025,Model giving different answers to the same question asked twice in the same session,User complaint - resolved,Non-deterministic model behaviour explained; temperature setting option shared
COP-026,Cannot upload a PDF document for analysis — getting error,Bug reported - escalated,File upload size limit of 10MB being exceeded; limit raised to 25MB in next release
COP-027,Copilot confidently told me a colleague left the company — they are still here,Bug reported - escalated,Directory data staleness issue; real-time directory sync implemented
COP-028,Extremely slow response when asking Copilot to process large Excel files,Performance issue - resolved,Large file processing not optimised; async processing added
COP-029,Cannot use Copilot in offline mode — need this for travel,User complaint - resolved,Offline mode not supported; feature request logged
COP-030,Code suggestion in wrong programming language — I specified Python but got JavaScript,Bug reported - escalated,Language preference not persisting across sessions; persistence fix applied
COP-031,Copilot unable to read emails from shared mailbox — only reads personal inbox,Bug reported - escalated,Shared mailbox permissions not supported in current integration; roadmap item
COP-032,Response formatting completely different between web and mobile app,Bug reported - escalated,Mobile markdown rendering inconsistency; mobile fix in progress
COP-033,Getting an error when I ask Copilot to write a document longer than 5 pages,Bug reported - escalated,Output length cap hitting before task completion; cap adjusted
COP-034,Copilot gave legal advice with high confidence — we are not a legal firm,Bug reported - escalated,Disclaimer guardrail for legal/medical/financial advice added to system prompt
COP-035,All my saved prompts disappeared after the update,Bug reported - escalated,Prompt library migration bug; prompts restored from pre-update backup
COP-036,Response latency increased specifically for document summarisation tasks,Performance issue - resolved,Document chunking algorithm inefficient for large documents; optimised
COP-037,Copilot not available during working hours — showing maintenance screen,Performance issue - resolved,Unplanned maintenance overrun; resolved; SLA credit issued
COP-038,Cannot integrate Copilot with our on-premise data warehouse,User complaint - resolved,On-premise integration not supported; cloud migration options discussed
COP-039,Asked Copilot to keep our discussion confidential — is that possible,User complaint - resolved,Confidentiality limitations and data handling policy explained
COP-040,Copilot generating responses in a very formal tone even for casual questions,User complaint - resolved,Tone adjustment setting explained; user updated preference
COP-041,Follow-up questions not being understood in context — treating each as a new question,Bug reported - escalated,Conversation history not being passed correctly to model; fix deployed
COP-042,Copilot summarised a 50-page document but missed the key conclusions section entirely,Bug reported - escalated,Chunking strategy skipping final section of long documents; fixed
COP-043,Getting duplicate responses — same answer appearing twice in the chat,Bug reported - escalated,UI rendering bug causing response duplication; fixed
COP-044,Copilot unable to access files in SharePoint even after granting permissions,Bug reported - escalated,SharePoint Graph API scope mismatch; corrected permissions scope in app registration
COP-045,Asking about a person in my organisation returns information about a different person with same name,Bug reported - escalated,Disambiguation logic not implemented for same-name entities; improvement in roadmap
COP-046,Copilot answer contains a table but table is rendering as raw markdown characters,Bug reported - escalated,Table rendering regression in v2.2 update; hotfix deployed
COP-047,First response in a new session always takes 30+ seconds — subsequent responses are fast,Performance issue - resolved,Cold start latency in session initialisation; warm pool added
COP-048,Copilot suggested an action that could have caused data loss — no warning given,Bug reported - escalated,Destructive action warning guardrail not applied to this command type; added
COP-049,Cannot find option to delete my conversation history,User complaint - resolved,Data deletion guide shared; history deletion available in account settings
COP-050,Copilot response contains information from a document I deleted last week,User complaint - resolved,Document deletion from retrieval index can take up to 72 hours; documented
COP-051,Copilot keeps suggesting I use a feature that was deprecated 6 months ago,Bug reported - escalated,Model knowledge not updated post-deprecation; product changelog added to context
COP-052,Charts and graphs requested in response not rendering — showing as text description only,Bug reported - escalated,Chart generation feature not enabled for this tenant; IT admin notified
COP-053,Copilot gave step-by-step instructions for bypassing a company security policy,Bug reported - escalated,Policy guardrail gap; system prompt updated; security team notified
COP-054,Response quality for non-English language queries significantly worse,User complaint - resolved,Non-English quality known limitation; documented; improvement roadmap shared
COP-055,Cannot connect Copilot to Salesforce CRM — integration not listed,User complaint - resolved,Salesforce integration in beta; waitlist registration shared
COP-056,Session disconnecting every hour even during active use,Bug reported - escalated,Session keepalive not working correctly; fixed
COP-057,Copilot confidently gave me the wrong date for a company event,Bug reported - escalated,Event data in retrieval index outdated; index refresh schedule increased
COP-058,Admin dashboard for managing user licences not loading,Bug reported - escalated,Admin portal outage; resolved within 2 hours
COP-059,Very long responses even for simple yes/no questions,User complaint - resolved,Verbosity setting explained; default verbosity reduced in next update
COP-060,Copilot not respecting formatting instructions given in the prompt,Bug reported - escalated,Instruction following regression in latest model update; investigated
COP-061,Concerned that Copilot responses may be stored and reviewed by vendor,User complaint - resolved,Data residency and audit access policy explained in full
COP-062,Copilot gave a different summary of the same document to two colleagues,Bug reported - escalated,Non-deterministic summarisation behaviour; investigation open
COP-063,All formatting lost when I copy a Copilot response into Word,User complaint - resolved,Rich text copy feature added in v2.3.0; user guided to update
COP-064,Copilot API integration stopped working after our SSO migration,Bug reported - escalated,SSO token issuer change broke API auth; updated token validation config
COP-065,Got a response that included information from another user's conversation,Bug reported - escalated,Critical: potential data isolation failure; incident opened; full investigation
COP-066,Copilot not useful for technical coding questions — gives very basic answers,User complaint - resolved,Advanced coding mode explained; user switched to developer-focused configuration
COP-067,Response times degraded consistently between 2-4pm every day,Performance issue - resolved,Peak load period identified; capacity added; latency reduced
COP-068,Copilot recommended a vendor that is on our blocked vendor list,Bug reported - escalated,Blocked vendor list not in retrieval context; added to system context
COP-069,Cannot export conversation to PDF on mobile,Bug reported - escalated,PDF export not available on mobile in v1; added to v2.3 roadmap
COP-070,Copilot asking for my password to access a system — should never do this,Bug reported - escalated,Prompt injection attack pattern detected; security team notified; guardrail added
COP-071,Follow-up questions after a long conversation getting slower and slower,Performance issue - resolved,Context window size causing inference slowdown; context compression implemented
COP-072,Cannot use voice input for Copilot on desktop,User complaint - resolved,Voice input not supported on desktop; mobile only; feature request logged
COP-073,Copilot providing different policy guidance depending on how question is phrased,Bug reported - escalated,Policy retrieval inconsistency; RAG grounding improved for policy documents
COP-074,Responses containing bullet points show correctly on screen but break in email export,Bug reported - escalated,Email export HTML conversion bug; fix in progress
COP-075,Copilot gave me a confident answer and cited a document I cannot access,Bug reported - escalated,Citation to restricted document without access check; access-aware citation added
COP-076,After inactivity Copilot forgets what role I asked it to play at the start,Bug reported - escalated,System prompt role context not persisted across inactivity timeout; fix applied
COP-077,Copilot not available in our region — only showing US English version,User complaint - resolved,Regional availability roadmap shared; local language version ETA provided
COP-078,Asking Copilot to act as a different AI assistant — is this allowed,User complaint - resolved,Usage policy on persona requests explained; request declined by system as expected
COP-079,Getting rate limit errors during peak hours — cannot use the product,Performance issue - resolved,Per-tenant rate limits increased; burst capacity added
COP-080,Copilot generated a response that contained offensive content when given an ambiguous prompt,Bug reported - escalated,Content safety filter gap; incident logged; filter retrained
```

---

### Copilot Test Case Notes

| Test purpose | Row IDs |
|-------------|---------|
| Clean single-category | COP-001, COP-002, COP-006, COP-010, COP-037 |
| Ambiguous — needs resolution notes | COP-011, COP-025, COP-042, COP-062, COP-073 |
| Multi-tag candidates (hallucination + privacy) | COP-065, COP-070, COP-075 |
| Multi-tag candidates (context loss + quality) | COP-007, COP-041, COP-076 |
| Uncategorised candidates (policy/usage question) | COP-039, COP-078 |
| Merge flag trigger (hallucination ↔ quality) | COP-005, COP-009, COP-011, COP-019 |
| Security / high-severity rows | COP-053, COP-065, COP-070, COP-080 |

---

## Validation Reference

For pre-launch validation against the ≥80% accuracy success metric, 
run the sample dataset for each domain using the No Suggestions mode 
and compare the LLM-generated categories against the expected taxonomy 
documented above. Any category name variation that correctly groups 
the right tickets counts as a match — exact label wording is not 
required, only correct grouping.

The following rows are the hardest classification cases and are 
the most important to validate manually:

| Dataset | Hard cases | Why |
|---------|-----------|-----|
| UPI | UPI-025, UPI-032, UPI-054 | Status contradiction between fields requires resolution notes to classify |
| UPI | UPI-049, UPI-074 | Low information signal — likely Uncategorised |
| Ads | ADS-022, ADS-042, ADS-077 | Root cause is expected behaviour, not a bug — easy to misclassify |
| Ads | ADS-072 | By-design data lag documented as an issue — should be Uncategorised |
| Copilot | COP-025, COP-062 | Non-determinism complaints — category depends on framing |
| Copilot | COP-039, COP-078 | Policy/usage questions with no technical issue — likely Uncategorised |
| Copilot | COP-065 | Critical security incident — should surface in Uncategorised analysis if not caught |

---

*Previous: [02a — PRD](./02a_prd.md)*  
*Next: [03 — UX Flow & Wireframe](./03_ux_flow_wireframe.md)*
