
# Firestore / Supabase Comparison

## Aggregation Queries
### Typical Business Requirements
- How many customers do we have?
- What were the total sales by month for last year?
- etc.

#### Firestore
Since Firestore has no native aggregation capabilities you'll need to either:

1.  Write a custom program to download every (full) document and process it locally on the client.
2.  Create custom code that keeps track of the exact metrics you need to track.  If the code fails or there is a database connectivity issue, you may need to perform process 1 to get back in sync then start your custom code again.

#### Supabase
Ad-hoc aggregation queries are simple and can be run at any time, added to code anywhere, can be easily optimized with indexes, and are way more efficient.

## Partial Document Retrieval
### Typical Business Requirements
- I just want a list of email addresses for all customers
- List the firstname, lastname, and age of all employees

#### Firestore
You cannot retrieve a partial Firestore document.  If you only want to get an email address from 1000 customer documents, you'll need to download all the data in all 1000 documents.  This is inefficient and often insecure as well.

#### Supabase
This is inherent to SQL and to the Supabase APIs.
`select email from customers`
`select firstname, lastname, age from employees`

You can access only the data you need, which is faster and more secure since other sensitive data need never be transmitted.

## Document Write Frequency
### Typical Business Requrements
- Update a set of counts in a table in real time (i.e. show number of users online)
- Update any data shared among multiple users

#### Firestore
Firestore has a hard limit of **1 document update per second**, so this can quickly cause an application to fail as it scales beyond just a few users.  There are workarounds such as sharding that are complex to implement and need to be continually tuned as scaling increases.

#### Supabase
Supabase (PostgreSQL) has no such limit on updating documents, so scaling is not a problem.

## Document Size Limit of 1MB
### Typical Business Requirements
- Users may often input large amounts of data into a single record
- Any application with large record sizes

#### Firestore
Firestore has a hard limit of **1MB (one megabyte)** per document.  You'll need to write code for this in your client so as not to exceed that limit and cause problems with your database and your application.  It may also force complex design changes to your app.

#### Supabase
Supabase (PostgreSQL) allows single records (documents) up to 1.6TB (terrabytes) in size, so you just don't need to worry about this.

## Query Flexibility
### Typical Business Requirements
- Select all single-family homes that have 4-6 bedrooms and are priced between $1,000,000 and $1,250,000
- Find all customers who are NOT from California
- List all customers who have locations within this set of geocoordinates (a bounding box)
- Show me our top customers along with information about the trade association to which they belong

#### Firestore
Firestore's query capabilities are quite limited in comparison to standard SQL.  Some of these limitations are:

- Only one range component per query
- No negation operators (no "NOT" or "NOT IN")
- Limited "OR" capabilities
- No joins or related data (the query must be split or a sub-query must be executed for each document retrieval)

Due to these limitations, a query such as "Select all single-family homes that have 4-6 bedrooms and are priced between $1,000,000 and $1,250,000" can't be performed.  You'd need to pull all the data that satisfied one of the requirements and then filter that set of data on the client, or use some other similar workaround.

This makes basic geo-data queries impossible, because a simple bounding box query would require two ranges (low and high latitude range and low and high longitude range).

#### Supabase
Supabase has all the power of PostgreSQL query capabilities, along with a rich set of additional extensions, such as PostGIS for geo-queries.  SQL is decades more mature as a query language.

## Query Optimization
### Typical Business Requirements
- Allow a user to search for records based on a large set of (optional) criteria
- Show me houses that are in my price range, have at least 3 bathrooms, have a pool, and are in one of the top school districts in the county

A good example of this is a real estate application where a user is allowed to search based on any combinations of dozens of fields.  There may be millions of possible search queries based on what a user is specifically looking for.

#### Firestore
Firestore requires a specific index based on the fields used in any particular query in order to optimize that query.  This requires that you know exactly which fields will be used in a query ahead of time.

By default, Firestore indexes every individual field in every document automatically.  While this is convenient if you're doing a simple lookup on a single field, it's also:

1. useless if your query is more complex than using a single field
2. very wasteful of database space if there are field you may never search on (and you end up paying for the space used by this unused index space)

##### Automatic index link generation
When Firebase senses that a query needs optimization, it will:

1. fail with an error message (so you don't get a result)
2. generate a URL in the error message you can click to create the required index in your database

While it's convenient to get an auto-generated link to create the required indexes you need, is the error console the right place to be doing database optimization?  And if you have a few dozen index combinations to be indexed, do you really want to use a client application to iterate through those combinations, generate error messages for each one, then click on links individually to build indexes?

##### Index limits per collection
Firestore has a limit of 200 "composite" indexes per collection.  So even if you do start creating indexes for each combination of search criteria, you'll hit this limit pretty quickly.

#### Supabase
Unlike Firestore, which can only use a single, composite index to optimize a query, Supabase (PostgreSQL) has an intelligent query optimizer that can use multiple indexes to optimize your query automatically.

You can create indexes on just the fields (or combinations of fields) you might want to search on, and the query optimizer takes care of the rest.  It's more efficient, and you can select which fields you want index ahead of time, so your app works immediately (and you don't need to wait for error messages telling you how to optimize your queries.)

## Backups
### Typical Business Requirements
- Back up critical data in case of failure
- Back up in case of hacking events, programmer or user error

#### Firestore
Firestore has no data backup option.  You can export data, but that's not a backup, and you'd need to write your own routine to extract and restore any important data.

#### Supabase
Supabase allows any third-party tools for PostgreSQL for backing up, including the free, open-source `pg_dump` utility.  You can run full backups or exports on your own at any time.

In addition, Supabase maintains its own backups of your data, and depending on your plan, you can can go back and restore from a previous backup with a single click.

## Document / Data Security Language
### Typical Business Requirements
- Don't allow users to read other users' data
- Validate data that's entered so invalid data can't be stored in the database

#### Firebase
Firebase a limited security and validation capability based on its own new, proprietary language

- This is a new language for your developers to learn and has no applicable use outside of Firestore
- The language has no complex functional capabilities
- Most complex data validation needs to be moved to cloud functions or client functions

#### Supabase
Supabase uses Postgresql RLS (Row Level Security) for security and validation.

- RLS is pure SQL code, so your developers don't need to learn anything new
- RLS, being SQL, is very robust and can call any PostgreSQL functions, which can be written in any of a number of language (SQL, PLPGSQL, JavaScript, Java, etc.)

## Data Duplication Model
### Typical Business Requirements
- Customers may have hundreds of invoices, each of which have hundreds of line items
- Each client has one or more portfolios of work, and each portfolio has many individual projects

#### Firestore
Since Firestore has no relational capability, data duplication is encouraged.  So if you have an invoice for a customer, you may often carry some basic customer information inside the invoice collection to optimize processes like the printing of an invoice.

While this usually works,

1. it duplicates data, leading to additional storage costs as you scale
2. it requires additional coding and workarounds if any related data changes (a company changes their name, for instance)
3. you need to know ahead of time what data you might need in the child records, and those requirements may, and quite often do change

A good example might be that you're carrying the customer name and address in the invoice document so it's easy to print invoices for mailing purposes.  Then a business requirement comes up where a phone number is required (to print a report for the accounts receivable people to call customers regarding past-due invoices.) This will require changing the application code and a data migration routine.  If a phone number or address changes, all old invoices will now be out-of-date as well.

#### Supabase
Supabase, built on PostgreSQL, was built for this.  PostgreSQL is a relational database, which is optimal for related data structures such as this.  No data duplication is necessary (which is much more efficient) and it's much easier to adapt to changing business requirements.

## Data Latency
### Typical Business Requirements
- Realtime applications may require fast response times
- Time-sensitive applications such as financial applications may have low-latency requirements

#### Firestore
Firestore latency times are not guaranteed and can be 10-20x slower than regular websocket broadcasts, as reported here:

[https://medium.com/@d8schreiber/firebase-performance-firestore-and-realtime-database-latency-13effcade26d](https://medium.com/@d8schreiber/firebase-performance-firestore-and-realtime-database-latency-13effcade26d)

#### Supabase
Supabase Realtime latency has the potential to have much lower latency for time-sensitive applications, although at this time there are also no strict guarantees.

## Document Database vs. Relational Database
### Typical Business Requirements
- Some applications need to store blocks of unstructured data or data with a variable structure, or deeply nested data.

#### Firestore
Firestore is, by nature, a document database, so it excels at storing document-based data such as JSON data.  However, there are some limitations to how document data can be stored in or retrieved from a Firestore database.

##### Collection -> Document -> Collection -> Document
While Firestore does support subcollections, all data in Firestore must adhere to a pattern where a collection must contain documents, and a document can contain a collection, which can contain other documents, one of which can have a sub-collection, and so on.  A document can't have a sub-document, nor can a collection have a sub-collection.

Firestore cannot index data deeply nested in a document so it's usually advised that you split things up into sub-collections and index those individually.  This can get complex and difficult when modeling certain data structures that have variable levels of nesting.

#### Supabase
While supbase is based on a relational database (PostgreSQL), its support for JSON (document) data is very powerful and easy to use.  Any PostgreSQL table can have any number of JSON fields which can contain any valid JSON data.

Data inside JSON fields can be indexed just like any other type of field, no matter how deeply nested.  JSON fields can contain objects, string data, numeric data, arrays, and can be nested to any level needed.  This brings all the power of document databases to a relational database, with complete indexing and retrieval capabilities.



