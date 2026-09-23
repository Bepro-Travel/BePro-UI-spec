# Hotel Search Results — Behaviour Reference for AI Assistant

## 0. How to use this document

This document describes how the hotel search and results filters **actually behave**, based on the source code. It is written for an AI assistant that answers questions from users of the hotel search page (travel agents and end clients).

Guidelines for answering:

- Answer in terms of what the user sees and does (buttons, fields, results). Do not mention code, function names, or storage keys unless the user explicitly asks a technical question.
- If a question is not covered here, say that you are not sure instead of guessing.
- Many controls are available only to agents (B2B users) or only when the account has a specific permission. These are marked **[Agent only]** or **[Permission]**. If a user says they don't see a control, explain that it depends on their account settings.
- For a visual guide to the interface, see the page **Hotel Search UI Filters**.

---

## 1. Core concepts

### 1.1 Search parameters vs. result filters

There are two different kinds of controls:

|              | Search parameters                                            | Result filters                                          |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------- |
| Where        | Search bar at the top, including the expanded (advanced) row | Filter toolbar, left sidebar, Room AI tab, map controls |
| When applied | Only after clicking **GO** (starts a new search)             | Instantly, no new search                                |
| Effect       | Change what the server returns                               | Hide / show hotels and rooms that were already loaded   |

A result filter can never show a hotel that the search did not return. For example, if the search radius is 2 km, the "km" filter cannot show hotels 5 km away.

### 1.2 Hotels and room offers

- Each hotel card contains a list of **room offers** (room + board + price + cancellation policy + supplier).
- Some filters work on the **hotel** level (e.g. stars, hotel name, facilities). Others work on the **room offer** level (e.g. meal plan, price, free cancellation, room name).
- **A hotel is shown only if it passes all hotel-level filters AND has at least one room offer that passes all room-level filters.** Room offers that fail the filters are hidden inside the card.

### 1.3 Results load progressively

- After **GO**, results arrive in batches from several suppliers over a period of time. The hotel count, filter options, and the numbers in brackets grow while loading.
- Filters that the user already selected stay active and are applied to every new batch automatically.
- The same hotel coming from several suppliers is merged into **one** hotel card; its room offers are combined.
- For users who do **not** see supplier names, identical offers are collapsed: if several offers have the same room category, room type, board, bed type and cancellation status, only the **cheapest** one is shown.
- After 20 minutes on the results page, a message appears that the search is outdated, because prices and availability may have changed. If the server session expires, the search is re-run automatically.

### 1.4 How filters combine

- **Different filters are combined with AND.** For example, 4★ + BB + Free shows only 4-star hotels that have at least one room with breakfast and free cancellation.
- **Options inside the same filter are combined with OR.** For example, 4 + 5 stars shows 4-star **or** 5-star hotels.
- **Exception: Hotel facilities use AND.** Checking Wi-fi + Gym shows only hotels that have both.
- **Room AI:** options inside one category use OR, and different categories use AND.

| Filter                        | Level | Logic between selected options                  |
| ----------------------------- | ----- | ----------------------------------------------- |
| Stars (★ / 3 / 4 / 5)         | Hotel | OR                                              |
| Preferred hotels (👑)         | Hotel | —                                               |
| Distance (km)                 | Hotel | —                                               |
| Hotel name                    | Hotel | OR between fields, AND between words in a field |
| Hotel type                    | Hotel | OR                                              |
| Hotel facilities              | Hotel | **AND**                                         |
| Display exactly (map)         | Hotel | —                                               |
| Meal plan (BB / HB / FB / AI) | Room  | OR (see special BB rule in 3.4)                 |
| Free cancellation             | Room  | —                                               |
| Accessibility                 | Room  | —                                               |
| Price range                   | Room  | —                                               |
| Room name                     | Room  | OR between fields, AND between words in a field |
| Room filter (bed types)       | Room  | OR between included bed types                   |
| Room AI categories            | Room  | OR inside a category, AND between categories    |
| Suppliers                     | Room  | OR                                              |

---

## 2. Search parameters (search bar)

All of these require clicking **GO**.

| Parameter                   | Behaviour                                                                                                                                                                         |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **New search**              | Opens an empty search form and clears the saved page state.                                                                                                                       |
| **Radius** [Agent only]     | Search radius around the destination. The value is saved to the user's profile and becomes the default for the next searches. End clients (B2C) always search with a 1 km radius. |
| **Destination**             | City, address or a specific hotel. If the destination is a **specific hotel**, the results are sorted by distance by default, and the searched hotel is shown first.              |
| **Check-in / Check-out**    | Stay dates. The number of nights is calculated automatically.                                                                                                                     |
| **Rooms / Guests**          | Number of rooms and guests per room. This also affects which **Room filter** (bed type) options appear, see 4.4.                                                                  |
| **Stars**                   | Minimum star rating sent to the search (e.g. `1+`). The default is 3 or the user's saved setting.                                                                                 |
| **Expand (⌄)** [Agent only] | Shows the advanced row below.                                                                                                                                                     |

### Advanced search options [Agent only]

| Parameter                  | Behaviour                                                                                                                            |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **Nationality**            | Guest nationality. Suppliers can return different prices and availability by nationality. The default comes from the user's profile. |
| **Suppliers** [Permission] | Free-text field to limit the search to specific suppliers. Visible only for specific branches.                                       |
| **Display Exactly**        | Searches only inside the boundaries (polygon) of the destination locality.                                                           |
| **Free cancellation**      | The search returns only offers with free cancellation. Other offers are not loaded at all.                                           |

> **Tip for users:** The _Free cancellation_ option in the search limits what is loaded. The **Free** button in the toolbar only hides already loaded offers and can be switched off again without a new search.

---

## 3. Filter toolbar

### 3.1 View mode (list icon) [Agent only]

- Opens a menu of display modes (Client, Agent…, Business…). The selected mode is shown in bold.
- A display mode changes how cards look: whether photos are shown, and how room offers are grouped (by boarding and category, or by category and type, with one or several prices).
- **"Single Row For Room"** shows each room offer as a **separate card**. In this mode, sorting applies to individual room offers, not to hotels.
- The selected mode is saved to the user's profile and stays after reload and in future searches.

### 3.2 Map

Switches between the hotel list and the map.

### 3.3 Sort

| Option                             | Behaviour                                                                                                                                      |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Recommended** (default)          | Preferred hotels (👑) first, with higher priority first. Then all hotels by cheapest room price, low to high.                                  |
| **Price: low → high / high → low** | By the cheapest **currently visible** room offer of each hotel. Prices are compared in USD, so the order doesn't change with display currency. |
| **Star: low → high / high → low**  | By hotel star rating.                                                                                                                          |
| **Name: A–Z / Z–A**                | Alphabetical by hotel name.                                                                                                                    |
| **Distance: 0–100 / 100–0**        | By distance from the reference point: the destination center, or the point chosen in the **Location** field.                                   |

Rules:

- If the destination is a specific hotel, the default sort is **Distance** and that hotel stays first.
- When the user chooses a point in the **Location** field, the sort automatically switches to **Distance: 0–100**.
- When filtering by hotel name, hotels whose name contains the typed word as a whole word are moved to the top.
- Inside a hotel card, visible room offers are always ordered from cheapest to most expensive.
- Pinned hotels (📌) are always shown above the list, whatever the sort.

### 3.4 Quick toggles

| Control               | Behaviour                                                                                                                                                                                                                                         |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **♿ Accessibility**  | Shows only room offers marked as accessible. It checks all _Accessibility_ options in Room AI at once, so both controls stay in sync. Disabled (greyed out) when no accessible rooms were found. The tooltip shows how many rooms are accessible. |
| **⚠ Free**            | Free cancellation filter for loaded results. First click: only offers whose cancellation deadline has **not passed yet** (plus offers without a deadline date). Second click: all offers again. For some accounts this is **on by default**.      |
| **👑 Crown**          | Shows only preferred hotels (hotels that have a recommendation priority).                                                                                                                                                                         |
| **BB / HB / FB / AI** | Meal plan filter, see below.                                                                                                                                                                                                                      |

Meal plan rules:

- **BB** = Bed & Breakfast, **HB** = Half Board, **FB** = Full Board, **AI** = All Inclusive.
- Several can be selected. They are combined with OR.
- **Special rule: selecting BB shows every offer that includes any meals** (BB, HB, FB, AI), meaning "at least breakfast". Only Room Only offers are hidden.
- HB, FB and AI show only offers with exactly that board.
- A button is disabled when no offers with that board were found.

### 3.5 Stars

| Button        | Behaviour                                   |
| ------------- | ------------------------------------------- |
| **★**         | Shows **2-star** hotels.                    |
| **3 / 4 / 5** | Shows hotels with exactly that star rating. |

- Several can be selected (OR).
- Half stars are rounded down: a 4.5★ hotel counts as 4★.
- A button is disabled when there are no hotels with that rating in the results.
- There is no button for 1-star or unrated hotels. To exclude them, use the minimum **Stars** parameter in the search.

### 3.6 Location & distance

| Control            | Behaviour                                                                                                                                                                                                               |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **km**             | Shows only hotels within the selected distance from the reference point. Options: 1, 2, 3, 5, 8, 13, 20 km.                                                                                                             |
| **Location** field | Choose a reference point (e.g. an office, venue or landmark). Two input modes: **pin** (search places by name) and **arrow** (search by street address). Suggestions are limited to about 50 km around the destination. |

After a point is selected:

1. Distances of all hotels are recalculated from that point (the distance shown on each card changes).
2. The list is re-sorted by distance, nearest first.
3. The **km** filter is reset. Select it again if needed.

> **Known behaviour:** Clearing the Location field does not recalculate distances back to the destination center. Distances stay measured from the last selected point until a new search.

### 3.7 Suppliers (list icon near Reg) [Agent only] [Permission]

- Visible only when the account is allowed to see supplier names.
- A table of suppliers for the current search:

| Column       | Meaning                                                                                                                                                   |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Supplier** | Supplier name with a checkbox.                                                                                                                            |
| **Hotels**   | Number of hotels this supplier returned.                                                                                                                  |
| **Rooms**    | Number of room offers this supplier returned.                                                                                                             |
| **Price**    | Lowest price from this supplier.                                                                                                                          |
| **Total**    | Sums of the columns. A hotel offered by several suppliers is counted once per supplier, so the total can be higher than the number of hotels in the list. |

- Checking suppliers shows only their offers (OR). **Clear** unchecks all.

### 3.8 Price type (Reg / Net / Net.Т) [Agent only] [Permission]

Changes which price is **displayed**. It doesn't change filters or sorting.

| Option            | Meaning                         | Shown when                            |
| ----------------- | ------------------------------- | ------------------------------------- |
| **Reg** (default) | Selling price including markup. | Always (when the selector is visible) |
| **Net** (purple)  | Supplier price without markup.  | User has Net permission               |
| **Net.Т** (green) | Net price after discount.       | User has discount permission          |

The selection is remembered for the current search.

### 3.9 Currency [Agent only]

- Display currency: USD, EUR, GBP. Some accounts also have THB.
- The default is USD.
- The selection is remembered for the current browser session.
- Changing currency does not re-calculate an already-set price range in the sidebar. If a price range is set, clear it after changing currency.

### 3.10 Clear filters (funnel with ✕)

- Highlighted when any result filter is active.
- Resets all result filters to defaults. **Free** returns to the account default.
- Does **not** reset sort, currency, price type, display mode or pinned hotels.

### 3.11 Keep filters for next search (funnel with 🔒) [Agent only]

- **On (highlighted):** the current result filters are kept and applied automatically to the next search.
- **Off (default):** every new search starts with clean filters.
- Works within the current browser tab/session.

---

## 4. Left sidebar

Tabs: 🛏 **Search filters**, 🏢 **Hotel facilities**, 🕘 **History** (latest searches), 🤖 **AI** (not available yet).

### 4.1 Hotel name

- Filters hotels by name. The match is case-insensitive and partial (a part of a word is enough).
- **Several words in one field = AND.** For example, `ibis mitte` matches "Ibis Styles Hotel Berlin Mitte".
- **Click + to add another field (up to 3) = OR.** For example, `ibis` | `hilton` shows both brands.
- The filter applies about half a second after typing stops.

### 4.2 Rooms (room name)

- Same rules as hotel name (words = AND, up to 3 fields = OR), applied to **room offer names**.
- The searched room name also includes the board description, so e.g. `breakfast` matches offers with breakfast.
- Hotels without any matching room are hidden.

### 4.3 Prices

- Range from the cheapest to the most expensive room offer in the results, in the display currency.
- **Min / Max** fields or slider. An empty **Max** means no upper limit.
- Applied to each room offer's total price for the stay.

### 4.4 Hotel type

- Hotel / Apartment / Villa. Only types present in the results are shown, with the number of properties in brackets.
- Several can be selected (OR).

### 4.5 Room filter (bed types)

- The list of options is configured per company and **depends on the number of guests per room** in the search. It can differ between searches.
- Selecting an option shows rooms with matching bed types and **automatically excludes conflicting ones** (e.g. rooms marked "no double bed").
- Options with an arrow (⌄) contain sub-options. Checking the parent selects all of them.
- **Extra bed** matches rooms with an extra bed, a sofa bed, or 3 beds.

### 4.6 Hotel facilities

- Shows only hotels that have **all** checked facilities (AND).
- The number in brackets is how many hotels in the results have that facility. Facilities with 0 hotels are disabled.
- **Groups (e.g. Parking & Transport):** checking the group checks all its sub-facilities, so a hotel must have **every** sub-facility. The number next to a group counts hotels that have **at least one**. Because of this, the result can contain fewer hotels than the group number shows. To find hotels with any one of them, expand the group and check a single sub-facility.
- Hotels with no facility information are hidden as soon as any facility is checked.

---

## 5. Room AI tab

- Room offers are described by attributes that AI extracts from room names and descriptions. Categories: **View, Accessibility, Balcony, Single use, Category, Capacity, Type, Bedroom, Boarding, Pax, Bed**.
- The number next to a category is how many options it has. The number next to an option is how many room offers have it. Hover over an option to see its lowest price.
- Options in **bold** are the most common / recommended ones.
- Logic: OR inside a category, AND between categories. For example, View: city + Type: suite shows suites with a city view.
- The **Search** box only filters the list of options. It does not filter hotels.
- Each category shows 20 options at first. Use **Show all** to see more.
- **Clear** resets Room AI selections, but not Accessibility (use the ♿ button for that).
- Options are generated from the current results, so they differ between searches.

---

## 6. Map controls related to filtering

| Control                       | Behaviour                                                                                                                                                                              |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Display exactly**           | Result filter. Shows only hotels inside the destination boundaries. If neighborhoods are selected, it shows only hotels inside the selected neighborhoods. Highlighted orange when on. |
| **Neighborhood**              | Select one or more neighborhoods of the destination. Works together with _Display exactly_.                                                                                            |
| **Show/Hide city boundaries** | Draws the destination polygon on the map.                                                                                                                                              |
| **Default center**            | Returns the map to the destination center.                                                                                                                                             |

---

## 7. Hotel card — elements users ask about

| Element                      | Behaviour                                                                                                                                                                                                                              |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Hotels X of Y** (tab)      | X = hotels matching the current filters, Y = all hotels loaded.                                                                                                                                                                        |
| **📌 Pin**                   | Pins the hotel to a block above the list. Pinned hotels ignore sorting. Pins are not saved after page reload.                                                                                                                          |
| **Dark badge** (e.g. `4`)    | Hotel star rating. For agents, clicking it opens an actions menu: Google, Tripadvisor (opens a search for the hotel), Monitor (copies a link), Copy name, Proposal, Find nearly (centers the map on the hotel), Preferred hotel, Info. |
| **👑 Crown**                 | Preferred hotel. The letter on the crown shows who recommended it: **S** – System recommended, **T** – Travel agent recommended, **C** – Client recommended. An empty (grey) crown means not preferred.                                |
| **Globe icon next to price** | Google price comparison. The tooltip shows the lowest price found on Google. Colour: **green** = our price is lower, **orange** = within ±3%, **red** = our price is higher. It compares the first refundable offer.                   |
| **Rating** (e.g. `3.7`)      | Guest review score.                                                                                                                                                                                                                    |
| **Distance**                 | From the destination center, or from the point chosen in **Location**.                                                                                                                                                                 |
| **Meal badge**               | RO – Room Only, BB – Bed & Breakfast, M – Meal.                                                                                                                                                                                        |
| **NO REF**                   | Non-refundable offer.                                                                                                                                                                                                                  |
| **Red price**                | The cancellation deadline has already passed (non-refundable).                                                                                                                                                                         |
| **Grey price** [Permission]  | Original supplier price in the supplier's currency. Shown only to users who see supplier names.                                                                                                                                        |
| **✉ Envelope**               | Adds the room offer to a **Proposal** (offer to send to a client). Highlighted when added.                                                                                                                                             |

---

## 8. Troubleshooting — typical situations

| Situation                                            | Likely reason                                                                                                                                                                                                                                           |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "A hotel I expect is missing."                       | It may be outside the search radius or below the minimum stars (search parameters). It may also be hidden by an active filter. Check whether **Clear filters** is highlighted. With **Display exactly**, hotels outside the city boundaries are hidden. |
| "The number in brackets doesn't match the results."  | Counts refer to all loaded hotels, not to the currently filtered list. For facility groups, the count is "at least one", while the filter requires "all".                                                                                               |
| "Numbers keep changing."                             | Results are still loading from suppliers.                                                                                                                                                                                                               |
| "My filters disappeared after a new search."         | The 🔒 **Keep filters** button is off.                                                                                                                                                                                                                  |
| "Filters from my previous search are still applied." | The 🔒 **Keep filters** button is on.                                                                                                                                                                                                                   |
| "BB shows half board and all inclusive too."         | BB means "at least breakfast".                                                                                                                                                                                                                          |
| "I don't see 1-star hotels in the star buttons."     | The buttons cover 2★ (★), 3, 4, 5 only.                                                                                                                                                                                                                 |
| "Distances changed."                                 | A point was selected in **Location**. Distances are now measured from it.                                                                                                                                                                               |
| "Free cancellation is on by default."                | The account is configured so. Click **Free** to switch it off.                                                                                                                                                                                          |
| "I don't see Suppliers / Net / currency / lock."     | These are agent features and depend on account permissions.                                                                                                                                                                                             |
| "Prices are purple / green."                         | The price type is **Net** (purple) or **Net.Т** (green) instead of **Reg**.                                                                                                                                                                             |
| "The same hotel from different suppliers."           | It is merged into one card. Offers from all suppliers are listed inside.                                                                                                                                                                                |
| "The page says the search is outdated."              | 20 minutes have passed. Run the search again for current prices.                                                                                                                                                                                        |

---

## 9. Example Q&A

**Q:** How do I find hotels with breakfast and free cancellation within 1 km of a conference venue?
**A:** Type the venue in the **Location** field and select it. The list will re-sort by distance. Then choose **1 km** in the km selector, click **BB** and switch on **Free**.

**Q:** I checked "Parking & Transport" and got only 2 hotels, but it says 17.
**A:** Checking the group requires hotels to have all its facilities (airport shuttle **and** car park). The 17 are hotels with at least one of them. Expand the group and check only the one you need.

**Q:** How do I search for two hotel chains at once?
**A:** Type the first name in the **Name** field in the sidebar, click **+**, and type the second one. Hotels matching either name are shown.

**Q:** What's the difference between Free in the toolbar and Free cancellation in the search?
**A:** _Free cancellation_ in the search loads only refundable offers (requires **GO**). **Free** in the toolbar hides non-refundable offers from the loaded results instantly, and can be switched off again.

**Q:** Why is the ★ button selected but I see 2-star hotels only?
**A:** The ★ button stands for 2-star hotels. Use 3, 4, 5 for other ratings.

**Q:** How do I keep my filters for the next search?
**A:** Click the funnel with the lock (🔒) so it's highlighted. The filters will be applied to your next search automatically.

**Q:** What does the green globe next to the price mean?
**A:** Our price is lower than the lowest price found on Google for that hotel. Orange means about the same (±3%), red means Google shows a lower price.

**Q:** Why is the price red?
**A:** The offer is non-refundable: its free cancellation deadline has already passed.
