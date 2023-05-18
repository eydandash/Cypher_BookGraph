# Book Graph in Neo4j

---

Implementation Story:
In my role, I was given the task of creating a graph database on Neo4j Desktop that represents a made-up social network where people share information about the books they've read. The database needed to include details about users (like their usernames and ages), books (such as the title, genre, author, and publisher), connections between users (who follows whom), relationships between users and books (who has read what), and user ratings for the books they've read.

To get started, I used the provided instructions and Cypher statements as a guide. I carefully built the graph database, making sure it accurately reflected the connections and information within the social network. This involved setting up user profiles with usernames and ages, and capturing book details like their titles, genres, authors, and publishers.

I also made sure users could follow each other and establish relationships in the network. Additionally, I created the ability for users to indicate which books they've read and provided a way for them to rate those books.

In the first part of the project, I used Cypher to create statements that allowed us to extract specific information from the database as required. This made it easy to retrieve user data, book data, relationship information, and user ratings efficiently.

For the second part of the project, I came up with simple recommendation algorithms using Cypher. These algorithms suggest users to follow and books to read based on factors like user preferences, past interactions, and common interests. These personalized recommendations were designed to enhance the user experience within the social network.

Overall, by applying my technical skills and understanding of the project requirements, I successfully implemented and delivered a functional graph database that accurately represents a fictitious social network focused on book sharing.

Setup:
To set up the graph database on Neo4j Desktop, follow these steps, and you can ensure that the database is created accurately and familiarize yourself with its structure.

1. Open Neo4j Desktop and create a new graph database named "Book Graph."
2. To import the necessary data, download the following CSV files: books.csv, users.csv, followers.csv, and ratings.csv. Copy these files to the import folder of your database. 
3. Execute the following Cypher statements one by one to populate the database with the necessary data about books, users, and their relationships:
a. Load the books.csv file and create nodes labeled as "Book" with properties such as bookID, title, genre, author, and publisher:
    
    ```sql
    LOAD CSV WITH HEADERS from 'file:///books.csv' AS book
    CREATE (:Book {bookID:book.BookId, title:book.Title, genre:book.Genre, author:book.Author, publisher:book.Publisher})
    
    ```
    
    This statement will create 101 nodes labeled as "Book," each containing the specified properties.
    
    b. Load the users.csv file and create nodes labeled as "User" with properties including username and age:
    
    ```sql
    LOAD CSV WITH HEADERS FROM 'file:///users.csv' AS user
    CREATE (:User {username: user.Username, age: toInteger(user.Age)})
    ```
    
    This statement will create 26 nodes labeled as "User," each with the specified properties.
    
    c. Load the followers.csv file and create relationships labeled as "FOLLOWS" between users:
    
    ```sql
    LOAD CSV WITH HEADERS FROM 'file:///followers.csv' AS fol
    MATCH (u1:User {username: fol.User1}), (u2:User {username: fol.User2})
    CREATE (u1)-[:FOLLOWS]->(u2)
    ```
    
    This statement will create 100 "FOLLOWS" relationships among users.
    
    d. Load the ratings.csv file and create relationships labeled as "READ" between users and books, including the rating property:
    
    ```sql
    LOAD CSV WITH HEADERS FROM 'file:///ratings.csv' AS rat
    MATCH (u:User {username: rat.User}), (b:Book {bookID: rat.Book})
    CREATE (u)-[:READ {rating: toInteger(rat.Rating)}]->(b)
    ```
    
    This statement will create 199 "READ" relationships between users and books, with each relationship containing the rating property.
    

## **Part 1**

1. Retrieve the titles and authors of the books that have been read by Lilly, but were not rated
    
    below 3 by other users. Expand the output table to include the received rating scores and
    
    usernames who provided the rating.
    
    ```jsx
    MATCH (book:Book)<-[r:READ]-(user:User)
    MATCH (book)<-[:READ]-(:User{username:"Lilly"})
    WHERE r.rating >=3 AND user.username <> "Lilly"
    RETURN book.title AS Title, book.author AS Author, r.rating AS Rating_Score, user.username AS Username
    ```
    
    Returns 12 records with 8 books, could have been distinct but the ratings are different. Could also be sorted by ratings or alphabetically for easier navigation.
    
2. Retrieve all, distinct publishers of the books along with the titles of the books they published.
    
    Display the list of publishers with largest number of books first.
    
    ```jsx
    MATCH (book:Book)
    WHERE book.publisher IS NOT NULL
    RETURN  book.publisher AS Publisher_Name, count(DISTINCT book.title) AS No_Of_Published_Books, collect(book.title) AS Published_Books
    ORDER BY count(DISTINCT book.title) DESC
    ```
    
    Returns 30 records with 30 different publishing houses, the no of the book they published and a list of the names of these books. The where clause was added to remove the records where a book’s publisher was null (4 books) because these books may have different publishers so this is inaccurate data.
    
3. Retrieve the titles, authors and genres of the books that have been read by Lilly and that are in the genres of history or fiction. 
    
    ```jsx
    MATCH (book:Book) <-[:READ]-(:User{username:"Lilly"})
    WHERE book.genre IN ["history", "fiction"]
    RETURN  book.title AS Title, book.author AS Author, book.genre AS Genre
    ```
    
    Returns 6 records with the book titles, authors and genres.
    
4. For each pair of users that follow each other, retrieve the titles and the publishers of the books that they don't share between them as read.
    
    ```jsx
    MATCH (u1:User)-[:FOLLOWS]->(u2:User)-[:FOLLOWS]->(u1), (u1)-[:READ]->(book:Book)
    WHERE NOT exists((book)<-[:READ]-(u2))
    RETURN DISTINCT book.title AS Title, book.publisher AS Publisher
    ```
    
    Returns 62 records with book titles and publishers.
    
5. Retrieve the names of the people who follow more than 5 others and list the number as well as the list of usernames they follow.
    
    ```jsx
    MATCH (follower:User)-[:FOLLOWS]->(user:User)
    WITH follower.username as Follower, collect(user.username) AS Username
    WHERE size(Username)>5
    RETURN Follower, size(Username) AS No_Of_Followed_Users, Usernames
    ```
    
    Returns 8 records with the follower names, number of followers and their names.
    
6. Retrieve the titles, number of received ratings and the average ratings of the 10 books with the highest average ratings
    
    ```jsx
    MATCH (book:Book)<-[review:READ]-(:User)
    RETURN book.title AS Title, count(review.rating) AS No_Of_Ratings, avg(review.rating) AS Average_Rating
    ORDER BY Average_Rating DESC
    LIMIT 10
    ```
    
    Returns 10 records with the book titles, their number of ratings received per book and the average of these ratings calculated, ordered as instructed.
    
7. Retrieve the titles, number of received ratings and the average ratings of all books that received more than 3 ratings.
    
    ```jsx
    MATCH (book:Book)<-[review:READ]-(:User)
    WITH book.title AS Title, count(review.rating) AS No_Of_Ratings, avg(review.rating) AS Average_Rating
    WHERE No_Of_Ratings >3
    RETURN Title, No_Of_Ratings, Average_Rating
    ```
    
    Returns 15 records with the book titles, their number of ratings received per book and the average of these ratings calculated.
    
8. Retrieve the titles of the books for which there is no publisher data and list all ratings they received.
    
    ```jsx
    MATCH (book:Book)<-[review:READ]-(:User)
    WITH book.title AS Title, book.publisher AS publisher, collect(review.rating) AS Ratings
    WHERE publisher IS NULL
    RETURN Title, Ratings
    ```
    
    Returns 4 records with the books name and list of ratings received per book.
    
9. Retrieve the paths that connect Adam to Lilly through the FOLLOWS relationship that is no longer than 5 hops. Print the list of connected usernames and the length of each path. Order the results by path length.
    
    ```jsx
    MATCH p = (:User{username:"Adam"})-[:FOLLOWS *0..5]-(:User{username:"Lilly"})
    RETURN [n in nodes(p) | n.username] AS Usernames, length(p) AS Path_Length
    ORDER BY Path_Length
    ```
    
    Returns 800 records with the usernames of the users included in each path, and the length of the path (how many hops of :FOLLOWS relationships).
    
10. Retrieve the shortest path (of any type of relationship) starting from user Steve to the most frequently read book.
    
    ```jsx
    MATCH (b:Book)<-[r:READ]-()
    MATCH p = shortestpath((steve:User{username:"Steve"})-[]->(b))
    RETURN  p AS Path, b.title AS Title, count(r) AS Frequency, length(p)
    ORDER BY Frequency DESC
    LIMIT 1
    ```
    
    Returns the shortest path as instructed.
    
11. Retrieve the users that read books in the same genre, as well as follow each other.
    
    ```jsx
    MATCH (u1:User)-[:FOLLOWS]->(u2:User), (u1)-[:READ]->(b1:Book), (u2)-[:READ]->(b2:Book)
    WHERE b1.genre = b2.genre
    RETURN DISTINCT u1.username as First_User, u2.username as Second_User
    ```
    
    Returns 99 records, the use of DISTINCT, although not given directly in the question, is used for optimising the results (without it 843 records are returned with many duplicates).
    
12. Retrieve books that were not read by any user, but are from a publisher that Charles read in
    
    the past.
    
    ```jsx
    MATCH (:User{username:"Charles"})-[:READ]->(book:Book), (unreadbook:Book)
    WHERE NOT EXISTS{ 
        ()-[:READ]->(unreadbook)
    } AND book.publisher = unreadbook.publisher
    RETURN DISTINCT unreadbook
    ```
    
    Returns 8 records conating full book information.
    
    ---
    
    ## **Part 2**
    
    ### User Recommendation System
    
    a.  Given a user u1 that we are looking to recommend users to, to follow, the first algorithm to create  a RECOMMENDED_USER relationship from u1 filters users that already follow u1, read books from similar genres to those read by 1 before and have given them a rating more than 3 (enjoyed reading them). This will ensure they have relatively similar interests and hence, will engage better.
    
    ```jsx
    MATCH (u1:User)-[r1:READ]->(b1:Book), (u2:User)-[r2:READ]->(b2:Book)
    WHERE NOT (u1)-[:FOLLOWS]->(u2) AND (u2)-[:FOLLOWS]->(u1) AND b1.genre=b2.genre AND r2.rating >= 3
    MERGE (u1)-[:RECOMMENDED_USER]->(u2)
    ```
    
    Created 84  Relationships.
    
    b.  Given a user u1 that we are looking to recommend users to, to follow, the second algorithm to create  a RECOMMENDED_USER relationship from u1 filters any users that have read more than 5 books before and are within the same age range as the user u1 (3 years younger or older). This ensures that they will be active users that u1 can benefit from their suggestions and also close in age to u1 which might make communication easier.
    
    ```jsx
    MATCH (u1:User), (u2:User), (u2)-[:READ]->(books:Book)
    WITH u1, u2, count(books) AS books_no, collect(books) AS books
    WHERE NOT (u1)-[:FOLLOWS]->(u2) AND books_no > 5 AND abs(u2.age - u1.age) = 3
    MERGE (u1)-[:RECOMMENDED_USER]->(u2)
    ```
    
    Created 12 Relationships that did not exist before (Merge functionality). 
    
    ### Book Recommendation System
    
    a. Given a user that we are looking to recommend books to read to, the first algorithm to create  a RECOMMENDED_BOOK Relationship from u1 filters books from the same genres and publishing houses as those previously read by the user, since users will likely be interested to read similar books to those they chose before.
    
    ```jsx
    MATCH (user:User)-[:READ]->(read:Book), (suggested:Book)
    WHERE NOT (user)-[:READ]->(suggested) 
    AND read.publisher = suggested.publisher
    AND read.genre = suggested.genre
    MERGE (user)-[:RECOMMENDED_BOOK]->(suggested)
    ```
    
    Created 440 Relationships
    
    b. Given a user, user1 that we are looking to recommend books to read to, the second algorithm to create  a RECOMMENDED_BOOK Relationship from user1 filters books that have been read by the users followed by user1, as well as being frequently read (read more than 5 times by any user except for user1). This suggests that these books are popular and relevant to user1’s interests.
    
    ```jsx
    MATCH (user1:User)-[:FOLLOWS]->(user2:User)-[:READ]->(book:Book), (book)<-[:READ]-(reader:User)
    WITH user1, book, count(reader) AS reading_frequency
    WHERE NOT (user1)-[:READ]->(book) AND reading_frequency > 5
    MERGE (user1)-[:RECOMMENDED_BOOK]->(book)
    ```
    
    Created 44 Relationships that did not exist before (Merge functionality).
    
    ---