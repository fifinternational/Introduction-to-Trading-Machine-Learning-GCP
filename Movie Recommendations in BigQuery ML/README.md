# Movie Recommendations using BigQuery ML

## Overview

BigQuery is Google's fully managed, NoOps, low-cost analytics database. With BigQuery, one can query terabytes of data without managing infrastructure or needing a database administrator. BigQuery uses SQL and a pay-as-you-go model, allowing you to focus on analyzing data to find meaningful insights.

BigQuery Machine Learning (BigQuery ML) is a feature in BigQuery that allows data analysts to create, train, evaluate, and predict with machine learning models with minimal coding.

Collaborative filtering provides a way to generate product recommendations for users or user targeting for products. The starting point is a table with three columns: a user id, an item id, and the rating the user gave the product. This table can be sparse—users don’t have to rate all products. Based on just the ratings, the technique finds similar users and similar products and determines the rating that a user would give an unseen product. Then, products with the highest predicted ratings can be recommended to users or target products at users with the highest predicted ratings.

To illustrate recommender systems in action, the MovieLens dataset was used. This is a dataset of movie reviews released by GroupLens, a research lab in the Department of Computer Science and Engineering at the University of Minnesota, through funding by the US National Science Foundation.

## Objectives

In this lab, the following tasks were performed:

- Created a BigQuery dataset to store and load MovieLens data
- Explored the MovieLens dataset
- Used a trained model to make recommendations in BigQuery
- Made product predictions for both single users and batch users

## Setup and Requirements

For each lab, a new Google Cloud project and set of resources for a fixed time at no cost were provided.

1. Signed in to Qwiklabs using an incognito window.
2. Noted the lab's access time and ensured completion within that time.
3. Started the lab.
4. Used provided lab credentials to sign in to the Google Cloud Console.
5. Opened Google Console and used the lab-provided credentials.

## Instructions and Tasks

### Task 1: Get MovieLens Data

The command line was used to create a BigQuery dataset to store the MovieLens data. The MovieLens data was then loaded from a Cloud Storage bucket into the dataset.

#### Start the Cloud Shell Editor

1. In the Google Cloud Console, clicked Activate Cloud Shell and continued when prompted.

#### Create and Load BigQuery Dataset

A BigQuery dataset named `movies` was created using the following command:

```sh
bq --location=EU mk --dataset movies
The data was then loaded with these commands:
```

```sh
bq load --source_format=CSV \
 --location=EU \
 --autodetect movies.movielens_ratings \
 gs://dataeng-movielens/ratings.csv
```

```sh
bq load --source_format=CSV \
 --location=EU \
 --autodetect movies.movielens_movies_raw \
 gs://dataeng-movielens/movies.csv
```

### Task 2: Explore the Data
The MovieLens dataset was explored and verified using the Query editor.

Count Users, Movies, and Ratings
In BigQuery's Query editor, the following query was executed:

```sql
SELECT
  COUNT(DISTINCT userId) numUsers,
  COUNT(DISTINCT movieId) numMovies,
  COUNT(*) totalRatings
FROM
  movies.movielens_ratings
The dataset consists of over 138 thousand users, nearly 27 thousand movies, and a little more than 20 million ratings.
```
Examine the First Few Movies
The following query was executed:

```sql
Copy code
SELECT
  *
FROM
  movies.movielens_movies_raw
WHERE
  movieId < 5
The genres column was parsed into an array and the results were rewritten into a table named movielens_movies:
```

```sql
CREATE OR REPLACE TABLE
  movies.movielens_movies AS
SELECT
  * REPLACE(SPLIT(genres, "|") AS genres)
FROM
  movies.movielens_movies_raw
```
###Task 3: Evaluate a Trained Model Created Using Collaborative Filtering
Metrics for a trained model generated using matrix factorization were viewed. A model had been created in the Cloud Training project's cloud-training-prod-bucket BigQuery dataset for use in the rest of the lab.

To view metrics for the trained model, the following query was run:

```sql
Copy code
SELECT * FROM ML.EVALUATE(MODEL `cloud-training-prod-bucket.movies.movie_recommender`)
Task 4: Make Recommendations
The trained model was used to provide recommendations.
````
Recommend Comedy Movies for User 903
The following query was executed to find the best comedy movies to recommend to the user whose userId is 903:

```sql
Copy code
SELECT
  *
FROM
  ML.PREDICT(MODEL `cloud-training-prod-bucket.movies.movie_recommender`,
    (
    SELECT
      movieId,
      title,
      903 AS userId
    FROM
      `movies.movielens_movies`,
      UNNEST(genres) g
    WHERE
      g = 'Comedy' ))
ORDER BY
  predicted_rating DESC
LIMIT
  5
```
This result included movies the user had already seen and rated in the past. To remove them, the following query was used:

```sql
SELECT
  *
FROM
  ML.PREDICT(MODEL `cloud-training-prod-bucket.movies.movie_recommender`,
    (
    WITH
      seen AS (
      SELECT
        ARRAY_AGG(movieId) AS movies
      FROM
        movies.movielens_ratings
      WHERE
        userId = 903 )
    SELECT
      movieId,
      title,
      903 AS userId
    FROM
      movies.movielens_movies,
      UNNEST(genres) g,
      seen
    WHERE
      g = 'Comedy'
      AND movieId NOT IN UNNEST(seen.movies) ))
ORDER BY
  predicted_rating DESC
LIMIT
  5
```

###Task 5: Apply Customer Targeting
The top-rated movies for a specific user were identified.

Identify Users Likely to Rate a Movie Highly
To identify users who are likely to rate movieId=96481 highly, the following query was used:

```sql
SELECT
  *
FROM
  ML.PREDICT(MODEL `cloud-training-prod-bucket.movies.movie_recommender`,
    (
    WITH
      allUsers AS (
      SELECT
        DISTINCT userId
      FROM
        movies.movielens_ratings )
    SELECT
      96481 AS movieId,
      (
      SELECT
        title
      FROM
        movies.movielens_movies
      WHERE
        movieId=96481) title,
      userId
    FROM
      allUsers ))
ORDER BY
  predicted_rating DESC
LIMIT
  100
```
###Task 6: Perform Batch Predictions for All Users and Movies
A query was performed to obtain batch predictions for users and movies.

Obtain Batch Predictions
The following query was entered to obtain batch predictions:

```sql
SELECT
  *
FROM
  ML.RECOMMEND(MODEL `cloud-training-prod-bucket.movies.movie_recommender`)
LIMIT
  100000
```
