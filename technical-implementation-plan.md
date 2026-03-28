# Technical Implementation Plan — Henry the Car Guy DriveCentric Upgrades

## Context
Henry the Car Guy is a car salesman using DriveCentric CRM. We've pitched him three upgrades (AI Speed-to-Lead, Autopilot Follow-Ups, Morning Scorecard). This plan details the technical implementation for each, including prerequisites, step-by-step configuration, message templates, and testing.

---

## Prerequisites (All Three)

Before starting any upgrade:
1. **Admin access** to Henry's DriveCentric account (or paired session with his admin)
2. **Discovery call** (30 min) to capture:
   - His DriveCentric tier and which add-on modules are active (Automation Hub? Genius? AIM?)
   - Lead sources currently feeding into DriveCentric (AutoTrader, Facebook, website form, walk-ins)
   - His current follow-up process (manual texts? any existing sequences?)
   - Existing dashboard/report configuration
   - His voice/tone for messages (casual? professional? emoji user?)
   - Sales goals (units/month target, average deal size)
   - Which vehicles he's currently pushing (inventory priorities)
3. **Module check**: If Automation Hub or Genius Reply are not on his plan, he'll need to add them ($TBD from DriveCentric). The Morning Scorecard works with the base platform.

---

## Upgrade 1: AI Speed-to-Lead

### Goal
Every new inbound lead gets a personalized SMS response within 60 seconds, 24/7, with no manual intervention from Henry.

### Implementation Steps

#### 1.1 — Verify Genius AI Module is Active
- Navigate to: DriveCentric Admin → Modules → Automation Hub / Genius
- If not active: flag to Henry that this requires an add-on (quote from DriveCentric)
- If active: proceed to configuration

#### 1.2 — Configure Lead Source Routing
- Map all inbound lead sources to trigger auto-response:
  - **AutoTrader leads** → confirm feed is connected in DriveCentric integrations
  - **Facebook leads** → verify Facebook Lead Ads integration is active
  - **Website form** → confirm website lead capture pushes into DriveCentric
  - **Manual/walk-in** → these won't trigger auto-response (handled by Sequence in Upgrade 2)
- For each source, verify the lead lands in DriveCentric with: name, phone, email, vehicle of interest

#### 1.3 — Build Auto-Response Templates
Write 3 variations (A/B tested by DriveCentric Genius):

**Template A — Direct:**
```
Hi [First Name]! This is Henry. I saw you're interested in the [Vehicle]. Great taste! I have [X] in stock right now. When's a good time for a quick test drive? 🚗
```

**Template B — Question-led:**
```
Hey [First Name], Henry here! Thanks for checking out the [Vehicle]. Quick question — are you looking to buy soon or just getting a feel for options? Either way, happy to help. 🤝
```

**Template C — Value-add:**
```
Hi [First Name]! Henry at [Dealership]. I saw your interest in the [Vehicle] — love that choice. I just ran some numbers and can get you into it for less than you'd think. Want me to send over a quick payment estimate?
```

#### 1.4 — Configure Appointment Booking
- Set up Genius to offer scheduling: "Want to come see it? I've got openings [tomorrow/this weekend]. What works?"
- Connect to Henry's calendar (Google Calendar or DriveCentric's built-in scheduling)
- Set available test drive slots (e.g., Mon-Sat 9am-6pm, Sun 11am-4pm)

#### 1.5 — Set Response Timing & Escalation
- Auto-response delay: 30-60 seconds (feels human, not robotic)
- If customer replies with a question Genius can't handle → escalate to Henry via push notification
- Business hours (9am-7pm): Genius responds + alerts Henry
- After hours: Genius responds autonomously, Henry reviews in morning brief

#### 1.6 — Testing
- [ ] Submit a test lead from each source (AutoTrader test form, Facebook test lead, website form)
- [ ] Verify SMS arrives within 60 seconds
- [ ] Verify personalization tokens populate correctly ([First Name], [Vehicle])
- [ ] Test appointment booking flow end-to-end
- [ ] Test escalation: reply with complex question, confirm Henry gets notified
- [ ] Test after-hours: submit lead at 10pm, verify Genius handles it
- [ ] Verify no duplicate responses (one lead = one auto-text, not multiple)

---

## Upgrade 2: Autopilot Follow-Ups

### Goal
5 automated multi-channel sequences that handle Henry's daily follow-up pipeline without manual effort.

### Implementation Steps

#### 2.1 — Audit Existing Sequences
- Check DriveCentric → Sequences for any existing automation
- Note what's active, paused, or broken
- Decide: replace, modify, or build fresh

#### 2.2 — Build Sequence 1: Hot Lead (Days 1-7)

**Trigger:** New lead enters pipeline, lead score = hot, no appointment booked
**Channel mix:** SMS → Email → Call task → Video → SMS

| Day | Time | Channel | Message |
|-----|------|---------|---------|
| 0 | Immediate | SMS | (Handled by Upgrade 1 — Genius auto-response) |
| 1 | 10am | Email | "Here's what I can do for you on the [Vehicle]" — include payment estimate, trade-in CTA, photo of vehicle |
| 2 | 2pm | Call Task | Push notification to Henry: "Call [Name] — interested in [Vehicle], hasn't booked yet" |
| 3 | 11am | Video | Henry records 1 generic video per vehicle category (truck, SUV, sedan) — Genius inserts customer name in text around it |
| 5 | 10am | SMS | "Hey [Name], just checking in — the [Vehicle] is still here but I've had a few other people asking about it. Want me to hold it for you?" |
| 7 | 9am | SMS | "Last shot before I move on — any interest in coming by this week? No pressure either way. 🤝" |

**Exit conditions:** Customer books appointment, replies "not interested", or enters a different sequence.

#### 2.3 — Build Sequence 2: Post-Test-Drive (Days 1-3)

**Trigger:** Opportunity stage changes to "Test Drive Completed"

| Day | Time | Channel | Message |
|-----|------|---------|---------|
| 0 | 1hr after | SMS | "Great meeting you today, [Name]! What did you think of the [Vehicle]? Any questions I can answer?" |
| 1 | 10am | Email | "Your [Vehicle] — the numbers" — include payment breakdown (lease + finance), trade-in value if applicable, any current incentives |
| 2 | 2pm | SMS | "Hey [Name], just wanted to let you know the [incentive/bonus] we talked about expires [date]. Want me to lock it in?" |
| 3 | 10am | Call Task | "Call [Name] — test drove [Vehicle] 3 days ago, hasn't committed" |

#### 2.4 — Build Sequence 3: Gone Cold Win-Back (Day 14+)

**Trigger:** No customer response for 14+ days, opportunity still open

| Day | Time | Channel | Message |
|-----|------|---------|---------|
| 14 | 10am | SMS | "Hey [Name], Henry here. Haven't heard from you in a bit — totally understand, buying a car is a big decision. Just wanted you to know we just got some new incentives on the [Vehicle]. Want me to send the details?" |
| 18 | 10am | Email | "Market update" — show what comparable vehicles are selling for, position Henry's deal as competitive |
| 25 | 11am | SMS | "Hi [Name], one last check-in. If the [Vehicle] isn't the right fit, I'd love to know what would be. I can search our inventory for something better. No hard sell. 🤝" |
| 30 | — | Auto-close | Move to "Lost" with reason "No response" — triggers future nurture |

#### 2.5 — Build Sequence 4: Sold Customer Nurture (Ongoing)

**Trigger:** Deal marked "Won/Sold"

| Timing | Channel | Message |
|--------|---------|---------|
| Day 3 | SMS | "Hey [Name]! How's the new [Vehicle] treating you? If you love it, a quick Google review would mean the world. [Review Link]" |
| Day 30 | Email | "Your 1-month check-in" — service reminder, answer any questions |
| Day 90 | SMS | "Hey [Name], know anyone in the market for a car? I'll take great care of them — and there's a $200 referral bonus for you. 🎁" |
| Day 180 | Email | "6-month service reminder" + current trade-in value of their vehicle ("your [Vehicle] is worth $X right now") |
| Day 365 | SMS | "Happy anniversary with your [Vehicle]! 🎉 If you're ever thinking about an upgrade, I've got you." |

#### 2.6 — Build Sequence 5: Service-to-Sales (Triggered)

**Trigger:** Customer (sold by Henry) checks into service department
**Note:** This requires DriveCentric Service Hub integration — verify it's active

| Timing | Channel | Message |
|--------|---------|---------|
| Same day | Push to Henry | "[Name] is in service today with their [Vehicle] ([Year]). Trade-in value: ~$X. Good upgrade candidates: [suggestions based on what they bought]" |
| Day 1 | SMS (from Henry) | "Hey [Name], hope the service visit went smoothly! By the way, your [Vehicle] is holding its value really well right now. If you ever want to see what an upgrade would look like, I can run the numbers. No pressure!" |

#### 2.7 — Message Personalization & Voice
- Do a 15-min call with Henry to capture his natural texting style
- Pull 10-15 of his actual past messages from DriveCentric to match tone
- Key questions: Does he use emojis? First name or full name? Casual or professional?
- Write all templates in his voice, then have him approve before activating

#### 2.8 — Testing
- [ ] Each sequence triggers correctly from the right pipeline stage change
- [ ] Messages send at correct day/time intervals
- [ ] Personalization tokens ([Name], [Vehicle], etc.) populate correctly
- [ ] Exit conditions work (reply stops sequence, stage change stops sequence)
- [ ] No sequence conflicts (customer isn't in two sequences simultaneously)
- [ ] Opt-out/unsubscribe handling works per TCPA compliance

---

## Upgrade 3: Morning Scorecard

### Goal
Custom DriveCentric Live Dashboard + 8 reports that give Henry instant clarity every morning.

### Implementation Steps

#### 3.1 — Configure Live Dashboard Widgets

**Row 1 — KPI Cards:**
| Widget | Metric | Source |
|--------|--------|--------|
| Units Sold (MTD) | Count of deals Won this month | Opportunities → Won |
| Revenue (MTD) | Sum of deal values this month | Opportunities → Won → Amount |
| vs. Last Month | % change from prior month | Calculated comparison |
| Active Pipeline | Count of open opportunities | Opportunities → Open |

**Row 2 — Priority View:**
| Widget | Content |
|--------|---------|
| Today's Follow-Ups | Sequence tasks + manual tasks due today |
| Hot Leads | Leads with highest engagement score, sorted by urgency |
| Today's Appointments | Scheduled test drives and meetings |

**Row 3 — Trend Charts:**
| Widget | Chart Type |
|--------|-----------|
| Weekly Sales Trend | Bar chart, last 8 weeks |
| Lead Source Performance | Pie/donut — which sources produce closed deals |

#### 3.2 — Build Custom Reports (8 total)

**Standard Reports (5):**
1. **Daily Activity Summary** — messages sent, calls made, appointments set, deals closed (today)
2. **Pipeline by Stage** — count and value at each stage (New → Contacted → Engaged → Test Drive → Closing → Won → Lost)
3. **Monthly Sales vs. Goal** — units and revenue vs. target, with daily run rate
4. **Lead Response Time** — average time to first contact, broken down by source
5. **Lost Deal Analysis** — reasons for lost deals, by vehicle and source

**Advanced Reports (3):**
6. **Lead Source ROI** — leads received vs. deals closed per source (AutoTrader, Facebook, website, walk-in, referral). Cost per lead if ad spend data is available
7. **Vehicle Type Performance** — close rate and average deal size by vehicle category (truck, SUV, sedan, EV)
8. **Customer Lifecycle** — days from first contact to close, segmented by lead source and vehicle type

#### 3.3 — Mobile Optimization
- Verify dashboard renders correctly on DriveCentric mobile app
- Prioritize KPI cards and Today's Follow-Ups for mobile view
- Configure push notification for daily summary (e.g., 7am MT: "Good morning Henry — you have 6 follow-ups today, 2 hot leads, and 1 appointment")

#### 3.4 — Daily Email Summary
- Configure DriveCentric scheduled report email
- Send at 6:30am daily to Henry's email
- Include: KPI snapshot, today's tasks, any new leads overnight

#### 3.5 — Testing
- [ ] All dashboard widgets load with correct data
- [ ] Reports generate accurately (cross-reference with raw DriveCentric data)
- [ ] Mobile app displays dashboard correctly
- [ ] Daily email arrives on time with correct data
- [ ] Pipeline stages map correctly to Henry's actual sales process
- [ ] Lead source tracking accurately attributes closed deals to original source

---

## Delivery Timeline

| Day | Activity |
|-----|----------|
| 1 | Discovery call + access setup + module verification |
| 2-3 | Upgrade 1: AI Speed-to-Lead configuration + template writing |
| 3-4 | Upgrade 2: Build all 5 sequences + write all message templates |
| 4-5 | Upgrade 3: Dashboard + reports configuration |
| 5 | Internal testing of all three upgrades |
| 6 | Walkthrough call with Henry (30 min) — demo everything, get template approvals |
| 7 | Go live — flip all switches, monitor first day |
| 8-14 | Tuning period — adjust templates, timing, dashboard based on real usage |

## Risks & Mitigations

| Risk | Mitigation |
|------|-----------|
| Henry doesn't have Automation Hub / Genius modules | Identify in discovery call Day 1. Quote the add-on cost upfront so there are no surprises. |
| Lead sources not properly connected to DriveCentric | Test each source on Day 1. If a feed is broken, escalate to DriveCentric support. |
| Service Hub not active (blocks Sequence 5) | Deliver Sequences 1-4 on time, position Sequence 5 as a bonus when Service Hub is added. |
| TCPA compliance for automated SMS | All sequences include opt-out language. Verify DriveCentric's built-in compliance handling. |
| Henry doesn't approve message templates quickly | Send templates for review on Day 3, give 24-hour deadline, use generic versions as fallback. |
