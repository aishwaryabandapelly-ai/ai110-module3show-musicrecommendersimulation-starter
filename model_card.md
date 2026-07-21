# 🎧 Model Card: Music Recommender Simulation

## 1. Model Name  

Give your model a short, descriptive name.  
Example: **VibeFinder 1.0**  

---

## 2. Intended Use  

Describe what your recommender is designed to do and who it is for. 

Prompts:  

- What kind of recommendations does it generate  
- What assumptions does it make about the user  
- Is this for real users or classroom exploration  

---

## 3. How the Model Works  

Explain your scoring approach in simple language.  

Prompts:  

- What features of each song are used (genre, energy, mood, etc.)  
- What user preferences are considered  
- How does the model turn those into a score  
- What changes did you make from the starter logic  

Avoid code here. Pretend you are explaining the idea to a friend who does not program.

---

## 4. Data  

Describe the dataset the model uses.  

Prompts:  

- How many songs are in the catalog  
- What genres or moods are represented  
- Did you add or remove data  
- Are there parts of musical taste missing in the dataset  

---

## 5. Strengths  

Where does your system seem to work well  

Prompts:  

- User types for which it gives reasonable results  
- Any patterns you think your scoring captures correctly  
- Cases where the recommendations matched your intuition  

---

## 6. Limitations and Bias 

Where the system struggles or behaves unfairly. 

Prompts:  

- Features it does not consider  
- Genres or moods that are underrepresented  
- Cases where the system overfits to one preference  
- Ways the scoring might unintentionally favor some users  

The scoring formula may over-prioritize genre by giving it the largest fixed bonus (+2.0), so a song can rank highly even when its mood, energy, and acousticness are not a perfect fit. Because the catalog is small, with only a limited number of songs and some genres or moods represented only once, certain user profiles may not have many strong candidates to choose from. The `likes_acoustic` field is also too simple because it forces every user into a True/False preference, while real acoustic preference is usually more flexible. The weight-shift experiment showed that changing feature weights can significantly change the rankings, meaning the recommender is sensitive to design choices and could accidentally override a user's stated preference.

---

## 7. Evaluation  

How you checked whether the recommender behaved as expected. 

Prompts:  

- Which user profiles you tested  
- What you looked for in the recommendations  
- What surprised you  
- Any simple tests or comparisons you ran  

No need for numeric metrics unless you created some.

### Profiles Tested

I ran `python -m src.main` against four kinds of user profiles to see how the recommender behaves: three "normal" listener personas, and one adversarial profile designed to expose weaknesses.

**1. High-Energy Pop** (`favorite_genre="pop"`, `favorite_mood="happy"`, `target_energy=0.85`, `likes_acoustic=False`)

```text
1. Sunrise City by Neon Echo
   Score: 4.79
   Reasons:
   - genre match (+2.0)
   - mood match (+1.0)
   - energy closeness (+0.97)
   - acoustic preference (+0.82)

2. Gym Hero by Max Pulse
   Score: 3.87
   Reasons:
   - genre match (+2.0)
   - energy closeness (+0.92)
   - acoustic preference (+0.95)

3. Rooftop Lights by Indigo Parade
   Score: 2.56
   Reasons:
   - mood match (+1.0)
   - energy closeness (+0.91)
   - acoustic preference (+0.65)
```

This one makes sense at a glance: the winner, "Sunrise City," is literally tagged `pop`/`happy` in the data, so it collects both fixed bonuses, and it also happens to be high-energy and not very acoustic — exactly what this listener asked for. Everything lined up, so the #1 pick feels obviously correct.

**2. Chill Lofi** (`favorite_genre="lofi"`, `favorite_mood="chill"`, `target_energy=0.35`, `likes_acoustic=True`)

```text
1. Library Rain by Paper Lanterns
   Score: 4.86
   Reasons:
   - genre match (+2.0)
   - mood match (+1.0)
   - energy closeness (+1.00)
   - acoustic preference (+0.86)

2. Midnight Coding by LoRoom
   Score: 4.64
   Reasons:
   - genre match (+2.0)
   - mood match (+1.0)
   - energy closeness (+0.93)
   - acoustic preference (+0.71)

3. Focus Flow by LoRoom
   Score: 3.73
   Reasons:
   - genre match (+2.0)
   - energy closeness (+0.95)
   - acoustic preference (+0.78)
```

Same story here — a soft, low-energy, mostly-acoustic song ("Library Rain") that's already tagged `lofi`/`chill` comes out on top with a near-perfect score. Compared to the High-Energy Pop result, the ranking "shape" is the same (a clean top pick everyone would agree with), just built from opposite raw numbers — low energy and high acousticness instead of high energy and low acousticness. That's a good sign: the formula treats "closeness to target" the same way regardless of which direction the target points.

**3. Deep Intense Rock** (`favorite_genre="rock"`, `favorite_mood="intense"`, `target_energy=0.90`, `likes_acoustic=False`)

```text
1. Storm Runner by Voltline
   Score: 4.89
   Reasons:
   - genre match (+2.0)
   - mood match (+1.0)
   - energy closeness (+0.99)
   - acoustic preference (+0.90)

2. Gym Hero by Max Pulse
   Score: 2.92
   Reasons:
   - mood match (+1.0)
   - energy closeness (+0.97)
   - acoustic preference (+0.95)
```

"Storm Runner" wins clearly and correctly — but this also reveals a catalog limitation: it's the *only* `rock`-tagged song in the whole dataset. There's no runner-up that also matches genre, so anyone with this taste profile only ever gets one "real" match no matter how the recommender is tuned. The #2 pick, "Gym Hero," is a `pop` song that only sneaks in because it happens to be high-energy — a plausible backup, but not a genuine rock recommendation.

**4. Edge case — Conflicting Signals** (`favorite_genre="rock"`, `favorite_mood="chill"`, `target_energy=0.15`, `likes_acoustic=True` — a listener who says they like rock, but everything else about them points toward soft, quiet music)

```text
1. Spacewalk Thoughts by Orbit Bloom
   Score: 2.79
   Reasons:
   - mood match (+1.0)
   - energy closeness (+0.87)
   - acoustic preference (+0.92)

2. Library Rain by Paper Lanterns
   Score: 2.66
   Reasons:
   - mood match (+1.0)
   - energy closeness (+0.80)
   - acoustic preference (+0.86)

3. Midnight Coding by LoRoom
   Score: 2.44
   Reasons:
   - mood match (+1.0)
   - energy closeness (+0.73)
   - acoustic preference (+0.71)

4. Storm Runner by Voltline
   Score: 2.34
   Reasons:
   - genre match (+2.0)
   - energy closeness (+0.24)
   - acoustic preference (+0.10)
```

This is the surprise. "Storm Runner" is the *only* song in the whole catalog that matches this listener's stated favorite genre (`rock`), yet it drops to 4th place, beaten out by three lofi/ambient songs that don't match genre at all. The reason is plain arithmetic: the genre bonus (+2.0) is fixed, but "Storm Runner" is a loud, non-acoustic song, so it loses almost a full point on energy and 0.9 of a point on acousticness — enough to sink below songs that match none of the categorical preferences but fit the mood/energy/acoustic numbers closely. In plain terms: a listener who explicitly says "I like rock" can end up not being shown the one rock song available, because the math lets quieter, non-rock songs out-earn it on the numeric side. That's a real limitation — the formula can quietly override an explicit, stated preference.

**Comparing across profiles:** when a listener's genre, mood, and numeric preferences all point the same direction (profiles 1–3), the top pick is obvious and satisfying, and the catalog's small size mostly just means "few options," not "wrong options." But profile 4 shows that once a listener's stated genre conflicts with their other preferences, the recommender doesn't recognize that tension — it just adds up four independent numbers and lets the biggest total win, even if that total buries the one song that actually matches what the listener said they wanted most.

### High-Energy Pop Result Explanation

For the High-Energy Pop profile (`favorite_genre="pop"`, `favorite_mood="happy"`, `target_energy=0.85`, `likes_acoustic=False`), the top recommendation was "Sunrise City" by Neon Echo, scoring 4.79 out of a possible 5.0. This song matched both categorical preferences (genre +2.0, mood +1.0) and scored well on both continuous features (energy closeness +0.97, since its energy of 0.82 sits close to the 0.85 target; acoustic preference +0.82, since its low acousticness of 0.18 suits a user who dislikes acoustic songs). All four components reinforced each other rather than conflicting, which made this an intuitive, easy-to-explain #1 result and confirms the scoring formula behaves as expected in the straightforward case.

---

## 8. Future Work  

Ideas for how you would improve the model next.  

Prompts:  

- Additional features or preferences  
- Better ways to explain recommendations  
- Improving diversity among the top results  
- Handling more complex user tastes  

---

## 9. Personal Reflection  

A few sentences about your experience.  

Prompts:  

- What you learned about recommender systems  
- Something unexpected or interesting you discovered  
- How this changed the way you think about music recommendation apps  
