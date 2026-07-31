# Converting Taste & “Feel” into Executable Acceptance Criteria

Subjective observations are high-value and easy to write badly.  
Turn “feels off / cheap / confusing / not premium” into criteria an implementer and `/solve` reviewer can verify without further product judgment.

## First strategy (always)

**Match an existing good surface in the same product** before inventing numbers.

> “Match the card hierarchy and spacing already used on the billing overview page.”

Only invent token names, rem values, or ms durations when no good sibling exists.

---

## Conversion patterns

### 1. Visual hierarchy & dominance

**Observation:** “The important number doesn’t stand out.”  

**Concrete AC:**

- Primary metric uses font-size 3rem (or token `text-4xl`), font-weight 700, and is the darkest text on the card.
- Surrounding padding reduced from 24px to 20px so the number dominates the card.
- Secondary labels use `text-muted` and smaller size.
- Match the hierarchy already used on the billing overview page.

### 2. Spacing & density

**Observation:** “Everything feels cramped / sparse.”  

**Concrete AC:**

- Consistent vertical rhythm of 16px / 24px between major sections (existing spacing scale).
- Card internal padding set to the design-system value already used on settings cards.
- No two interactive elements closer than 8px.

### 3. Motion & transitions

**Observation:** “The modal just pops in; it feels abrupt.”  

**Concrete AC:**

- Enter: 200 ms ease-out, scale 0.96 → 1, opacity 0 → 1.
- Exit: reverse of enter.
- Use the same transition already present on the confirm-delete dialog (or shared `Modal` component).

### 4. Loading & perceived performance

**Observation:** “It feels slow / janky when data loads.”  

**Concrete AC:**

- Show a skeleton that matches the final layout shape while data is loading.
- No layout shift (CLS) when content arrives.
- Skeleton uses the same border-radius and spacing as the real content.

### 5. Interaction feedback

**Observation:** “Buttons don’t feel responsive.”  

**Concrete AC:**

- Hover state darkens background by the existing hover token or 8% opacity overlay.
- Active/pressed state scales to 0.98 or uses the design-system pressed style.
- Focus ring is visible and uses the brand focus color.

### 6. Emotional / brand tone

**Observation:** “Doesn’t feel luxurious / trustworthy / calm.”  

**Concrete AC (pick measurable levers):**

- Increase whitespace around primary actions to match marketing site cards.
- Soften shadows or reduce border contrast to match quieter cards elsewhere.
- Replace harsh red error text with the softer destructive token already used.
- Prefer the darker teal primary (token or exact hex already in theme) for the main CTA.

### 7. Copy & micro-interactions

**Observation:** “The empty state feels cold / unhelpful.”  

**Concrete AC:**

- Empty state shows [illustration or icon] + short headline + one-sentence explanation + primary CTA.
- Exact suggested copy: “No projects yet. Create your first project to get started.”
- CTA uses the primary button variant.

---

## Bad AC vs good AC

| Bad (do not file) | Good (file) |
|-------------------|-------------|
| Make the dashboard feel premium | Primary KPI uses `text-4xl`/`font-bold`; secondary metrics `text-sm`/`text-muted`; match `/billing` overview hierarchy |
| Improve UX of settings | Group “Profile” and “Security” into separate cards with 24px section gap; primary Save remains sticky footer like `/team` settings |
| Polish the modal | Modal enter 200ms ease-out opacity+scale; reuse `Dialog` from `components/ui/dialog` |
| Better empty state | Empty state: icon + “No invoices yet” + “Invoices appear after your first charge” + primary “View billing docs” |

---

## Rules of thumb

1. Prefer “match the pattern already used on [existing good surface]” over inventing new values.
2. When inventing values, give exact numbers, durations, easing, or token names.
3. Always keep a **do not change** list so the implementer does not “improve” unrelated elements.
4. Pin the owning route + component in the code map even for pure visual work.
5. **One conversion attempt:** if still subjective after one serious pass → **drop** the finding or escalate once to the user for intent. Do not file fuzzy tickets.

---

## Escalation prompt (rare)

Only if the finding is high-impact and still ambiguous:

> “On [surface], hierarchy feels wrong. Should primary emphasis be [A metric] or [B CTA]? I’ll file a concrete ticket once you pick.”

Then stop inventing brand direction without an answer.
