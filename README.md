# 🎵 Music Recommender Simulation

## Project Summary

In this project you will build and explain a small music recommender system.

Your goal is to:

- Represent songs and a user "taste profile" as data
- Design a scoring rule that turns that data into recommendations
- Evaluate what your system gets right and wrong
- Reflect on how this mirrors real world AI recommenders

Replace this paragraph with your own summary of what your version does.

---

## How The System Works

This recommender uses a simple content-based scoring system: it compares each song's attributes against the user's taste profile and gives every song a score, then returns the highest-scoring songs.

Some prompts to answer:

- What features does each `Song` use in your system
  - For example: genre, mood, energy, tempo
  - `genre`, `mood`, `energy`, and `acousticness`.
- What information does your `UserProfile` store
  - `favorite_genre`, `favorite_mood`, `target_energy`, and `likes_acoustic`.
- How does your `Recommender` compute a score for each song
  - Each song gets a `final_score` made up of four parts:
    - **Genre match** — `+2` points if `song.genre == user.favorite_genre`. Genre is one of the clearest signals of music preference.
    - **Mood match** — `+2` points if `song.mood == user.favorite_mood`. Mood directly represents the "vibe" the user wants.
    - **Energy closeness** — `energy_score = 1 - abs(song.energy - user.target_energy)`. A song doesn't need an exact energy match, but closer energy should rank higher.
    - **Acoustic preference** — if `likes_acoustic` is `True`, `acoustic_score = song.acousticness`; if `False`, `acoustic_score = 1 - song.acousticness`. This captures the texture/production style the user prefers.
    - `final_score = genre_score + mood_score + energy_score + acoustic_score`, where `genre_score` and `mood_score` are each either `2` or `0`, and `energy_score` / `acoustic_score` are each between `0` and `1`.
- How do you choose which songs to recommend
  - Every song in the catalog is scored, then sorted from highest to lowest `final_score`. The top `k` songs are returned, each with a short explanation such as: *"Recommended because it matches your favorite genre and mood, and its energy level is close to your target energy."*

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

Paste a sample of your recommender's output here as a text block so a reader can see what it produces:

```
# e.g.:
# User profile: genre=indie, mood=chill, energy=low
# Recommendations:
#   1. ...
#   2. ...
#   3. ...
```

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



