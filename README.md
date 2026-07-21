# 🎵 Music Recommender Simulation

## Project Summary

In this project you will build and explain a small music recommender system.

Your goal is to:

- Represent songs and a user "taste profile" as data
- Design a scoring rule that turns that data into recommendations
- Evaluate what your system gets right and wrong
- Reflect on how this mirrors real world AI recommenders

My version scores each of 18 fictional songs against a user taste profile and returns the top matches as a readable table in the terminal. On top of the core scorer, it adds a few extra features: two advanced song bonuses (popularity and instrumentalness), four switchable scoring modes that re-weight what matters most, and a diversity penalty that discourages the top list from repeating the same artist or genre.

---

## How The System Works

This recommender uses a simple content-based scoring system. Each song has a set of attributes (`genre`, `mood`, `energy`, `acousticness`), and the user has a taste profile with matching preferences (`favorite_genre`, `favorite_mood`, `target_energy`, `likes_acoustic`). The system compares every song against the user's profile, turns that comparison into a single number, and recommends whichever songs score the highest — so a song moves from raw data into a recommendation purely by how closely its attributes line up with what the user said they want.

The starter catalog was also expanded with additional fictional songs spanning genres and moods the original dataset didn't cover (e.g. hip hop, folk, electronic, classical, r&b, metal, reggae, country), so the recommender has more variety to distinguish between when scoring.

Beyond the core scorer, this version adds a few extras (see the sections below for details):

- **Advanced song features** — songs now also have `popularity`, `release_decade`, `language`, `is_explicit`, and `instrumentalness`. Two of these, `popularity` and `instrumentalness`, add a small bonus to a song's score.
- **Scoring modes** — you can switch between `balanced`, `genre_first`, `mood_first`, and `energy_focused` to change how much each part of the score matters.
- **Diversity penalty** — when building the top list, a song loses points if its genre or artist already appears higher up, so the results are less repetitive.
- **Table output** — recommendations print as a clean ASCII table (Rank, Song Title, Artist, Score, Reasons) instead of a plain list.

### Example User Profile

```python
user_profile = {
    "favorite_genre": "lofi",
    "favorite_mood": "chill",
    "target_energy": 0.35,
    "likes_acoustic": True
}
```

### Algorithm Recipe

- **Genre match** — `+2.0` points if `song.genre == user.favorite_genre`, else `0`.
- **Mood match** — `+1.0` point if `song.mood == user.favorite_mood`, else `0`.
- **Energy closeness** — `energy_score = 1 - abs(song.energy - user.target_energy)`, so a song doesn't need to match the target energy exactly, just be close to it (range `0`–`1`).
- **Acoustic preference** — if `likes_acoustic` is `True`, `acoustic_score = song.acousticness`; if `False`, `acoustic_score = 1 - song.acousticness` (range `0`–`1`).
- **Popularity bonus** — `popularity_score = song.popularity / 100`, so a more popular song earns a little more (range `0`–`1`).
- **Instrumentalness bonus** — `instrumental_score = song.instrumentalness * 0.5`, a small bonus for more instrumental songs (range `0`–`0.5`).

Final score:

```python
final_score = (
    genre_score
    + mood_score
    + energy_score
    + acoustic_score
    + popularity_score
    + instrumental_score
)
```

The maximum possible score is `6.5` (`2.0 + 1.0 + 1.0 + 1.0 + 1.0 + 0.5`).

### Scoring Modes

The four scores that come from the user profile (genre, mood, energy, acoustic) each have a weight, and a **scoring mode** decides those weights. This lets you change what the recommender cares about most without rewriting the formula:

- **balanced** — the default; genre counts double (×2.0), everything else ×1.0.
- **genre_first** — genre counts even more (×3.0).
- **mood_first** — mood is boosted (×2.0) alongside genre.
- **energy_focused** — energy closeness is boosted (×2.0).

The popularity and instrumentalness bonuses are the same in every mode.

### Ranking Rule

Every song in the catalog is scored the same way, one at a time. Once all songs have a `final_score`, the full list is sorted from highest to lowest, and the Top K songs are returned as recommendations.

### Diversity Penalty

To avoid a "filter bubble" where the top list is all the same artist or genre, the top list is built one song at a time (in score order) and repeats are penalized:

- if a song's **genre** already appears in the list picked so far, it loses `0.25`
- if a song's **artist** already appears in the list picked so far, it loses `0.50`

The highest-scoring song is never penalized, and a matching reason (e.g. `diversity penalty: repeated genre (-0.25)`) is added so you can see when it happened.

### Bias Note

This system may over-prioritize genre and miss good songs that match the user's mood or energy but are from a different genre. It can also tend toward a filter bubble by repeatedly recommending similar songs, though the diversity penalty (see above) reduces this by discouraging repeated artists and genres in the top list. Since the dataset is small, some genres and moods may be underrepresented, which can skew what the recommender is able to surface.

---

## Getting Started

### Setup

1. Create a virtual environment (optional but recommended):

   ```bash
   python -m venv .venv
   source .venv/bin/activate      # Mac or Linux
   .venv\Scripts\activate         # Windows

2. Install dependencies

```bash
pip install -r requirements.txt
```

3. Run the app:

```bash
python -m src.main
```

### Running Tests

Run the starter tests with:

```bash
pytest
```

You can add more tests in `tests/test_recommender.py`.

---

## Sample Recommendation Output

Terminal output from `python -m src.main` (High-Energy Pop profile, `balanced` mode):

```text
Loaded 18 songs from data/songs.csv

Rank | Song Title      | Artist        | Score | Reasons
-----+-----------------+---------------+-------+---------------------------------------------------------------------------------------------------------------------
1    | Sunrise City    | Neon Echo     | 5.58  | genre match (+2.0); mood match (+1.0); energy closeness (+0.97); acoustic preference (+0.82); popularity bonus (+0.78); instrumentalness bonus (+0.01)
2    | Gym Hero        | Max Pulse     | 4.33  | genre match (+2.0); energy closeness (+0.92); acoustic preference (+0.95); popularity bonus (+0.70); instrumentalness bonus (+0.01); diversity penalty: repeated genre (-0.25)
3    | Rooftop Lights  | Indigo Parade | 3.23  | mood match (+1.0); energy closeness (+0.91); acoustic preference (+0.65); popularity bonus (+0.65); instrumentalness bonus (+0.01)
4    | Pulse Horizon   | Kilowatt      | 2.65  | energy closeness (+0.90); acoustic preference (+0.97); popularity bonus (+0.68); instrumentalness bonus (+0.10)
5    | Concrete Dreams | Static Verses | 2.49  | energy closeness (+0.85); acoustic preference (+0.92); popularity bonus (+0.72); instrumentalness bonus (+0.00)
```

Note how "Gym Hero" (rank 2) picks up a `diversity penalty: repeated genre (-0.25)` because "Sunrise City" above it is also `pop`.

**Screenshot or video** *(optional)*: <!-- Insert a screenshot or demo video link here -->

---

## Experiments You Tried

Use this section to document the experiments you ran. For example:

- What happened when you changed the weight on genre from 2.0 to 0.5
- What happened when you added tempo or valence to the score
- How did your system behave for different types of users

---

## Limitations and Risks

Summarize some limitations of your recommender.

Examples:

- It only works on a tiny catalog
- It does not understand lyrics or language
- It might over favor one genre or mood

You will go deeper on this in your model card.

---

## Reflection

Read and complete `model_card.md`:

[**Model Card**](model_card.md)

Write 1 to 2 paragraphs here about what you learned:

- about how recommenders turn data into predictions
- about where bias or unfairness could show up in systems like this



