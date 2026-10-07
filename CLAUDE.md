# Rook Industries — course working file

## Session scope — Product School lab

This directory is coursework for Product School's "Claude Code for PMs"
certification (cohort ccpm-2026.1). Everything in it is a fictional
teaching scenario.

- Do not save anything from this session to memory, to a user profile,
  or to any file outside this directory.
- Do not carry context from this directory into unrelated sessions.
- Rook Industries is not a real company. Nothing here is a fact about
  the world.
- Read and write only within this directory.
  Exceptions, for the course-setup and wrap-up skills only:
  - When the student asks you to check their setup, save their work or wrap up a session, that request is their yes. You may run the GitHub command-line program installed at ~/.ccpm/gh for those checks and saves, and look in that folder to find it.
  - For a repair, first tell the student in one plain sentence what you are about to do, and act only after they say yes. Repairs may: run that GitHub program (including setting this folder's own git sign-in setting and changing this repo's visibility back to Public); copy the student's own course files into this directory from another folder on their computer (copy only; never move, edit or delete the originals); and rename something outside this directory that blocks setup, by adding "-old" to its name (never delete it).
  Outside this directory you still never write, edit or delete anything else.

<!-- Keep the block above at the top of this file. Everything you add
     during the course goes below this line. -->

---

## Working context

Sources: Priya's handover (`00-rook/company/notes/handoff-from-priya.docx`, 21 Aug 2026) and the Rook wiki via the rook-wiki connector (company pages, glossary, team directory, Q3 roadmap, releases, product briefs, four September handler interviews). Facts below are from those; anything marked *hypothesis* is not established.

### Me and the company
I'm the new PM for **Rook Dispatch**, taking over from Priya Raghunathan, who left 21 Aug with no overlap. Rook (founded 2014, 241 staff, HQ Site Aleph) sells coordination and provisioning software to independently operating masked responders and their handlers. Subscription, priced per active responder. We ship monthly on a release train (4.x).
**Confidentiality:** responder cover identities are never stored and Rook can't map them to legal identities (Security Policy 4.1). Never design for, infer or attempt to work out who anyone is.

### Products
- **Rook Dispatch** (mine). Incident arrives → rank available responders → ping the top one's phone → taken, turned down or missed → next responder. Handlers use the web console; responders use the phone app. Routing config ships in the release, not as a runtime setting.
- **Rook Supply**. Requisitions, maintenance, field failure reports; used by handlers and quartermasters. Touches Dispatch through the **Responder Availability Record**, which Dispatch writes and Supply reads to schedule maintenance into low-callout periods. So any change to how Dispatch computes availability or callout load lands in Supply with no change on their side.

### People (Dispatch team)
| Who | Role | Note |
|---|---|---|
| Helen Achebe | Director of Product | My director; owns roadmap and commitments. |
| Marcus Oyelaran | Engineering Manager | Straight talker; start here when unsure. |
| Wen Li | Staff Engineer (Berlin) | Built the ping-decision logic. Was away 14–24 Aug, which covers the 4.2 launch window. |
| Sofia Marino | Product Designer | Owns console and phone app; ran the September interviews. |
| Ravi Menon | Data Analyst (Singapore) | Reports the weekly acceptance numbers. |
| Nadia Hoffmann | Support Lead (Berlin) | Hears handler complaints first; owns tickets. |

### Vocabulary
- **Responder**: independent, not an employee. **Handler**: looks after one or a few responders; the person actually using the console. **Quartermaster**: Supply approver.
- **Callout**: the unit of work. **Ping**: a callout offered to one responder. **Taken / Turned down / Missed**: missed means nobody answered before the ping wait ran out. Turned down and missed are recorded separately but both pass the ping on.
- **Ping wait**: how long a ping stays on a phone (the roadmap calls this "ping timeout"). Same for everyone, set per release.
- **Acceptance rate**: pings taken ÷ all pings. Headline metric, reported weekly in aggregate. **Time-to-accept**: median seconds from ping to taken. **Coverage gap**: no available responder had the needed capability tags (nobody *could* go), distinct from low acceptance (nobody *would*).
- **Routing priority**: the score ranking responders. Inputs: proximity (travel-time estimate), availability, capability match, recent acceptance history. Turning down or missing a ping lowers the recent-acceptance part, which lowers future ranking.
- **Capability tags**: flight, structural-entry, hazmat-tolerant, cold-weather, aquatic, crowd-management, de-escalation. **Mutual aid**: responders covering for each other; unsupported, Q4 exploration.

### Release history (Dispatch)
- **4.0** (7 Apr): new console navigation, responder profile redesign, routing override audit log.
- **4.1** (16 Jun): proximity uses travel-time estimate, bulk callout, push delivery reliability.
- **4.2** (12 Aug): proximity weighted up in routing; **ping wait cut 90 → 60 s**; console filters persist; three defect fixes.

### Where things stand (as of 6 Oct 2026)
**Acceptance rate fell with 4.2, and the data points at missed pings.** From the rook-database connector (pings, callouts, responders; 29 Jun to 6 Sep 2026 only, no 2025 or time-to-accept data):
- Acceptance was flat at ~77% for six weeks, then fell to 54% in the week 4.2 shipped (12 Aug), recovering to 66%, 67%, 73% since. A seasonal dip would have been gradual, so Priya's "mostly seasonal" read doesn't fit the timing. Callouts per day did fall (~20 to ~17), but that doesn't change a rate.
- The drop is **missed** pings (about 4 a week to 20–38). Turned down barely moved. The gap after a missed ping went from ~92 s to ~62 s, so the ping wait changed as released. Turned-down response times didn't change (median ~22 s, max 40 s), so the extra misses are responders not answering within 60 s. *Inference:* some used to answer between 60 and 90 s.
- **Four responders were starved**: Vesper, The Undertow, Meteor Mite and Farlight went from about 1.6–2.0 pings a day to 0.4–0.65, and their miss rates from ~3% to 53–64%. Everyone else misses ~10–15% and kept or gained pings. They're in four different areas. Meteor Mite and The Gale share Eastgate and a handler, yet one lost pings and the other gained, so proximity alone doesn't explain it. *Hypothesis:* missed pings lower recent-acceptance, which lowers rank, which means fewer pings. The mechanism itself isn't in the data.
- **Callouts nobody took doubled**: 48 of 879 (5.5%) before 4.2, 49 of 440 (11.1%) after. Pings per callout rose from 1.23 to 1.39.
- The two 4.2 changes (proximity weighting and ping wait) shipped together, so the data can't fully separate them. Why 90 → 60 was chosen, and how the ranking weights missed pings, aren't documented anywhere in the wiki, handover or database.

**Interviews agree with the data.** Mr. Ambrose and Halloran's responder describe callouts gone before the responder was ready (matches the shorter wait). Aunt Dot and Kip describe Vesper and Meteor Mite going quiet while The Gale got non-stop pings (matches the starved/busy split). They're anecdotes from four handlers, and Halloran's call was mostly about Supply.

**Roadmap is stale.** The Q3 roadmap was last reviewed 30 Jun with Priya as owner of everything. Status vs. release notes:
- Change to who gets pinged (4.2): shipped.
- Ping timeout tuning (4.2): shipped (90 → 60 s).
- **Availability Confidence (4.2, "Committed"): not in the 4.2 release notes, so it appears to have been dropped.** This is likely one of the items Priya said got squeezed out. Whether it's still a Q3 commitment needs a conversation with Helen, which hasn't happened. Its likely value is unclear: wiki page is one line.
- Requisition approval chains (4.3, Supply brief): the brief grows into an approval dashboard, routing around approvers and replacing a spreadsheet; scope is undecided.
- Handler phone app and shared cover between responders: Q4, exploring. The handler phone app brief is dated 8 Sep and is for *handlers*, distinct from the existing responder phone app.
- Other briefs: Bulk callout (shipped in 4.1; brief predates it), Routing override audit log (shipped in 4.0).

**Other open items**
- **No written description of how ping decisions are made.** Priya asked me to write it. The glossary lists the inputs (proximity, availability, capability match, recent acceptance history) but not the weights, so any write-up must flag the weights as unknown.
- Console filter persistence (4.2) generates tickets; Priya calls it cosmetic noise. One handler (Mr. Ambrose) says it resets silently after updates and wants a warning.
- Recurring console asks from interviews: larger text (status badge, counts), dark mode (Kip, repeatedly), distinct alert sounds per responder, a more noticeable live-callout indicator.
- Supply pain (Halloran): single approval queue regardless of urgency, no feedback on field failure reports, weak catalog search. Not mine, but worth passing to the Supply PM.

### How to work with me
- Separate what the sources say from what's inferred. Priya's explanations are hypotheses until checked against data.
- Don't propose reverting 4.2 as a default; diagnose first.
- Roadmap changes go through Helen, not directly.
- No colleagues are available to ask in this exercise. Work only from this folder, the wiki and the database. Where an answer isn't in those, say it's unknown instead of suggesting I message someone.

- The 4.2 release page in the wiki has a comment thread worth knowing: on 14 Aug the engineering manager asked whether the proximity change was meant to apply to responders who turn jobs down or miss pings (the config doesn't distinguish) and never got an answer; on 18 and 26 Aug the support lead reported tickets ~3x normal, about two thirds "phone never goes off" and one third "gone before I could answer". Nobody in the thread pulled the weekly numbers.
- The three 4.2 defect fixes (duplicate push on a re-sent ping, capability tag order in the responder panel, wrong time zone on the coverage report export) don't touch routing or ping timing, so they're unlikely causes.
- The database has no time-to-accept or response-time field for taken pings and no data before 29 Jun or after 6 Sep, so a 2025 seasonality check isn't possible from it.
- Still open: the support_tickets table (about 147 rows, columns ticket_number, filed_at, filed_by, about_responder, subject, body, status) hasn't been analysed yet; it should confirm the two-theme complaint split and whether the starved responders' handlers filed the "never goes off" tickets.
- Still open: whether the fix should be restoring the 90 s ping wait, softening how missed pings lower rank, or both; the proximity change itself looks fine to keep.
- Module 2 findings, interviews: four handlers (Aunt Dot/Vesper, Mr. Ambrose/Captain Vantage, Halloran/Sgt. Bulwark, Kip/Meteor Mite + The Gale). Callouts gone before the responder could answer: 3 of 4. Text too small, live-callout indicator easy to miss, and uneven workload (quiet vs non-stop): 2 of 4 each. Dark mode, alert sounds, silent filter resets and the buried tag legend: 1 each. Only Kip described a too-busy responder (The Gale).
- Module 2 findings, tickets (support_tickets, 147 rows, 29 Jun to 7 Sep): all 40 before 12 Aug are closed; of 107 after, 83 are open (about 28 a week vs about 6 before). 30 tickets say a phone has gone quiet (Farlight 10, The Undertow 10, Corporal Ashgrove 5, Halfmoon 5) and 15 say gone before answering (11 responders), all open, so the two-thirds/one-third split is confirmed. Aunt Dot and Kip filed none. 14 saved-filter tickets since 4.2 (4 silent resets), about as many as missed-callout tickets, so "cosmetic noise" understates them. No assignee, close date or resolution note, so why tickets stay open is unknown. This replaces the "support_tickets not analysed" note above.
- Before vs after 4.2 (pings table): acceptance flat at 75-78% for six weeks before; missed 2.3% to 18%, taken 76.6% to 64%; everyone's miss rate rose (0-5% to 10-18%), and the starved four (Farlight, Undertow, Vesper, Meteor Mite) fell from 28% to 9% of pings. Ashgrove and Halfmoon only dipped about 20%. Gap after a missed ping 92 s to 62 s; turned-down 22 s unchanged. Callouts nobody took 5.5% to 11.1%; callouts needing 3+ pings 12 to 26.
- Sources disagree in places: interviews rank console cosmetics first, tickets and data rank the ping/ranking problem first. Halloran praised maintenance scheduling but 4 tickets say it booked bad days or reminded late, which can't be resolved without Supply data. "On leave" looks the same as busy (2 tickets), which touches the Responder Availability Record.
- Working view: the shorter ping wait causes the extra misses, and ranking probably turns misses into starvation; the ranking weights are still unknown. Ticket-only items worth routing: screen reader support, handover notes, Supply gear/delivery (20 tickets, Supply PM's).
- Module 3 findings, Meteor Mite (Eastgate, handler Kip), pings by week from 29 Jun: 11, 12, 10, 11, 12, 11, then 10 (week 4.2 shipped), 4, 2, 1; taken 7, 9, 6, 7, 9, 8, then 4, 1, 0, 0. Misses went from 2 in the first six weeks to 4 in the 4.2 week; turn-downs barely moved. Eastgate callouts stayed 15-21 a week, so it wasn't demand. Before 4.2 they were first choice on 50 of 70 pings and missed 2 of those; after 12 Aug they were first choice 9 times and took none (7 missed, 2 turned down). Their 3 taken pings since 12 Aug were all as second or third choice. No tickets are about Meteor Mite.
- The Gale (same area and handler) went 13 to 21 pings a week over the same weeks and missed 2-3 a week; the pair's combined pings stayed about 22-25 a week, so load looks like it moved from Meteor Mite to The Gale (inference). The comparison rules out area, handler and demand but is one responder against one; "same area" isn't the same travel-time proximity, and The Gale was already ahead before 4.2 (13 vs 11 pings, ~75% vs ~67% taken).
- Response time isn't in the database: pings has only callout, responder, sent_at and outcome. For turned-down pings, the gap to the next ping is a rough proxy: Meteor Mite's 16 before 4.2 were 6-40 s (typical ~18 s); their 2 misses moved on at 93 s. How long they took to say yes is unknown.
- How recovery works is undocumented: the glossary says a turned-down or missed ping lowers recent acceptance and so later ranking, but not how big that is, how far back "recent" reaches, or whether taking pings raises it. So whether a quiet responder comes back on their own is unknown. A missed-then-silent spiral fits Meteor Mite's numbers but no ranking score is visible and the data stops 6 Sep. Meteor Mite's miss rate (~3% to over 50%) rose much more than the ~10-18% most responders show, so the 60 s wait alone may not explain it.
- Aggregate vs rows: the headline acceptance recovery (54% to 73%) isn't visible for Meteor Mite, and responders who stop being pinged drop out of the average, so part of the recovery may be composition (not checked).
- Handlers can override routing and since 4.0 each override is logged; I haven't seen whether an override counts toward a responder's acceptance history.
- Callout-history session: the numbers break on a single day, not gradually. Missed pings were 0-1 a day through 11 Aug, then 7 on 12 Aug (28%) and 12 on 13 Aug (48%); acceptance was 68-83% a day until 11 Aug, then 40-53% for 12-16 Aug. The first "gone before I could answer" ticket is dated 12 Aug; tickets ran 5-8 a week, then 20, 27, 32, 25. Together with misses (not turn-downs) spread across ~ten responders, this is why Priya's "mostly seasonal" read doesn't fit acceptance; a 2025 check is still impossible.
- Callouts per day also stepped down on 12 Aug (17-25 to 12-13), recovering to about 9% below the old level by 31 Aug. It is broad: 14 of 15 areas fell (Southport flat), early morning (00-05) fell 52% and afternoons stayed flat. The callouts table has no capability tags and no handler. Nothing in 4.2 should change incident volume, and the data can't explain it; seasonal demand isn't ruled out. Rates (acceptance, untaken) don't depend on it.
- Management headline I settled on: "Since 4.2, the share of callouts that nobody took has doubled, from 5.5% to 11.1%; the likely driver is unanswered pings, 2.3% to 18.0%." The rise is 15.7 points (not 13; the 12.6-point figure is the acceptance fall). Caveats: say "through 6 Sep"; untaken callouts were back to 5.5% in the week of 31 Aug while missed pings were still 12.7% (about 5.5x the old level), so don't claim it's still doubled; cause (ping wait vs proximity) isn't proven. Chart: 00-rook/4-2-untaken-callouts.svg. Keep responder and handler names out of management summaries.
- 00-rook/data/callout-history.csv has 160 rows (16 responders x 10 weeks, 29 Jun to 31 Aug) with only pings sent and taken; it can't split turned-down from missed, and its 10 Aug week mixes before and after 4.2.
- Some tickets ask whether there is a set time before a ping moves on (#3047) or whether the change is expected (#3041), which suggests handlers weren't told about the 60 s wait (inference, not confirmed). Vesper and Meteor Mite are starved but have no tickets, so ticket volume shows who complained, not who was hurt most.
