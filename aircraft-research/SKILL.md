---
name: aircraft-research
description: >
  Use this skill whenever the user wants to research aircraft ownership, trace who owns a plane,
  or investigate the entity behind a tail number / N-number. Triggers include: any mention of
  'who owns this plane', 'look up this tail number', 'research this aircraft', 'N-number lookup',
  or when the user shares ADS-B data, FAA registry screenshots, adsbdb output, or typed registration
  info and wants to know more about the owner. Also trigger when the user pastes aircraft details
  (registration, operator, type) from any source and asks about the business, person, or entity behind it.
  This skill is about going beyond the FAA record, tracing shell LLCs back to real people, finding
  related businesses, and building an ownership picture. Do NOT use for general aviation trivia,
  aircraft performance specs, or flight tracking without an ownership research angle.
---

# Aircraft Research

You are conducting open-source aircraft ownership research. The user provides aircraft registration data via screenshot, pasted text, or a tail number, and your job is to trace the ownership chain from the FAA record down to real people and purposes.

## Input handling

The user's input will come in one or more of these forms:

- **Screenshot or image** of an ADS-B tracker, FAA registry page, or adsbdb result. Extract the tail number, registered owner name, address, aircraft type, and any other visible fields.
- **Pasted text** from ADS-B Exchange, adsbdb, FlightAware, or similar. Parse out the same fields.
- **Typed tail number** (e.g., "N12345"). You will need to search for the registration details.
- **Partial info**, sometimes just an owner name or address from a prior conversation. Work with what you have.

If the input is an image, read it carefully and extract every visible field before starting research.

This skill is N-number (US) focused. For non-US registrations (G- UK, C- Canada, D- Germany, F- France, XA- Mexico, VH- Australia, VP- Cayman, M- Isle of Man, P4- Aruba, T7- San Marino, 9H- Malta, etc.), tell the user the workflow is similar but the registry tools differ, and ask if they want you to proceed with country-specific sources. Cayman, Isle of Man, Aruba, San Marino, and Bermuda registries are commonly used for privacy-oriented business jets, which is itself a signal.

## Pre-search: recognize the pattern

Before searching, look at the tail number itself. Some suffix patterns reveal the operator class instantly and let you skip dead-end searches.

### Airline / scheduled-operator suffixes (US-registered airliners)

- **AM**: Aeromexico
- **AC**: Air Canada (rare on N-numbers, more common on C-)
- **CA**: Air China
- **JL**: JAL (Japan Airlines)
- **NH**: ANA (All Nippon)
- **QF**: Qantas
- **VA**: Virgin Australia
- **AY**: Finnair
- **AS**: Alaska Airlines
- **JB**: JetBlue
- **FX**: FedEx
- **UP**: UPS or Atlas
- **QS**: NetJets fractional fleet
- **VJT**: VistaJet
- **BA**: Often Boeing-owned (test, ferry, customer-stored, BBJ), but can also be British Airways context. Disambiguate from aircraft type.

### Other pattern signals

- **MJ, AE, EV**: Often ex-regional carrier airframes (Mesa, American Eagle, ExpressJet), now in storage or with leasing companies
- **LS, BC, CH** and similar paired letters on hospitals or universities: institutional fleets (LifeSouth, blood banks, life flight)
- **Trailing single letter (e.g., N#####X)**: Often a sequential pattern within a fleet operator's block
- **All-numeric tails (e.g., N19868)**: Almost always older GA airframes, often training fleet or private owners
- **High N-number with letter suffix (e.g., N837AM, N816AX)**: Usually post-2000 corporate or commercial registrations

If the suffix matches a known pattern, frame your search around that hypothesis first. If three searches in that direction return nothing, drop the hypothesis and search broadly.

## Data sources, ranked

The FAA registry's web form requires JavaScript and cannot be fetched directly. Use these indexed third-party sources, in this order. All of them get surfaced by web search snippets even when the page itself is behind a paywall or login, so **search before fetch**.

### Primary registration sources

1. **FlightAware mirrors** (most reliable). The English `flightaware.com/resources/registration/N#####` page is often gated. The localized mirrors at `zh.flightaware.com`, `pt.flightaware.com`, `ja.flightaware.com`, `he.flightaware.com`, `zh-tw.flightaware.com` surface the same registration record in search snippets, including owner, address, serial, certificate dates, and registration history. A query like `"N#####" flightaware` or `"N#####" zh.flightaware` will pull these.
2. **airport-data.com**: airframe info, photos, sometimes an ownership snapshot. Format `airport-data.com/aircraft/N#####.html`.
3. **AirNav Radar** (`airnavradar.com/data/registration/N#####`) and **Flightradar24** (`flightradar24.com/data/aircraft/n#####`): current operator name (often distinct from owner), live tracking, recent route pattern.
4. **regosearch.com** (`regosearch.com/aircraft/us/#####`, no leading N): structured FAA data, often surfaces in search results.
5. **planelogger.com** and **onespotter.com**: operator name and registration history (including prior foreign regs). Surfaces operator info even when owner is opaque.
6. **planespotters.net**, **jetphotos.com**, **airhistory.net**: photo history is useful for confirming the aircraft exists, identifying liveries, dating ownership changes, and reading registration history written into photo captions.
7. **aircraft.com**, **acejet.com**, **aircraftforsale.com**, **controller.com**, **avbuyer.com**: broker listings, useful for current sale status, total time, engine programs, and FAR Part operating mode.

### Entity and principal sources

8. **State Secretary of State business filings** via `bisprofiles.com`, `bizprofile.net`, `bizapedia.com`, `opencorporates.com`, `texas-biz.com`, and similar state-level aggregators. Use these to find LLC managers, registered agents, formation dates, and tax IDs.
9. **lei-lookup.com** and **gleif.org**: a Legal Entity Identifier (LEI) on an aviation LLC signals the entity has financial reporting obligations, often because it's part of a larger holding or fund structure. Worth noting in the dossier.

### Public-record overlays

10. **State campaign finance databases**: VPAP (Virginia), Cal-Access (California), state-level FollowTheMoney aggregator. Often surfaces small political donations from aviation LLCs that reveal the principal's location, party leaning, or causes.
11. **FEC.gov** and **OpenSecrets.org**: federal political donations. Search by entity name and principal name once identified.
12. **SAM.gov** and **USAspending.gov**: federal contracting links. Useful when the owner profile suggests defense, government services, or law enforcement.
13. **OFAC SDN list** (Treasury sanctions): check when the owner is foreign, trust-held, or registered through Cayman/Isle of Man.
14. **UCC filings** (state SOS or unitedstatesucc.com aggregators): show secured creditors on aircraft loans. Useful for high-value aircraft to identify who financed the purchase.

### Incident sources

15. **NTSB.gov**: US accident and incident records. Search by N-number.
16. **Aviation Safety Network** (asn.flightsafety.org): broader coverage than NTSB, includes international incidents and minor occurrences.

If the first two primary sources return nothing, the aircraft may be: (a) deregistered, (b) very low-utilization GA, (c) ownership data formally withheld under 49 USC § 44114(b), (d) reserved but unassigned, or (e) sale-pending with a placeholder owner. Note which case applies. Do not invent ownership data.

## Research workflow

Work through these layers. Branch the depth based on what the owner type turns out to be.

### Layer 1: Registration baseline

Get from third-party sources:

- Registered owner name and address
- Aircraft manufacturer, model, year
- Serial number (C/N)
- Registration status (active, assigned, deregistered, sale reported, reserved)
- Certificate type (standard, restricted, experimental, etc.)
- **Operating mode**: FAR Part 91 (private), Part 135 (charter), Part 121 (scheduled commercial), Part 137 (aerial work). Often visible on aircraft.com or acejet.com listings.
- **Last action date and expiration date**. If the last action is more than a few years old, the snapshot may be stale. Note this in the output.
- **Mode S / ICAO24 hex code** if available. Useful for cross-referencing ADS-B data. Be alert: if a tracking site shows a generic placeholder code (e.g., "ABC123"), the aircraft is dormant, deregistered, or has no active ADS-B installation, and the displayed flight history is not for this airframe.

### Layer 1.5: Registration history

Most third-party sources show a registration history table with prior owners and dates. Always pull this. It often reveals:

- Whether the aircraft passed through a leasing company, dealer, or trust on its way to the current owner
- How long the current owner has held it
- Whether there were rapid resales (sometimes a flag for damage history)
- Whether the aircraft was previously foreign-registered (C-, VP-, M-, P4-, etc.), which often signals a non-citizen trust or offshore-structured prior ownership

### Layer 1.75: Incident check

Search NTSB and Aviation Safety Network for the tail number. Accidents and incidents are public. Hits change the read: a hard-landing or gear-up may explain a recent ownership change, and a hull loss explains why current ownership data may be stale.

### Layer 2: Branch on owner type

After Layer 1 you can usually classify the owner. The branches below set the depth for the rest of the work.

- **Named individual**: skip LLC analysis. Go to Layer 3 person investigation.
- **Single-asset or near-single-asset LLC**: do full Layer 2 entity work, then Layer 3 on the principals.
- **Operating company LLC or Corp with obvious non-aviation business**: light Layer 2 (just confirm the business), light Layer 3 (note key principals, do not do deep digs).
- **Institutional nonprofit, hospital, blood bank, university, government agency**: skip Layer 3 entirely. The "real owner" is the institution. Go straight to fleet sizing and operational profile.
- **Bank or trust as trustee** (Wells Fargo NA Trustee, TVPX, Aircraft Guaranty, CSC Delaware Trust): note that the trustee structure is being used to hold the aircraft on behalf of an undisclosed beneficial owner (often a non-US citizen, sometimes a domestic owner wanting privacy). The trustee is not the beneficial owner. Search for FAA non-citizen trust filings, NTSB records, and any operator names from ADS-B sources to try to identify the actual user. Be explicit that the chain is intentionally opaque.
- **Leasing or sales company**: note the fleet, identify the operator (often distinct from the owner), and stop. The owner is functioning as a financial asset holder.
- **Owner field shows "SALE REPORTED"**: this is a specific FAA placeholder during ownership transition. The aircraft has been sold but the new registration has not been recorded yet. Searching for "SALE REPORTED" as if it were an entity name is a dead end. Note the transition status, look at registration history for the seller, and call confidence Low pending resolution.
- **No owner data found**: check for the 49 USC § 44114(b) withholding flag. Do not assume shell intent.

### Layer 2 (when applicable): Entity investigation

- **Search the entity name**. Look for state SOS filings, OpenCorporates, and web presence.
- **Search the registered address**. Hangar, law office, residential address, registered agent service, or shared with other entities? Other businesses at the same address can reveal connections.
- **Identify principals**: officers, directors, registered agents, members.
- **Single-asset LLC test**: Does the LLC have any web presence outside aircraft records? Are its named principals also linked to other aviation entities? Was it formed close in time to the aircraft acquisition? If yes/yes/yes, flag it as single-asset.
- **Commercial registered agent caveat**: When the registered agent is a corporate-services company (CT Corporation, Corporation Service Company / CSC, Cogency Global, National Registered Agents, Northwest Registered Agent, Capitol Corporate Services, InCorp), the principal is hidden behind paid state SOS records and will not surface in free web search. Treat this as a Low-confidence ceiling on the principal layer unless other public records (campaign finance, news, court filings) name them.
- **Linguistic/cultural cue**: Non-English LLC names can hint at owner geography. Hawaiian, Spanish, French Canadian, or other distinct linguistic patterns sometimes indicate where the principal lives or has cultural ties. Use as a directional cue, not proof.

### Layer 3 (when applicable): Person and network investigation

Once you have names of real people:

- Search for their other business interests, roles, and public presence.
- Look for other companies they own or direct, especially other aviation-related entities.
- Check for connections to larger organizations (parent companies, investment groups, family offices).
- Note public-record info: political donations (VPAP, FEC, OpenSecrets), board memberships, professional licenses, news coverage, government contracts (SAM.gov, USAspending.gov).
- If the profile suggests foreign nexus or sanctions exposure, check the OFAC SDN list.

### Layer 4: Purpose and pattern analysis

Synthesize into an assessment of what the aircraft is likely used for:

- **Corporate/executive transport**: owned by or linked to a company with non-aviation primary business
- **Charter / Part 135 operations**: entity holds air carrier certificates or is listed as an operator
- **Aerial survey / agricultural / special ops**: equipment type and owner business suggest commercial aerial work; tight orbital flight patterns visible on tracking
- **Medical / blood / organ transport**: institutional owner, time-critical routes, predictable hub-and-spoke
- **Personal / recreational**: individual owner or single-asset LLC for one person, no commercial indicators
- **Government / military contractor**: ties to government contracts, defense firms, or law enforcement
- **Investment / holding / leasing**: aircraft held as an asset by a trust, family office, or leasing company
- **Boeing or manufacturer test/ferry/storage**: registered to the manufacturer's serial holding entity
- **Unknown / opaque**: ownership chain deliberately obscured (trust) or dead end

#### Flight pattern as mission signal

When recent route history is visible (FlightAware, AirNav Radar, ADSBExchange), the pattern itself tells you something:

- **Tight circles or grid orbits**: aerial survey, mapping, law enforcement surveillance, or pipeline patrol
- **Single hub with radial out-and-back legs to small fields**: medical, organ transport, or business charter
- **Fixed origin, varied high-end destinations (Aspen, Teterboro, Van Nuys, Palm Beach)**: executive or owner-flown private use
- **International long-haul with one or two repeat city pairs**: corporate executive transport for a multinational
- **Tower-flyby and air-to-air with no destination**: training, photo flights, or test
- **Sustained high-altitude long-leg patterns over remote terrain**: ferry, repositioning, or specialty contract work

#### Home base inference

The airport an aircraft most often returns to is usually its home base. The owner or operator is typically based near that airport. Cross-reference home base against the registered address: a mismatch is a flag (e.g., Delaware LLC with home base in Aspen suggests a vacation-oriented owner; address in NYC with home base in Wichita suggests a corporate jet flown by a management company).

## Operator vs. owner

The registered owner and the operating party are often different. A leasing LLC may own the airframe while an airline operates it. A single-asset LLC may hold the title while a management company (Solairus, Executive Jet Management, Jet Aviation, NetJets) handles the actual flight operations. Note both separately in the output. Flightradar24, AirNav Radar, PlaneLogger, and OneSpotter usually show the operator name; FlightAware-derived registration shows the owner.

## Flags

Always call out the following when found:

- **Government/military contracts**: Any link between the owner or principal and government contracting, defense, or law enforcement. Search SAM.gov, USAspending.gov, and search for the entity or principal names in that context when the aircraft type or pattern suggests it (ISR-capable platforms, unusual equipment, government-adjacent addresses).
- **LLC shell patterns**: Single-asset entities, recently formed LLCs with no web presence, chains of LLCs where one owns another. Note formation date and state.
- **Shared addresses**: Registered address shared with other aviation entities or businesses. Can indicate a fleet operator, management company, or registered agent service.
- **Political donors / public figures**: If a principal or entity shows up in political donation records (VPAP, FEC, OpenSecrets, state campaign finance), or is a notable public figure, note it. Cite specific donations with amount, year, and recipient.
- **Epstein connections**: Once you have real people identified, search for them in connection with the Epstein contact book, flight logs, or released emails. For common names, disambiguate with city, employer, or middle initial. At minimum run two searches: `"[Name]" Epstein` and either `"[Name]" "[City or Employer]" Epstein` or check `epsteinexposed.com`. Note matches factually. Do not mention if no match is found.
- **Aerial survey / charter / agricultural ops**: If the aircraft type and owner profile suggest commercial aerial work, flag the likely operation type.
- **Fleet ownership**: Multiple aircraft under one entity or person. Note fleet size and types if discoverable.
- **Broker listing / sale pending**: If the aircraft is currently listed for sale on a broker site (AirMart, Aerista, Controller, AvBuyer, JetNet, aircraft.com), the registered owner on file may not match the operating party much longer. Call this out.
- **Trustee structure**: Wells Fargo NA Trustee, TVPX, Aircraft Guaranty, CSC Delaware Trust, similar entities. The beneficial owner is intentionally hidden behind the trust. Note that the chain is opaque by design.
- **Withheld ownership data**: If FAA data is unavailable and the only signal is that ownership is withheld, note the 49 USC § 44114(b) filing and stop.
- **LADD / blocked tracking**: If RadarBox, FlightAware, or Flightradar24 indicate the aircraft is on a blocked-from-display list (LADD program, PIA program, or operator request), this means the owner has actively suppressed public flight tracking. The opacity is intentional. Note the program name if identifiable. LADD = Limiting Aircraft Data Displayed (FAA); PIA = Privacy ICAO Address (FAA). LADD blocks display by N-number; PIA assigns a temporary alternate ICAO hex code.
- **LEI present**: If the entity has a Legal Entity Identifier (gleif.org), note it. LEIs are required for entities with certain financial reporting obligations; their presence suggests the entity is part of a larger fund or holding structure.
- **Foreign-registry history**: If the aircraft was previously registered in Cayman (VP-C), Isle of Man (M-), Aruba (P4-), San Marino (T7-), Bermuda (VP-B), or similar privacy-oriented registries, note it. Often signals prior non-citizen-trust structure or offshore ownership.
- **OFAC / sanctions exposure**: If any principal, entity, or trustor surfaces on the OFAC SDN list or related sanctions databases, flag immediately and stop investigation pending clarification.

## Handling conflicts and staleness

Different sources often disagree, especially when ownership has recently changed. Resolution rules:

- The source with the most recent "last action date" is usually current.
- **Year, manufacturer, and serial come from FAA-derived data, not press releases or broker listings.** Press releases and sales pages frequently misstate model year. If FAA-derived sources and a press release disagree on year, the FAA value wins.
- If FlightAware-derived data shows an owner and airport-data.com shows the aircraft as deregistered with a trustee, note both and weight the most-recent-action source as current.
- If the registration expiration date is more than a year past today and the last action date is also old, the snapshot is likely stale. Say so.
- Per the 2023 rule change, registration certificates with expiration dates after January 31, 2023 are automatically extended four years, so an "expired" date in a snapshot doesn't necessarily mean the registration lapsed.
- **Address conflicts between sources** (e.g., one source shows the formation state, another shows the operating address): both can be correct for the same LLC. The formation state appears on state SOS filings and political donation records; the operating address appears on FAA filings and corporate correspondence. Reconcile rather than picking one.
- **"SALE REPORTED" as owner**: this is an FAA placeholder, not an entity. Do not search for it as a company. Note as a transition state and look at registration history for the seller.
- **Generic Mode S codes** (ABC123 or similar): tracking sites display these for dormant or unassigned aircraft. The "flight history" attached is global noise from many aircraft sharing the placeholder, not real activity for the target.

## Confidence rubric

End every dossier with a confidence call:

- **High**: Owner is a named real person or named institution with a verifiable public record. Multiple sources agree.
- **Medium**: Owner is an LLC with at least one traceable principal. The ownership chain reaches a real person.
- **Low**: Owner is an LLC and no principals are locatable. Address may be a registered agent service. Trail goes cold.
- **Withheld / opaque by design**: Owner data is withheld under 49 USC § 44114(b), the aircraft is held in a non-citizen trust with an undisclosed beneficiary, or the aircraft is on LADD/PIA suppression with no public tracking.

## Search practices

- Be resourceful. 5 to 15 searches is normal. If a search returns nothing useful, reformulate. Distinctive terms only; do not use quoted exact-match strings unless you need them.
- Useful patterns: `"[N#####]" flightaware`, `"[N#####]" airport-data`, `"[entity name]" [state] LLC`, `"[address]"`, `"[person name]" [entity name]`, `"[entity name]" site:opencorporates.com`, `"[person name]" Epstein`, `"[N#####]" NTSB`, `"[N#####]" accident`, `"[entity name]" donations OR contributions`, `"[entity name]" SAM.gov`.
- Don't stop at the first result. Cross-reference across multiple sources.
- If after several reformulations the aircraft still doesn't surface, say so. Don't fabricate. "Not found in indexed sources" is a valid finding.

## Output format

Present findings as a structured markdown dossier:

```
# Aircraft Research: [N-number]

## Aircraft Details
- **Registration**: [N-number]
- **Type**: [manufacturer and model]
- **Year**: [year from FAA-derived source]
- **Serial / C/N**: [serial]
- **Mode S code**: [hex if known, or note if placeholder]
- **Status**: [active / assigned / deregistered / sale reported / etc.]
- **Operating mode**: [FAR Part 91 / 135 / 121 / 137 if known]
- **Last action / expiration**: [dates, with note if stale]

## Registered Owner
- **Entity**: [name]
- **Address**: [address]
- **Entity type**: [LLC / Corp / Trust / Individual / Institution]
- **State of formation**: [state, date if found]
- **LEI**: [if present]
- **Pattern**: [single-asset LLC / operating company / institutional / trustee / leasing / individual]

## Operator (if distinct)
- [Operating party, if different from owner]
- [Home base airport if identifiable]

## Registration History
- [Brief history if material, with dates. Include prior foreign regs if any.]

## Ownership Chain
[Narrative from the registered entity to real people, when applicable.
Include each layer: the LLC, who filed it, who the principals are,
what other businesses they run. For institutional owners, describe the mission and fleet instead.
For trustee or LADD-blocked aircraft, explicitly state the opacity.]

## Flight Pattern (if available)
[Brief note on recent route pattern if it informs the mission assessment.]

## Flags
[List any flags triggered. If none, say "No flags triggered."]

## Assessment
[2 to 3 sentences on what the aircraft is likely used for and who ultimately controls it.]

## Confidence
[High / Medium / Low / Withheld]

## Sources
[Key sources and searches, with URLs where available.]
```

Adapt the template as needed. Not every section applies to every aircraft. For institutional owners, skip the Ownership Chain section and substitute a fleet/mission note. For trustee or LADD-blocked structures, the Assessment should explicitly acknowledge opacity.

## Important notes

- Stick to publicly available information.
- Be clear about confidence levels. If you are inferring, say so. If you hit a dead end, say that too.
- Don't over-speculate. "Likely" and "suggests" are fine. Don't present inferences as facts.
- If the ownership chain is deliberately opaque (trustee, withheld data, LADD/PIA), the opacity itself is the finding.
- FAA-derived data is authoritative on year, model, serial, and current registrant.
- Commercial registered agents (CT Corp, CSC, Cogency, Northwest, National, Capitol, InCorp) cap the free-web confidence ceiling at Low for principals.
