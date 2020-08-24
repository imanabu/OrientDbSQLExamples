# OrientDbSQLExamples
Various examples to clarify the use of the OrientDB SQL

## What You Will Need

I will be referring to the OrientDB as ODB or OD throughout this document.

After you have set up your ODB, please install the MovieRatings DB. There is a quick setup for this in the ODB Studio login panel, from where you can create a new DB.

## Quick Check: List all the movies in ODB

    SELECT FROM Movies
  
## The Graph Entity Relations

![Movies DB Entity Relations](Entities.png)

* Users can rate movies and they also have occupations
* Movies can be rated by the users and they have genres

## The Query Collection

### List all the movies a user id 2 has rated

    SELECT EXPAND(OUT("rated")) FROM Users WHERE id=2
  
#### The Key Points
  
* If you forget "rated" in OUT("rated") and did just OUT() you will pull both Occupations and Movies nodes
* If you forget EXPAND() you will get the target record IDs and not actual columns in the movies

### List all the movies rated by people under age 25

    SELECT expand(out("rated")) from Users where age < 25
  
### List all the users who are lawyers to prep for the list movies rated by lawyers

    SELECT FROM Users where out("hasOccupation").description = "lawyer"

* Note that the Users table also has the occupation id, and the lawyer is id = 11 so we can conform the
graph link is proper.

### List movies rated by lawyers
  
We will build on the all above examples. The key is to feed the set of users in to the top level query.

    SELECT EXPAND(OUT("rated")) FROM (
      SELECT FROM Users where out("hasOccupation").description = "lawyer"
      
### With Traversal

    select from (traverse * from (select from users where id=11)) where @class='Movies'
    
### List all the "Crime" movies that were rated by "Lawyers" 

This is fairly complex considering that we have only 4 types of edges and 4 types of entities.

1. Get all the users who are the lawers
2. Get all the movies the lawers rated
3. Get the lawyer rated movies and have the Genere of "Crime"
4. Bonus sort the movies by title

      SELECT FROM (
      SELECT EXPAND(OUT("rated")) FROM (
          SELECT FROM Users where out("hasOccupation").description = "lawyer")
        ) WHERE OUT("hasGenera").description = "Crime"
      ORDER BY title

#### With Traverse

    select from (traverse * from (
      select * from Occupation where description='lawyer'
      )) where @class='Movies'

#### Tempting, but do not do this. It will take forever!

    select from (traverse * from (
      select * from Occupation where description='lawyer'
      )) where @class='Movies' AND OUT("hasGenera").description = "Crime"
      
Nor this,

    select FROM (
    select from (
      select from (traverse * from (
          select * from Occupation where description='lawyer'
        )) where @class='Movies'
      ) ) WHERE OUT("hasGenera").description = "Crime"


## More on Traverse

.. to be continued from this query which selects Users who are 50 or older who have rated Drama type movies.

    select FROM (
    select From (traverse in() From (
      select from Genres where description="Drama")) where @class="Users")
        WHERE age > 50




  
  
