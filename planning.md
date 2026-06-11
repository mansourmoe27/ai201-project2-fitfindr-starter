# FitFindr — planning.md

> Complete this document before writing any implementation code.
> Your spec and agent diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Your planning.md will be reviewed as part of your submission.
> Update it before starting any stretch features.

---

## Tools

List every tool your agent will use. For each tool, fill in all four fields.
You must have at least 3 tools. The three required tools are listed — add any additional tools below them.

### Tool 1: search_listings

**What it does:**
Searches the mock secondhand listings for items that match the user's requested description, optional size, and optional maximum price. it returns matching listings sorted by revelence so the agent can chose the best item.

**Input parameters:**
- `description` (str): Keywords decribing what the user is looking for.
- `size` (str): ... Optional size filter. If porvided, matching should be case insenitive and flexable enough for values like "M" matching "S/M" or "M/L".
- `max_price` (float): Optional maximum price. If provided, only listings with price<= max_price should be returned

**What it returns:**
A list of lintings dictionaries. Each result may include 'id', 'title', 'description', 'category', 'style_tags', 'size', 'condition', 'price', 'colors', 'brand', and 'platform'. Results are sorted by keyword relevance.
**What happens if it fails or returns nothing:**
Return an empty list. The agent should stop the workflow, store an error message, and tell user no matching listings were found. It should suggest loosening the size, price, or decription instead of continuning to outfit generation.

---

### Tool 2: suggest_outfit

**What it does:**
Suggests one or two outfirs using the selected listing and the user's wardrobe.if the wardrobe has items, it should name specific wardrobe pieces. If the wardrobe is empty, it shuld give general styling adivce for selected item.
**Input parameters:**
- `new_item` (dict): Th listing selected from 'search_listings', including title, category, style tags, colors, price, and platform.
- `wardrobe` (dict): A wardrobe dictionary with an 'items' key containing a listof wardrobe item dictionaries.

**What it returns:**
A non-empty string decribing outfit combinations. For a non-empty wardrobe, it should reference specific wardrobe itemsby name. For an empty wardrobe, it should provide general styling guidance.
**What happens if it fails or returns nothing:**
If the wardrobe is empty, return general styling advice instead of crashing. If the LLM fails or return an empty string, return a fallback outfit suggestion based on the item's category, colors, and style tags.
---

### Tool 3: create_fit_card

**What it does:**
Creates a short, shareabele outfit caption based on the selected listing and outfit suggestion. The caption should sound casual and social media ready.
**Input parameters:**

- `outfit` (...): The outfit suggestion returned by 'suggest_outfit'.
- 'new_item' (dict): The selected listing dictionary from 'search_listings'.

**What it returns:**

A 2 to 4 sentence string that mentions the item name, price, platform, and outfit vibe naturally.

**What happens if it fails or returns nothing:**
If 'outfit' is missing or blank, return a description error message string. If the LLM fails, return a simple fallback caption using the title, price, platform, and outfit summary

---

### Additional Tools (if any)



---

## Planning Loop

**How does your agent decide which tool to call next?**
After search:
- If the returend results list is empty, the agent stores an error message in the session and stops. It does not call 'suggest_outfit' or 'create_fit_card'.
- If the wardrobe is empty, the tool still retruns general styling advice.
Finnaly, the agent calls 'create_fit_card(session["outfit_suggestion"] session ['selected_item"])'.
- If a fit card is created, it stores it in 'session["fit_card"]'.
- The loop is complete when the selected item, outfit suggestion, and fit card are stored, or when an error stops the workflow early.

Finnaly, the agent calls 'create-fitcard(session["outfit_suggestion"], session["selected_item"])'.
- If a fit card is created, it stores it in 'session[fit_card"];.
- The loop is complete when the selected item, outfit suggestion, and fit card are stored, or when an error stops the workflow early.
---

## State Management

**How does information from one tool get passed to the next?**
The session tracks:
- 'query': the original user request
- 'description' extracted item decription
- 'size' extrcted size, if any
- 'max_price' extracted price limit, if any
- 'search_result' list of matching listings
- 'selected_item': the chosen listing from search result
- 'wardrobe': the user wardrobe used for styling
- 'outfit_suggestion': output from 'suggest_outfit'
- 'fit_card': output from 'create_fit_card'
- 'error': error message if the workflow stops early

The slelected item from 'search_listings' becomes the impput to 'suggest_outfit'. The outfit suggestion form 'suggest_outfit' becomes the input to o'create_fit_card'.


---

## Error Handling

For each tool, describe the specific failure mode you're handling and what the agent does in response.

| Tool | Failure mode | Agent response |
|------|-------------|----------------|
| search_listings | No results match the query | Store an error in 'session ["error"]|, tell the user no matching listings were found, and suggest loosening the price,size,or decription. Stop the workflow early.|
| suggest_outfit | Wardrobe is empty |Return general styling advice for the selected item instead of using named wardrobe pieces, continue to fit card creation. |
| create_fit_card | Outfit input is missing or incomplete |Return a decriptive error message string and store it in 'session["error"]; if needed. Do not crash app.| |

---

## Architecture

```text
User Query
   |
   v
Planning Loop / run_agent()
   |
   |-- Extract description, size, max_price
   |
   v
search_listings(description, size, max_price)
   |
   |-- results == [] ----------------------------\
   |                                             |
   v                                             v
Session["search_results"]                  Session["error"]
Session["selected_item"] = results[0]       Return error to user
   |
   v
suggest_outfit(selected_item, wardrobe)
   |
   v
Session["outfit_suggestion"]
   |
   v
create_fit_card(outfit_suggestion, selected_item)
   |
   v
Session["fit_card"]
   |
   v
Return selected item + outfit suggestion + fit card
---

## AI Tool Plan


**Milestone 3 — Individual tool implementations:**
I will use ChatGTP to help implement each too one at a atime. for search_listings, i wil provide the Tool 1 spec and ask for code that uses load_listings(), filters by size and price, scores keyword overlap, and returns an empty list when no matches exist. i wil verify it with at least three direct terminal tests, including one no-results case.

For suggest_outfit, i wil provide the tTool 2 spec, the wardrobe schema and Groq client helper. I will verify that the function handles both example wardrobes and empty wardrobes without crashing.

For create_fit_card, i will povide the Tool 3 spec and verify that it returns a captin for valid outfit input and a clear error message for empty outfit input.

**Milestone 4 — Planning loop and state management:**
I will use ChatGTP with the Planning Loop, State Management, and Architecture section from this file. I expect it to help implement run_agent() so that the output of one tool is stored in session state and passed to the next tool. I will verify that agent does not call outfit orfit card tools when search returns no reslut, and that selected_item, outfit_suggestion, and fit_card are all stored in the final session for a successful query. 
---

## A Complete Interaction (Step by Step)



**Example user query:** 
I'm looking for a vintage graphic tee under $30. I mostly wear baggy jeans and chunky sneakers. What's out there and how would i style it?


**Step 1:**
The agent extracts:
- decription: "vintage graphic tee"
- size: None
- max_price: 30.0

It calls:
search_listings("vintage graphic tee", size=None, max_price=30.0)
The too searched listings.jason, filters out items over $30, secores keyword matching and retruns matching listings.

**Step 2:**
The agent checks the search results.
if results are empty, it returns an error message and stops.
if reults exist, it selects the listing and stores it in session["selected_item"].

**Step 3:**
The agent loads or receives the example wardrobe and calls:
suggest_outfit(selected_item, wardrobe)
The outfit tool suggest how to style the selected tee with wardrobe items such as baggy jeans and chunky sneakers.

**Step 4**
The agent stores the outfit suggestion in session["outfit_suggestion"], then call create_fit_card(outfit_suggestion, selected_item)
The fit card tool creates a short social media style caption.

**Final output to user:**
The user sees the selected listing, why it matched, one or two outfit suggestions, and a shareable fit card caption.
