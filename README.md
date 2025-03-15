# Netflix Movies and TV Shows Data Analysis using SQL

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.


## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)


## Schema
``` sql
CREATE TABLE IF NOT EXISTS netflix
(
    show_id      VARCHAR(5),
    type        VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts         VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```


### 1. Count the Number of Movies vs TV Shows
``` sql
SELECT type, count(type) as Number_of_TV from netflix
group by 1;
```

### 2. Find the Most Common Rating for Movies and TV Shows
``` sql
SELECT type, rating, common_rating
from (
    SELECT 
        type, 
        rating, 
        count(rating) as common_rating,
        ROW_NUMBER() over (order by count(rating) desc) as rn
    from netflix
    group by type, rating
) as ranked_ratings
where rn <= 2;
```
### 3. List All Movies Released in a Specific Year (e.g., 2020)
``` sql
select * from netflix
where type = 'Movie' and release_year = 2020 
```


### 4. Find the Top 5 Countries with the Most Content on Netflix
``` sql
SELECT 
    unnest(STRING_TO_ARRAY(country, ',')) as country,
    count(*) as total_content from netflix
where nullif(TRIM(country), '') IS not NULL
group by 1
order by total_content desc
limit 5
```
### 5. Identify the Longest Movie
``` sql
SELECT * from netflix
where type = 'Movie'
order by SPLIT_PART(duration, ' ', 1)::INT DESC;
```

### 6. Find Content Added in the Last 5 Years
``` sql
SELECT *from netflix
where TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```
### 7. Find the average number of content releases in India on Netflix for each of the last 10 years.
``` sql
SELECT country, release_year,
  count(show_id) as total_release,
  ROUND(count(show_id)::numeric /
        (SELECT count(show_id) from netflix where country = 'India')::numeric * 100, 2
    ) as avg_release
from netflix
where country = 'India'
group by country, release_year
order by avg_release desc
limit 5; 
```

### 8. Categorize Content Based on the Presence of 'love','romantic','kill' and 'horror' Keywords
``` sql
SELECT category, count(*) AS content_count 
   from (SELECT case 
                when description ~ ('love|romantic') then 'good'
				when description ~ ('kill|horror') then 'bad'
				else 'neutral'
		        end as category 
				from netflix
	) as categorized_content
group by category;
```

## Conclusion
This analysis provides key insights into Netflix’s content distribution, ratings, and trends. It highlights the dominance of movies over TV shows, the most common ratings, and content trends across different years and countries. Additionally, categorizing content based on keywords helps understand thematic preferences. These findings can assist in making data-driven decisions for content strategy and audience engagement.























