# Entity Resolution

## What defines a business
In the dataset there could be identified three kinds of entities:
- Common single businesses
This type of businesses usually operate in one single location, they usually have one specific field of activity and
while they may have similar names to other small businesses (e.g indiana veg restaurant & indiana exotica) they can be easily
distinguished by their web domain, phone number, or social media links.
- Multi-National Corporations (e.g Tescoma, Marvell)
This type of business cannot be distinguished by emails, phone numbers or locations, the only features that
identfiy them are the website, the name and social media links.
- Business platforms (e.g 1ecard.in)
This type of business offers an online platform for small businesses. The businesses hosted by this
site will show the domain name of the platform so the businesses that operate on an online platform won't be distinguished
by their web domain, neither by their phone very often, but by their company name and social media links.

## Technology used
This project uses the [splink](https://github.com/moj-analytical-services/splink) library developed by the UK Ministry of Justice
and it is one of the most popular approaches when it comes to problems related to record linkage and it implements the [Fellegi-Sunter](https://www.robinlinacre.com/intro_to_probabilistic_linkage/) probabilistic deduplication model.


### Approach
The nature of the problem, determining whether two different records describe the same entity could be approached using
the intuition of what does describe a business and encode it in a probabilistic model which does the deduplication
in a reasonable amount of time (10s-30s for the deduplication step).

Many columns in the dataset were either completely redundant or they had information subsumed to another column and could be removed.

The deduplication process consists of a pre-processing step where the company names and web domains are normalized
such that accidental differences are ignored.

Then the blocking step creates candidate groups for pairwise matching among records.
Records are primarily grouped by web domains. The reason for this choice is that the other candidate column: the company name can
have slight variations depending on the subsidiary making an exact match impossible (which is required for blocking since similarity comparisons are 
expensive and the database api is optimized for exact matches). It has also been observed that an exact match between social media links (the only
one with significant statistical presence being the Facebook link) is the absolute best predictor for a duplicate match, followed by an exact match
on the company name, but since these two scenarios are rare in the dataset they cannot be used for blocking. The only column that produces
exact matches, can reliably relate two records (in the vast majority of cases) and is present in the vast majority of records is the web_domain
hence it was used as the primary blocking criterion. Exact matches on company names, facebook links and social media links are also used for blocking.

After blocking is completed and the model has estimated the m and u parameters for every column, the pairwise comparison begins in which the
relevant columns are matched accordinly. After the pairwise matching probabilities have been generated, they are then used to create the final duplicate
clusters.
