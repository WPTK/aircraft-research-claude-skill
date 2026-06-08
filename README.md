# Aircraft Research OSINT Claude Skill

An [Anthropic Agent Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) that turns Claude into an open-source aircraft ownership researcher. Give it a tail number, an ADS-B screenshot, or pasted registry text, and it traces the ownership chain from the FAA record down to real people and a likely purpose, then returns a structured dossier.

This is N-number (US) focused. Non-US registrations are supported with a different source set, but the deep workflow assumes US registry data.

## What it does

The skill walks a layered investigation rather than a single lookup:

- **Registration baseline**, pulled from indexed third-party mirrors:
  - Owner name and address
  - Type, year, serial number
  - Status (active, deregistered, sale reported, reserved)
  - Operating mode (FAR Part 91 / 135 / 121 / 137)
  - Mode S hex code
  - Last-action and expiration dates
- **Registration and incident history:**
  - Prior owners and how long the current one has held it
  - Leasing or trust pass-throughs
  - Foreign-registry history
  - NTSB and Aviation Safety Network hits
- **Owner classification**, which sets the depth for everything after:
  - Named individual
  - Single-asset LLC
  - Operating company
  - Institution
  - Trustee
  - Leasing company
- **Entity and principal tracing:**
  - State SoS filings, OpenCorporates, registered agents
  - Single-asset-LLC test (web presence, linked aviation entities, formation timing)
  - Confidence drops to a Low ceiling when a commercial registered agent (CT, CSC, Cogency, Northwest) hides the principal
- **Network and public-record overlay:**
  - Other businesses and roles
  - Political donations (FEC, OpenSecrets, state databases)
  - Government contracts (SAM.gov, USAspending)
  - LEI presence
  - OFAC SDN sanctions check on each named principal and entity
  - Epstein cross-reference once real people are named (contact book, flight logs, released emails), disambiguated by city or employer
- **Purpose and pattern analysis:**
  - Flight-pattern reading (survey orbits, hub-and-spoke medical, executive city pairs)
  - Home-base inference, cross-checked against the registered address
  - A synthesis of what the airframe is actually used for

Every dossier ends with a confidence call (High / Medium / Low / Withheld) and treats deliberate opacity (non-citizen trust, withheld data under 49 USC § 44114(b), LADD/PIA suppression) as a finding in itself rather than a failure.

## How it works

An Agent Skill is a folder containing a `SKILL.md` file. The YAML frontmatter holds a `name` and a `description`; the description is what Claude uses to decide when to load the skill, and the body is the instruction set Claude follows once it triggers. There is no code to run. The intelligence is in the prompt.

This skill leans heavily on web search. The FAA registry web form requires JavaScript and cannot be fetched directly, so the workflow is **search before fetch**: it queries indexed mirrors (including localized FlightAware subdomains that surface gated registration records in search snippets) and cross-references across sources instead of trusting any single one.

## Requirements

- A Claude surface that supports Agent Skills and has web search available (for example Claude Code, or the API with the skills capability enabled).
- No API keys, dependencies, or build step inside the skill itself.

## Usage

Once the skill is loaded, just ask in natural language. It triggers on ownership-research intent, not on a command.

```
Who owns N628TS?
```

```
Here's an ADS-B Exchange screenshot. Trace the owner.
```

```
Look up this tail number and tell me what it's used for: N12345
```

It will not engage for general aviation trivia, performance specs, or plain flight tracking with no ownership angle.

## Output

Findings come back as a markdown dossier. The template:

```
# Aircraft Research: [N-number]

## Aircraft Details
## Registered Owner
## Operator (if distinct)
## Registration History
## Ownership Chain
## Flight Pattern (if available)
## Flags
## Assessment
## Confidence
## Sources
```

Sections that don't apply are dropped. Institutional owners get a fleet/mission note instead of an ownership chain; trustee and LADD-blocked structures get an Assessment that states the opacity outright.

## Scope and limits

- US N-numbers are the primary target. Cayman (VP-C), Isle of Man (M-), Aruba (P4-), San Marino (T7-), and Bermuda (VP-B) registries are noted as privacy signals but not deeply mined.
- Free-web sources only. Behind a commercial registered agent, the principal layer caps at Low confidence unless other public records name them.
- FAA-derived data is treated as authoritative on year, model, serial, and current registrant; press releases and broker listings lose ties on those fields.
- "Not found in indexed sources" is a valid result. The skill is instructed not to fabricate ownership data.

## Responsible use

Be a good grown up.

## License

CC BY-SA 4.0.
