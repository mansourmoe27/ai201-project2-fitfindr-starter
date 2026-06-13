# FitFindr

FitFindr is an AI-powered fashion assistant that helps users discover secondhand clothing, generate outfit recommendations, and create social-media-ready outfit captions. The agent uses multiple tools and a planning loop to search listings, style outfits, and generate fit cards based on the user's wardrobe.

## Tool Inventory

### 1. search_listings(description, size, max_price)

**Purpose:**
Searches the mock secondhand listings dataset and returns relevant clothing items.

**Inputs:**

* `description (str)` – keywords describing the item the user wants
* `size (str | None)` – optional size filter
* `max_price (float | None)` – optional maximum price filter

**Returns:**

* `list[dict]` containing matching listing objects sorted by relevance

**Failure Handling:**

* Returns an empty list if no listings match the criteria.

---

### 2. suggest_outfit(new_item, wardrobe)

**Purpose:**
Generates outfit recommendations using the selected item and the user's wardrobe.

**Inputs:**

* `new_item (dict)` – selected listing returned by search_listings
* `wardrobe (dict)` – user's wardrobe

**Returns:**

* `str` containing one or more outfit suggestions

**Failure Handling:**

* If the wardrobe is empty, returns general styling advice instead of failing.

---

### 3. create_fit_card(outfit, new_item)

**Purpose:**
Generates a short social-media-style caption based on the outfit recommendation.

**Inputs:**

* `outfit (str)` – outfit suggestion from suggest_outfit
* `new_item (dict)` – selected listing

**Returns:**

* `str` containing a shareable fit card caption

**Failure Handling:**

* Returns a descriptive error message if the outfit string is empty.

---

## Planning Loop

The agent follows a multi-step workflow.

1. Parse the user query.
2. Extract description, size, and maximum price.
3. Call search_listings().
4. If no listings are found:

   * Set an error message.
   * Stop execution.
5. Select the top search result.
6. Call suggest_outfit() using the selected item and wardrobe.
7. Call create_fit_card() using the outfit suggestion.
8. Return the completed session.

The agent changes behavior based on search results. If no items are found, it does not continue to the outfit or fit card steps.

## State Management

The application uses a session dictionary to store information throughout the workflow.

Stored values include:

* Original query
* Parsed query values
* Search results
* Selected item
* Wardrobe
* Outfit suggestion
* Fit card
* Error messages

State is passed between tools through the session dictionary. For example:

search_listings() → selected_item → suggest_outfit() → outfit_suggestion → create_fit_card()

This allows information from earlier tools to be reused later without requiring the user to re-enter it.

## Error Handling

### search_listings()

Failure Mode:

* No matching items found

Response:

* Returns an empty list
* Agent displays:
  "I couldn't find any listings that match your request. Try loosening the price, removing the size filter, or using broader keywords."

### suggest_outfit()

Failure Mode:

* Empty wardrobe

Response:

* Generates general styling advice instead of outfit combinations.

### create_fit_card()

Failure Mode:

* Missing outfit suggestion

Response:

* Returns:
  "I couldn't create a fit card because the outfit suggestion is missing."

## Example Interaction

User Query:

"vintage graphic tee under $30"

Step 1:

* search_listings() finds matching items.

Step 2:

* The top result is selected.

Step 3:

* suggest_outfit() generates outfit recommendations using the user's wardrobe.

Step 4:

* create_fit_card() generates a social media caption.

Final Output:

* Listing details
* Outfit recommendation
* Fit card caption

## Spec Reflection

The planning document helped define the workflow before implementation and made it easier to build the planning loop.

One difference between the original design and the final implementation was query parsing. Initially, multiple approaches were considered, including using an LLM. The final implementation uses regular expressions because they are faster, simpler, and easier to debug.

## AI Usage

### Example 1

I used ChatGPT to help implement the search_listings() function.

Input Provided:

* Tool specification from planning.md
* Required inputs and outputs
* Failure handling requirements

Output:

* Python implementation for filtering, scoring, and sorting listings.

Verification:

* Tested multiple queries and verified correct filtering behavior.

### Example 2

I used ChatGPT to help implement the planning loop in agent.py.

Input Provided:

* Planning Loop section
* State Management section
* Agent architecture diagram

Output:

* Session-based workflow connecting all three tools.

Verification:

* Tested both successful and failure scenarios.
* Confirmed state passed correctly between tools.

## Technologies Used

* Python
* Gradio
* Groq API
* Llama 3.3 70B Versatile
* JSON datasets
* dotenv

## Demo Scenarios

### Successful Query

vintage graphic tee under $30

### No Results Query

designer ballgown size XXS under $5

### Empty Wardrobe Query

vintage graphic tee under $30 with Empty Wardrobe selected
