# toronto-food-geoscore
A geospatial recommendation engine that ranks 19 Toronto neighbourhood zones against a user's food preferences, built on ~3,300 places from the Google Places API and a custom scoring model that balances availability against distinctiveness.

# Toronto Food GeoScore

A geospatial recommendation engine that ranks 19 Toronto neighbourhood zones against a user's food preferences, built on ~3,300 places from the Google Places API and a custom scoring model that balances availability against distinctiveness.

Built as an interactive Streamlit map for University of Toronto and TMU students: pick what you're craving (Thai, bubble tea, ramen, coffee, etc.) and see which part of downtown is the best match.

## Demo

![Demo](your-demo-gif-or-screenshot-here.gif)

*Selecting categories, the top 3 matching zones highlighting on the map, and individual places shown as markers.*

## What it does

1. Select one or more of 31 food and drink categories.
2. The app scores all 19 zones and highlights the **top 3** on the map, with the exact place counts behind each ranking.
3. Optionally toggle on individual place markers, color-coded by category, to see every matching spot (hover for name and rating).

It never returns an empty result: if no zone has a perfect match, you still get the best available options, ranked.

## Why this scope

Rather than covering all of Toronto, the project is scoped to a walkable area around UofT and TMU, bounded roughly by Dupont, Parliament, Front, and Ossington. The goal was a genuinely useful tool for a specific audience (students without a car), not a generic city-wide directory.

## How it works

### Data collection
- **Source:** Google Places API (New). Nearby Search for categories that have a dedicated Google type (e.g. `thai_restaurant`), and Text Search for categories that don't (e.g. bubble tea, which Google shows as a label on Maps but does not expose as a queryable type).
- **Category design:** An exploratory pull surfaced ~150 raw Google place types in the study area. I manually grouped them into 31 user-facing categories (e.g. `cafe`, `coffee_shop`, and `coffee_roastery` all became "Coffee"), keeping a category only if it had a meaningful number of places locally.
- **Cost control:** Nearby Search requests are batched by final category instead of by raw Google type (one call covering all four Coffee subtypes instead of four calls). This cut the full pull from ~14,000 API calls to ~3,300 at the same grid resolution.

### Zones
19 zones, hand-drawn over the study area instead of a regular grid, so boundaries follow real street patterns and walkable clusters rather than arbitrary grid lines. Each place is assigned to a zone with a point-in-polygon check.

### Scoring
For each selected category, a zone's score is the geometric mean of:
- **(a) availability:** its place count, normalized against the highest count of that category in any zone
- **(b) distinctiveness:** that count as a share of the zone's own total places, also normalized

A zone only ranks well if it is *both* well-stocked in the category *and* characterized by it. This avoids two failure modes I hit while testing: the busiest zone overall winning nearly every search just by volume (it won 14 of 31 single-category searches under raw counts alone), and a nearly empty zone winning because 2 of its 3 places happened to match. Multi-category picks average the per-category scores, with a small overall-density tiebreaker so no zone scores a literal zero.

### Data cleaning
Text Search's location bias is a soft hint, not a hard filter, so a few Bubble Tea results came back from far outside the study area. These are filtered out against the study boundary before zone assignment.

## Known limitations

- Category assignment relies on Google's own type data, which is sometimes incomplete or inconsistent (e.g. a sushi-and-poke restaurant tagged only as `meal_takeaway`). Results were spot-checked, not exhaustively verified.
- Batching requests by category means all types in a category share one 20-result cap per search point, so very dense pockets (e.g. Coffee near campus) may be slightly undercounted.
- The data is a single point-in-time pull (~3,300 places, September 2026), not live. Places may have opened or closed since.
- Zone density varies by design, from ~40 places in the quietest zone to ~340 in the busiest, since zones follow real neighbourhood boundaries instead of being equal-sized.

## Tech stack

- **Data collection:** Python, Google Places API (New)
- **Processing:** pandas, point-in-polygon zone assignment
- **App:** Streamlit, Plotly (choropleth map with place-level overlay markers)

## Repository contents

```
app.py                     Streamlit app
score_zones.py             Scoring logic (also usable standalone)
requirements.txt
data/
  zones.geojson            The 19 zone polygons
  places_with_zones.csv    ~3,300 places, each tagged with categories and a zone
  zone_category_counts.csv Zone x category counts used for scoring
```

## Running locally

```
pip install -r requirements.txt
streamlit run app.py
```

No API key is needed to run the app. It reads the CSV and GeoJSON files in `data/`.

### Re-pulling the data (optional)

The Google Places pull is only needed if you want fresh data. It requires a Google Places API key in a `.env` file:

```
GOOGLE_PLACES_API_KEY=your_key_here
```

then run `final_pull.py` followed by `assign_zones.py` (not required to use the app).
