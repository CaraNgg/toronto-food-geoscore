# 🗺️ Geospatial Food Recommendation Engine

Pick what you're craving and see which of 19 Toronto neighbourhood zones is the best match. The app scores 3,000+ places from the Google Places API across 31 food categories and highlights the top 3 zones on an interactive map.

Built for University of Toronto and TMU students: a walkable area bounded roughly by Dupont, Parliament, Front, and Ossington.

## 🎥 Live Demo

https://github.com/user-attachments/assets/6cd781c5-a906-46ea-9119-1a9da6c62b01


*Selecting categories, the top 3 matching zones highlighting on the map, and individual places shown as markers.*

## Features

- **Food preferences:** choose one or more of 31 categories, like Thai, bubble tea, ramen, or coffee
- **Top 3 zones:** the best matches are highlighted on the map, with the exact place counts behind each ranking
- **Place markers:** toggle on every matching spot, colour-coded by category (hover for name and rating)
- **Never empty:** if no zone is a perfect match, you still get the best available options, ranked

## How it works

1. Google Places API (New) is queried over a grid of points covering the study area
2. Results are cleaned and assigned to one of 19 hand-drawn zones with a point-in-polygon check
3. A Pandas pipeline builds a zone × category count table
4. A scoring model ranks the zones for the user's selected categories
5. Streamlit and Plotly draw the zone polygons (GeoJSON) and place markers on the map

## Design decisions

- **Scoring that balances volume and specialty:** each zone's score for a category is the geometric mean of its place count (normalized against the best zone) and its share of that category among its own places (also normalized). A zone only ranks well if it is both well stocked *and* known for that food. Raw counts alone let the busiest zone win 14 of 31 single-category searches, and a nearly empty zone could win because 2 of its 3 places happened to match.
- **Any number of categories:** per-category scores sit on a 0-1 scale and are averaged, so results stay comparable whether you pick one category or six. A small density tiebreaker (5%) keeps zones from tying.
- **Categories chosen by hand:** the first exploratory pull surfaced 255 Google place types. I reviewed them in Excel and Pandas, dropped non-food types, and grouped the rest into 31 categories (for example, `cafe`, `coffee_shop`, and `coffee_roastery` became "Coffee").
- **Batching under an API budget:** searching one place type at a time would have taken roughly 14,000 calls at this grid resolution. Searching once per category instead took about 3,300, staying under a 5,000-call budget.
- **Hand-drawn zones:** boundaries follow real street patterns and walkable clusters instead of a regular grid.
- **Filtering stray results:** Text Search (used for bubble tea, which Google doesn't expose as a type) only biases toward the study area, so a few results came back from far away. These are filtered against the study boundary before zone assignment.

## Known limitations

- Categories rely on Google's own type data, which is sometimes incomplete or inconsistent. Results were spot-checked, not exhaustively verified.
- All types in a category share one 20-result cap per search point, so very dense pockets (like Coffee near campus) may be slightly undercounted.
- The data is a single point-in-time pull (~3,300 places, September 2026), so places may have opened or closed since.
- Zone density varies from ~40 places in the quietest zone to ~340 in the busiest, since zones follow real neighbourhood boundaries.

## 🛠️ Tech stack

Python · Pandas · Streamlit · Plotly · GeoJSON · Google Places API · Excel
