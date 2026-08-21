# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: [name your flow]

### Step 1: Expand, build new screens in a strict sequence
```
Build the next phase of this app in a strict sequence:
1. Add a screen "Recruiter Insights". Match the layout and spacing with other screens
```

### Step 2: Behavior, hard-code the states
```
Apply the following logic constraints to the REQ details screen:
- List the Candidate details in a grid view 
- If no data is present, show the empty state: "No records found".
- On Candidate workspace screen help me navigate to the next record or previous record. Add Next and Previous button for navigation ease.

Maintain the same design language throughout and tether all behavior strictly to these rules.
```

### Step 3: Refine, one surgical polish
```
The Dashboard needs a polish.
1. Candidate Name and details sticks on the screen along with the Days in Pipeline, Time to Hire and Stage. Remove all 3

Don't change anything else in the project or touch the underlying logic.
```

## Reusable techniques learned

- _____
- _____

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

_____
