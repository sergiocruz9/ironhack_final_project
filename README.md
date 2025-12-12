What makes a song popular?

Spotify audio features analysis (2010–2020)

1. Project overview

This project looks at one simple question:

Given only the audio features of a track, what makes it more likely to be popular on Spotify?

I worked with a large Spotify dataset (Kaggle, 1921–2020) and focused on the modern streaming era (2010–2020).
The project combines:

Data wrangling and feature engineering

Exploratory analysis and statistical tests

A simple logistic regression model to predict popularity from audio features

The angle is business-oriented: think of an analyst working inside a label who wants to give A&R and producers a clearer picture of the “audio profile” of popular tracks.

2. Business questions

Audio profile

How do popular tracks differ from non-popular tracks in terms of duration, danceability, loudness, instrumentalness and other Spotify audio features?

Statistical evidence

Are those differences large and consistent enough to be statistically meaningful, or are they just noise?

Prediction

With only the audio features, can we build a simple model that flags tracks that are likely to be at least “very popular”?

Actionable recommendations

How can a label use these insights when scouting new songs or developing artists?

3. Key insights (executive summary)

All insights below are based on tracks released between 2010 and 2020.

3.1 Audio profile of more popular songs

Comparing very popular tracks (popularity 55–75) with non-popular tracks (≤ 25):

Shorter tracks perform better

Very popular tracks are on average 0.4 minutes shorter
(about 25 seconds shorter).

More danceable

Danceability is about 0.07 points higher on Spotify’s 0–1 scale.

Louder mixes

Very popular tracks are on average ~1.4 dB louder.

Much less instrumental

Instrumentalness is about 0.24 points lower (scale 0–1),
meaning popular tracks are rarely purely instrumental.

These differences are statistically significant (confirmed with ANOVA and t-tests).

In plain language:

Popular tracks tend to be shorter, more danceable, louder, and clearly vocal-driven.

3.2 Predictive model

I trained a logistic regression model to predict whether a song is at least “very popular” (popularity ≥ 55):

Features: 10 Spotify audio features + explicit flag

Data: 2010–2020 tracks, 80/20 train–test split

Scaling: StandardScaler

Imbalance handling: class_weight="balanced"

On the test set:

Accuracy: ~0.62

ROC–AUC: ~0.67

For the “popular” class:

Precision: ~0.34

Recall: ~0.61

So the model is far from perfect, but it does capture real signal:

It correctly identifies around 61% of popular tracks using only audio features.

The coefficients of the model line up with the EDA:

Higher loudness, danceability and being explicit → increase the chance of being popular

Higher instrumentalness, acousticness, liveness, longer duration, and very high energy → decrease it

3.3 Business takeaways

If you are a label or producer working in mainstream streaming:

Aim for a 3:00–3:30 duration for tracks that are meant to perform on playlists.

Focus on groove and danceability, not just raw energy.

Master tracks to be competitively loud (within reasonable limits).

For big commercial bets, a strong vocal presence is almost a requirement; instrumental tracks are structurally disadvantaged.

A simple scoring model based on audio features can help prioritize demos for A&R to listen to, but should not be used alone to decide releases.

4. Data

Source

Kaggle: “Spotify Dataset 1921–2020, 600k+ Tracks” (by Yamac Eren Ay)

Built from the Spotify Web API

Tables used

Tracks table: track-level information and audio features

Artists table: artist popularity, follower counts and genre tags

Relevant columns

From the tracks data (after filtering and cleaning):

track_id, name, release_date, popularity

duration_ms → converted to duration_min

danceability, energy, loudness, speechiness

acousticness, instrumentalness, liveness, valence, tempo

explicit

Derived: release_year, pop_bucket, is_popular

From the artists data:

id (artist id), name, genres

Derived: main_genre (coarse genre category, used mainly for exploration)

5. Data preparation

Main steps (in Python / pandas):

Filter time window

Keep only tracks with 2010 ≤ release_year ≤ 2020.

Ensure one row per track

Drop duplicate track_id rows.

Feature engineering

duration_min = duration_ms / 60000

release_year from release_date

explicit_int (0/1) from the boolean explicit flag

Popularity buckets:

non_pop (≤ 25)

somewhat_pop (25–55)

very_pop (55–75)

hit (> 75)

Binary label for modeling:

is_popular = 1 if popularity ≥ 55 (very_pop + hit), else 0

Merge with artist information

Extract the first artist ID per track and join with the artists table.

Build a rough main_genre from the list of artist genres (e.g., pop, rock, hip hop, etc.) to support breakdowns by genre.

Missing values

Drop rows missing core audio features or popularity (very small fraction of the data).

Final modeling dataset

Around 120k tracks from 2010–2020 with complete audio features and popularity.

6. Analysis approach
6.1 Exploratory data analysis (EDA)

Checked distribution of tracks per year to confirm coverage.

Explored popularity distribution and defined bucket thresholds.

Visualized average audio profiles by popularity bucket:

Duration, danceability, loudness, instrumentalness, etc.

Built a correlation matrix between popularity and all audio features.

The EDA already suggested a clear pattern: more popular tracks are slightly shorter, more danceable, louder and less instrumental.

6.2 Statistical testing

To move beyond visual impressions:

Used one-way ANOVA to test whether the means of each feature are equal across the four popularity buckets.

For features with clear differences (duration, danceability, loudness, instrumentalness), ANOVA p-values were essentially zero → we reject the null of equal means.

As a complement, used two-sample t-tests between the extremes (non_pop vs very_pop) to quantify the size and direction of the differences.

Result: the differences in duration, danceability, loudness and instrumentalness between popularity groups are large and statistically significant.

6.3 Predictive modeling

Model: Logistic Regression

Features: 10 audio features + explicit flag

Scaling: StandardScaler

Class imbalance: class_weight="balanced"

Split: 80% train / 20% test, stratified by label.

Evaluation:

Accuracy, ROC–AUC

Precision, recall, F1 for the “popular” class

Confusion matrix

The model is intentionally simple and interpretable: it is meant as a proof of concept and to confirm whether the patterns seen in the EDA are strong enough to support prediction.

7. How to run the project

Basic requirements (Python):

pandas

numpy

matplotlib

seaborn

scikit-learn

Typical workflow:

Download the Kaggle dataset and place the CSVs under data/raw/ (or similar).

Open the main notebook (notebooks/spotify_popularity_analysis.ipynb, for example).

Run the cells in order:

Data loading and cleaning

EDA plots

Statistical tests

Logistic regression training and evaluation

The notebook is written to be readable and to mirror the structure described in this README.

8. Possible next steps

If this were a real project inside a label, natural next steps would be:

Add non-audio variables: playlist placements, campaign data, artist followers, release strategy, etc.

Try non-linear models (e.g. tree-based methods) and compare to logistic regression.

Explore genre-specific models (pop vs hip hop vs rock) to capture different patterns.

Turn the model into a small internal tool where A&R can upload audio / track IDs and get back a popularity score plus the key audio features.

For the scope of this bootcamp project, the current version already shows that:

Audio features alone are enough to capture a meaningful part of what makes a track popular on Spotify, and to give the business side a clearer, data-backed picture of the sound of success.# ironhack_final_project
