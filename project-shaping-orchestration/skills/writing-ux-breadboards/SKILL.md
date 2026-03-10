---
name: shaping-writing-ux-breadboards
version: 1.0.0
user-invocable: false
description: Produce UX Breadboard documents that map user flows, screen states, error states, empty states, navigation maps, and interaction affordances using text-based breadboarding notation. Use when shaping a feature that involves user-facing interaction and the team needs to reason about flow before visual design. Triggered by: ux breadboard, user flows, screen states, error states, empty states, navigation map, interaction design, affordances, breadboarding.
allowed-tools:
  - Read
---

# UX Breadboards — Flows, States, Errors & Navigation Maps

Expert guidance for producing UX breadboard artifacts during the shaping process.

## Purpose

A UX Breadboard captures the interaction design of a feature using text notation instead of wireframes. It maps every place a user can be, every affordance they can interact with, and every connection between places — including error states, empty states, and edge cases.

Use this skill when a shaped feature needs its user-facing flows defined before visual design or implementation begins. The breadboard is the canonical reference for what the user experiences, decoupled from how it looks.

Breadboarding comes from Shape Up: it strips away visual design so the team can focus on flow, completeness, and interaction logic without getting distracted by layout, color, or typography.

---

## Output Format

```markdown
# UX Breadboard: <Feature Name>

**Date:** <YYYY-MM-DD>
**Problem Frame:** <reference to the Problem Frame this breadboard addresses>
**Domain Scenarios:** <reference to the Domain Scenarios this covers>

---

## 1. Flow Diagrams

### <Flow Name> (e.g., "Create New Widget")

> Brief description of what the user is trying to accomplish in this flow.

STARTING_PLACE
- information: <what the user sees here>
- [affordance_1] -> NEXT_PLACE
- [affordance_2] -> ANOTHER_PLACE
- [cancel] -> PREVIOUS_PLACE

NEXT_PLACE
- information: <what the user sees here>
- [affordance_3] -> CONFIRMATION_PLACE
- [back] -> STARTING_PLACE

CONFIRMATION_PLACE
- information: <success message or result>
- [affordance_4] -> STARTING_PLACE

### <Flow Name 2> (e.g., "Edit Existing Widget")

> Brief description of this flow.

<... same notation ...>

---

## 2. Error States

| Place | Error Condition | What User Sees | Recovery Action |
|-------|----------------|----------------|-----------------|
| STARTING_PLACE | Network unavailable | "Unable to load. Check your connection." | [retry] -> STARTING_PLACE |
| NEXT_PLACE | Validation failure | Inline field errors with specific messages | User corrects fields, [submit] again |
| NEXT_PLACE | Server rejection (409) | "This name is already taken." | User changes name, [submit] again |
| CONFIRMATION_PLACE | Timeout | "Save is taking longer than expected." | [retry] -> NEXT_PLACE |

---

## 3. Empty States

| Place | When Empty | What User Sees | Primary Action |
|-------|-----------|----------------|----------------|
| STARTING_PLACE | No items exist yet | "No widgets yet. Create your first one." | [create widget] -> NEXT_PLACE |
| STARTING_PLACE | Search returns nothing | "No results for '<query>'. Try different terms." | [clear search] -> STARTING_PLACE |
| DETAIL_PLACE | Item has no history | "No activity yet. Actions will appear here." | None (informational) |

---

## 4. State Inventory

| Place | Possible States | Transitions | Data Required |
|-------|----------------|-------------|---------------|
| STARTING_PLACE | loading, empty, populated, error | loaded -> populated; failed -> error | List of items, count |
| NEXT_PLACE | pristine, dirty, submitting, error | edited -> dirty; submitted -> submitting | Form fields, validation rules |
| CONFIRMATION_PLACE | success, partial-success | completed -> success | Result summary |

---

## 5. Navigation Map

ENTRY_POINT
├── STARTING_PLACE
│   ├── DETAIL_PLACE
│   │   ├── EDIT_PLACE
│   │   └── DELETE_CONFIRMATION
│   └── CREATE_PLACE
│       └── CONFIRMATION_PLACE
└── SETTINGS_PLACE

Reachable from global navigation: STARTING_PLACE, SETTINGS_PLACE
Deep-linked: DETAIL_PLACE (via URL with item ID)

---

## 6. Open Interaction Questions

1. <Question about a UX decision deferred to implementation>
2. <Question about edge case behavior not yet decided>
3. <Question about progressive disclosure or phasing>
```

---

## Instructions

1. **Read Domain Scenarios to identify all user workflows.** Extract every distinct user goal that involves interacting with the system. Each goal becomes a flow. Pay attention to alternate paths, not just the happy path. Note the personas and their mental models — these inform how places should be named and what information should be displayed.

2. **Map each workflow to PLACES using breadboard notation.** A PLACE is any distinct screen, page, dialog, or panel the user occupies. Name places in UPPER_CASE using nouns that describe what the user sees, not implementation details. Every flow must have a clear starting place and at least one ending place. Keep place names stable across flows — if two flows visit the same screen, use the same place name.

3. **Identify affordances at each place and where they lead.** An affordance is anything the user can interact with: buttons, links, form fields, toggles. Write them in [brackets]. Every affordance must point to a destination PLACE using -> arrows. Also note non-interactive information displayed at each place. If an affordance changes state within the current place (rather than navigating), note that in the State Inventory instead.

4. **Enumerate error and empty states for every place.** For each place, ask: "What happens when this fails?" and "What happens when there is no data?" Fill in the Error States and Empty States tables completely. Every place that loads data must appear in the Error States table. Every place that displays a list or collection must appear in the Empty States table.

5. **Build the navigation map showing how places connect.** Draw the hierarchy of places as a text tree. Note which places are reachable from global navigation, which require deep linking (URL with ID), and which are only reachable through specific flows. Verify that every place in the flow diagrams appears in the navigation map.

---

## Quality Checklist

- [ ] Every PLACE has at least one affordance or is explicitly marked as terminal (e.g., success confirmation)
- [ ] Every affordance has a destination — no dead-end interactions
- [ ] Every place appears in the Error States table with at least one error condition
- [ ] Every list-type place appears in the Empty States table
- [ ] The State Inventory covers all places with their possible states and transitions
- [ ] The Navigation Map is consistent with the flow diagrams — no orphan places
- [ ] Open Interaction Questions capture genuine UX decisions, not implementation details

---

## Anti-Patterns

### 1. Visual Design Creep
Breadboards describe flow and interaction, not layout or styling. If you catch yourself specifying column widths, colors, or component types, stop. The breadboard is deliberately abstract — it says "the user sees a list of items" not "a data table with sortable columns and a blue header."

### 2. Implementation Leakage
Place names should reflect what the user sees, not technical structure. Use INVOICE_LIST, not INVOICES_INDEX_PAGE. Use CREATE_WIDGET, not POST_WIDGET_FORM. The breadboard is a user-mental-model document, not a route map.

### 3. Happy-Path-Only Flows
A breadboard that only shows the success path is incomplete. Error states and empty states are first-class elements. If the Error States table is sparse, the breadboard is not finished. Real users encounter errors and empty screens more often than designers expect.

### 4. Affordance Without Destination
Every bracketed affordance must have an arrow pointing somewhere. If clicking a button does not navigate the user to a new place, it either changes state within the current place (note that in the State Inventory) or it is not an affordance — it is information display.

### 5. Conflating Places and States
A loading spinner is not a separate PLACE; it is a state of an existing place. Use the State Inventory to capture within-place state changes. Reserve PLACES for genuinely distinct screens the user can be "at." If you can deep-link to it, it is a place. If you cannot, it is a state.

---

## Examples

### Example 1: Team Invitation Flow

**Context:** A workspace owner invites a new team member by email.

```
TEAM_MEMBERS_LIST
- information: table of current members (name, role, status)
- [invite member] -> INVITE_DIALOG
- [member row] -> MEMBER_DETAIL

INVITE_DIALOG
- information: email input field, role selector
- [send invitation] -> TEAM_MEMBERS_LIST (with success toast)
- [cancel] -> TEAM_MEMBERS_LIST

MEMBER_DETAIL
- information: member name, email, role, join date, activity
- [change role] -> ROLE_SELECTOR (inline)
- [remove member] -> REMOVE_CONFIRMATION
- [back] -> TEAM_MEMBERS_LIST

REMOVE_CONFIRMATION
- information: "Remove [name] from workspace? They will lose access immediately."
- [confirm remove] -> TEAM_MEMBERS_LIST (with success toast)
- [cancel] -> MEMBER_DETAIL
```

**Error States:**

| Place | Error Condition | What User Sees | Recovery Action |
|-------|----------------|----------------|-----------------|
| INVITE_DIALOG | Invalid email format | Inline: "Enter a valid email address" | User corrects email |
| INVITE_DIALOG | User already a member | "This person is already a member." | [dismiss] stays on dialog |
| INVITE_DIALOG | Seat limit reached | "Your plan allows 5 members. Upgrade to add more." | [upgrade plan] -> BILLING |

### Example 2: API Key Lifecycle

**Context:** A developer creates, views, and revokes API keys.

```
API_KEYS_LIST
- information: table of keys (name, prefix, created date, last used, status)
- [create key] -> CREATE_KEY
- [key row] -> KEY_DETAIL

CREATE_KEY
- information: name input, expiration selector, scope checkboxes
- [generate key] -> KEY_REVEAL
- [cancel] -> API_KEYS_LIST

KEY_REVEAL
- information: full key value (shown once), copy button
- warning: "This key will not be shown again. Copy it now."
- [copy to clipboard] -> KEY_REVEAL (with copied confirmation)
- [done] -> API_KEYS_LIST

KEY_DETAIL
- information: key name, prefix, created, last used, scopes, status
- [revoke key] -> REVOKE_CONFIRMATION
- [back] -> API_KEYS_LIST

REVOKE_CONFIRMATION
- information: "Revoke [key name]? Applications using this key will stop working."
- [confirm revoke] -> API_KEYS_LIST (with success toast)
- [cancel] -> KEY_DETAIL
```

**Navigation Map:**

```
WORKSPACE_SETTINGS
└── API_KEYS_LIST
    ├── CREATE_KEY
    │   └── KEY_REVEAL
    └── KEY_DETAIL
        └── REVOKE_CONFIRMATION
```