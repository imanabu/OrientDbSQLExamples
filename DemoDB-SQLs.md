# OrientDB "DemoDB" SQL Collections (More Advanced)

![Demo Schema Example](https://github.com/imanabu/OrientDbSQLExamples/blob/master/demoDB.png)

## The Background

OrientDB (ODB) ships the **DemoDB** and usually it is installed out of the
box.

This DB contains a "Travel Industry" demo data where you can traverse a
diversity of edges to discover a travellers path. 

Should this have been a **live** real data and if you are a marketing consultant
to a travel agency, we can use the OrientDB's Graph Query capabilities to
find situations like.

* Where did Luca eat during his visit to Italy?
* List who visited `Castel Drena` and out of them which restaurant they
  visited.
* List who are the friends and friends of someone and see where they have all visited.

Likewise, if you have a set of health records, you could ask questions like:

Who have visited our hospital in the past 30 days with a symptom of cold and if
any of the patients' friends also have visited for the same situation, and then
trace who is likely the first person who spread the cold etc.

Now imagine yourself as a travel agent in Milano Italy and see how you can form
your agency's advertisement campaigns based on the data with the OrientDB SQL.

## What You Need

Be sure that the DemoDB is loaded. If you did not load it, you can load it
again from the top right corner of the OrientDB studio login box.

## Customer and Profiles

Customer => HasProfile => Profiles

Customer is unique to a one person, a customer could have more than one profiles, though not normally.

Profile has the Email, name and Bio and Customer has a phone number only. The assumption is that
an individual can only have one phone number that will uniquely identify as a person. Not quite
true but for playing around, let's assume that is the case. 

A profile links to the person's friends using the Edges labelled as "HasFriend". 
A customer links to a whole bunch of interesting records like places, restaurants, hotels where that
customer has visited. 

### Who are the friends of Luca?

        select expand(out("HasFriend")) from Profiles where Email = "luca@example.com"

Luca has 10 friends. But nobody considers Luca as friends :-( because there is no incoming "HasFriend" link to him.

To find that out just change  expand(out("HasFriend")) to  expand(in("HasFriend"))

We will come back to this as we traverse other stuff.

### What's Luca's Customer ID?

        select expand(in("HasProfile")) from Profiles where Email = "luca@example.com"

On my install it is #194:0 If you need to just derive this records number as a subquery, you would:

        select in("HasProfile") from Profiles where Email = "luca@example.com"

In the following examples, I will just use the record ID.

## Customer is the center node for lots of stuff and not the Profile

So the strategy is to find the customer's ID and if needed be, we can
traverse over to Profile to see who the person really is.

## Where did Luca Visit?

        select expand(out("HasVisited")) from #194:0

## Where did Luca's friends visit? Not Luca himself.

1. Locate Luca's profile
2. Locate Luca's friend
3. Traverse over to their Customer record via HasProfile
4. Traverse through each of the Customer's HasVisited

        select expand(out("HasFriend").in("HasProfile").out("HasVisited")) from Profiles where Email = "luca@example.com"

Note that from here we can find where Luca's friends has stayed and eaten at restaurant. Simply swap
"HasVisited" to "HasStayed" or "HasEaten"

Or you cal pull all of them by just out(), which is the same as out("HasVisited", "HasStayed", "HasEaten")

It's time to send Luca and his friends 2 for 1 campaign coupons if you run the restaurants or a points of interest based on these query results.

## Traversing a bit deeper. Let's find Luca's friends' friends 

We already know that Luca's record ID is #62:0 so we will start from there. Please note that where
it says #62:0 can be a subquery returning many profile records. 

        SELECT $depth FROM
        (Traverse out("HasFriend") from #62:0 MAXDEPTH 10)
       

However, based on this query, we know Luca's friends are 1 level deep. It's a tight knit set of people.
Luca won't be a good candidate to spend a lot of Lira to send a shiny brochure of travel deals.

## Traversing Profile and Who Has Lots of Friends

We now even understand Luca's social style. He will **not** likely be a target of our travel agent 
marketing. Let's see who has lots of friends. 

        SELECT FROM (Traverse out("HasFriend") from Profiles) where $depth > 18

This resulted in only two people, pupjetvir@rogra.co.uk and jeju@su.edu These two people could bring in
a lot of revenue by sending all of them a short Italian coast cruise package.

## Do you want more queries listed? You can:

* Solve and share more SQLs and edit this file and send me a PR. Please create some good and funny examples
  where a "profit minded greedy travel agency" maraketer would love to see.
* Request a query in the form of how the marketing head of a travel agency (without SQL skills) might
  ask and put that in the "Bug" report on this Git Repo.

