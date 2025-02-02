# SQL; data analytics
Data Cleaning, analysis of users , comments and posts

-- Loading and Exploring Data
-- Explore the structure and first 10 rows of each table.
-- Identify the total number of records in each table.
-- total number of data is 10 rows per table, the column names are different but following are common column name
-- id, user_id, creation_date;

select * from badges limit 10;

select * from badges;
-- the badges and user_id are repeating ,

Select * from comments;
-- post_id and user_ id are repeated, till now user _id is unique identifier/primary ,

select * from post_history ;
-- post_id is linked with user_id, if he made any amendments the same link is shown here;

select * from posts_answers;
-- creation date and owner_user_id(user_id) are common with post history

select * from tags;
-- only program names,
select * from users;
-- display_name is given with user_id . hence making it convenient to connect with each other


select* from votes;
-- the common column is is, creation_date across all table,

select * from posts;
-- the column heading are differnt and need to be corrected, id is wrong , yrs are given and owner_user_id is suppose to be id,
-- post_histor_type_id , post_ id user_id and creation date are same as above comments table,

select * from post_links ;



-- Filtering and Sorting -- Find all posts with a comment_count greater than 2

select * from comments ;

SELECT post_id, user_id 
FROM comments
where user_id > 2 ;

SELECT post_id, user_id, count(user_id) as comment_count
FROM comments
GROUP BY post_id, user_id
HAVING COUNT(*) > 2;
select * from posts; 

 -- Display comments made in 2012, sorted by creation_date (comments table).
SELECT *
FROM comments
ORDER BY creation_date;


-- Count the total number of badges (badges table). 
select * from badges;

SELECT COUNT(*) AS total_badges
FROM badges;


-- Calculate the average score of posts grouped by post_type_id (posts_answer table).

select * from  posts;

select post_type_id, avg(score)
from posts
group by post_type_id;

-- Combine the post_history and posts tables to display the title of posts and the corresponding changes made in the post history.  

SELECT p.title, ph.text
FROM posts AS p
JOIN post_history AS ph ON p.id = ph.post_id;

-- Join the users table with badges to find the total badges earned by each user.
SELECT u.display_name, COUNT(b.id) AS total_badges_earned
FROM users u
LEFT JOIN badges b ON u.id = b.user_id
GROUP BY u.display_name
ORDER BY total_badges_earned DESC;

-- Fetch the titles of posts (posts), their comments (comments), and the users who made those comments (users).
SELECT 
    p.title AS post_title,
    c.text AS comment_text,
    u.display_name AS commenter_name
FROM 
    posts p
JOIN 
    comments c ON p.id = c.post_id
JOIN 
    users u ON c.user_id = u.id;
    
-- Combine post_links with posts to list related questions
SELECT 
    p1.title AS OriginalQuestionTitle,
    p2.title AS RelatedQuestionTitle
FROM 
    posts AS p1
JOIN 
    post_links AS pl ON p1.id = pl.post_id
JOIN 
    posts AS p2 ON pl.related_post_id = p2.id
WHERE 
    pl.link_type_id = 1;  -- Filter for related questions (link_type_id = 1)
    
-- Join the users, badges, and comments tables to find the users who have earned badges and made comments.
SELECT DISTINCT
    u.display_name
FROM
    users u
JOIN
    badges b ON u.id = b.user_id
JOIN
    comments c ON u.id = c.user_id;
    
-- Single-Row Subqueries
-- Find the user with the highest reputation (users table).

SELECT display_name, reputation
FROM users
ORDER BY reputation DESC
LIMIT 1;

-- Retrieve posts with the highest score in each post_type_id (posts table).
SELECT p1.*
FROM posts p1
JOIN (
    SELECT post_type_id, MAX(score) AS max_score
    FROM posts
    GROUP BY post_type_id
) AS p2 ON p1.post_type_id = p2.post_type_id AND p1.score = p2.max_score;

-- For each post, fetch the number of related posts from post_links
SELECT 
    p.id AS PostId,
    p.title AS PostTitle,
    COUNT(pl.related_post_id) AS NumberOfRelatedPosts
FROM 
    posts AS p
LEFT JOIN 
    post_links AS pl ON p.id = pl.post_id
GROUP BY 
    p.id, p.title
ORDER BY 
    NumberOfRelatedPosts DESC;


-- Create a CTE to calculate the average score of posts by each user and use it to:
-- List users with an average score above 50.
-- Rank users based on their average post score

WITH UserAverageScores AS (
    SELECT 
        u.id AS user_id,
        u.display_name AS user_name,
        AVG(p.score) AS average_score
    FROM 
        users u
    JOIN 
        posts p ON u.id = p.owner_user_id
    GROUP BY 
        u.id, u.display_name
)

SELECT 
    user_name, 
    average_score
FROM 
    UserAverageScores
WHERE 
    average_score > 50
ORDER BY 
    average_score DESC;


-- To rank users based on their average post score:
WITH UserAverageScores AS (
    SELECT 
        u.id AS user_id,
        u.display_name AS user_name,
        AVG(p.score) AS average_score
    FROM 
        users u
    JOIN 
        posts p ON u.id = p.owner_user_id
    GROUP BY 
        u.id, u.display_name
)

SELECT 
    user_name, 
    average_score,
    RANK() OVER (ORDER BY average_score DESC) AS user_rank
FROM 
    UserAverageScores;
    
-- Simulate a hierarchy of linked posts using the post_links table
    SELECT
    p1.Id AS ParentPostId,
    p1.Title AS ParentPostTitle,
    p2.Id AS ChildPostId,
    p2.Title AS ChildPostTitle
FROM
    posts AS p1  -- Parent posts
INNER JOIN
    post_links AS pl ON p1.Id = pl.post_id  -- Join on PostId
INNER JOIN
  posts AS p2 ON pl.related_post_id= p2.Id  -- Join on RelatedPostId
WHERE
    pl.link_type_id = 1;  -- Filter for related posts (link_type_id = 1)
    
    
-- Rank posts based on their score within each year (posts table).

SELECT
    EXTRACT(YEAR FROM creation_date) AS post_year,  -- Extract the year from creation_date
    Title,                                         -- post_title
    Score,                                          -- post_score
    RANK() OVER (PARTITION BY EXTRACT(YEAR FROM creation_date) ORDER BY Score DESC) AS rank_within_year
FROM
    Posts;
    
-- Calculate the running total of badges earned by users (badges table).

SELECT
    user_id,
    Date,
    Name,                      -- name
    SUM(1) OVER (PARTITION BY user_id ORDER BY Date) AS RunningTotal -- Running total of badges
FROM
    badges;
    
-- Which users have contributed the most in terms of comments, edits, and votes?
-- user_id 1001 and 1002
--  What types of badges are most commonly earned, and which users are the top earners?
-- badge name : bronze Reviewer, earned by user_id 1001
-- Which tags are associated with the highest-scoring posts?
-- introduction to SQL and Window function
-- how often are related questions linked, and what does this say about knowledge sharing?
-- 3times the posts are linked, ppl get information in three attempts. 
