# Question 3: Matchmaking

## Background

Every year, the Senate Appropriations Committee solicits funding requests from Senators for specific projects, accounts, or legislative language. This process contributes to the annual appropriations bill, such as the [FY24 bill](https://www.congress.gov/bill/118th-congress/house-bill/2882) passed in March 2024. Member offices sometimes submit duplicate requests, particularly for Congressionally Directed Spending (CDS). CDS requests are requests for specific projects (e.g., to repair a bridge in Seattle, WA). More information on funded CDS requests for FY24 can be found [here](https://www.appropriations.senate.gov/fy-2024-congressionally-directed-spending).

## Part A: SQL

Suppose the CDS request data is stored in a SQL database with two tables: `requests` and `senators`. Below is a sample of the data in both tables.

### Schema: (CDS) Requests

The `requests` table consists of unique request IDs, Senator IDs, titles, and recipients. This table includes only CDS requests.

| request_id | senator_id | title                           | recipient                              |
| ---------- | ---------- | ------------------------------- | -------------------------------------- |
| 43829      | 1          | Improved pavement on Highway 23 | Minnesota Department of Transportation |
| 43830      | 2          | Community Center for the Arts   | Community Arts Initiative Inc.         |
| 43831      | 3          | Entrepreneurial Fusion Center   | University of Mississippi              |

### Schema: Senators

The `senators` table consists of unique IDs, last names, party affiliations, and states.

| senator_id | last_name | party      | state |
| ---------- | --------- | ---------- | ----- |
| 1          | Murray    | Democrat   | WA    |
| 2          | Collins   | Republican | ME    |
| 3          | Wicker    | Republican | MS    |

### Question A.(i)

Write a SQL query (in your SQL dialect of choice) that lists the `request_id`s for requests made by Democrat Senators.

#### Answer:
```
select request_id
from requests as rt, senators as st
where rt.senator_id = st.senator_id
and st.party='Democrat';
```

### Question A.(ii)

Each state is represented by two Senators, but not all Senators make CDS requests. Write a SQL query that lists all states where only one of the two Senators made a request.

#### Answer:
```
select state
from (
	select st.state, count(st.state) as total
	from senators as st
	FULL JOIN requests as rt on st.senator_id = rt.senator_id
	where rt.request_id is null
	group by st.state
)
where total=1;
```

## Part B: Architecture Design

### Question B.(i)

Suppose you are building a system that lets staff track duplicate requests. It provides the following features:

1. Staff can mark CDS requests A and B as duplicates.
2. More than two requests can be duplicates. If A and B are already marked as duplicates, and staff mark C as a duplicate of B, all three requests (A, B, and C) should be considered duplicates.
3. Staff can export all duplicate requests into a CSV or Excel file. The exact format does not matter, but the backend architecture should allow for the efficient retrieval of all duplicate requests. For example, if A, B, and C are considered duplicates, the export should list them as such.

To the best of your ability, describe how you would architect such a system that satisfies the above requirements. In your answer, make sure to respond to these questions:

1. What kind of database would you use to store the data?
2. How might you design the data schema? In particular, how would you efficiently track groups of duplicate requests?
3. For questions 1 and 2, what other options did you consider? What are the tradeoffs between your selected approach and others you might have taken?

We do not expect you to have a perfect or fully fleshed out solution. Feel free to describe or draw out an answer. We just want to see your thought process!

#### Answer:
Use a relational database like Postgresql where in addition to the existing tables "requests" and "senators" another table called "duplicate_requests" will store request_id (integer) and the request_id_duplicates (integer) which indicates they are duplicated like below. The table will have a unique constraint when combining both columns.

| request_id |  request_id_duplicates  |
| ---------- | ----------------------- |
| A          | B                       |
| B          | A                       |

When staff marks B as a duplicate of A, for example A is 1 and B is 2. The table will insert (request_id, request_id_duplicates ) values (2, 1) after validating the unique contraint that the combination previously does not exist. The backend will then check if A (1) is a duplicate of any request in order to add more entires to the table but in this case there is none. It will also insert the inverse relation (request_id,  request_id_duplicates) values (1, 2) after validating the unique contraint. Like below:

| request_id |   request_id_duplicates |
| ---------- | ----------------------- |
| 2          | 1                       |
| 1          | 2                       |

When staff marks C as a duplicate, for example C is 3. The table will insert (request_id, request_id_duplicates ) values (3, 2) after validating the unique contraint that the combination previously does not exist. The backend will then check if B (2) is a duplicate of any request. In this case, B (2) is a duplicate of A (1) so the table will insert (request_id, request_id_duplicates ) values (3, 1) after validating the unique contraint. The backend will then check if A (1) is a duplicate of any request. In this case, B (2) duplicates A (1) so the table will attempt to insert (request_id, request_id_duplicates) values (3, 2) but will be unsuccessful since it violated the unqiue constraint suggesting the chain of duplication was all checked. Lastly, it will add the inverse relation by inserting (request_id, request_id_duplicates) values (1, 3) and (2,3) after validating the unique contraint. 

| request_id |  request_id_duplicates  |
| ---------- | ----------------------- |
| 2          | 1                       |
| 1          | 2                       |
| 3          | 2                       |
| 3          | 1                       |
| 1          | 3                       |
| 2          | 3                       |

When the user wants to retrieve duplicates requests as a csv/excel file they can use the table "duplicate_requests" by selecting request_id and grouping by request_id and aggregating on request_id_duplicates to get groups of requests that are duplicates, like below:

| request_id | array_agg(request_id_duplicates)|
| ---------- | ------------------------------- |
| 2          | {1,3}                           |
| 1          | {2,3}                           |
| 3          | {2,1}                           |

Another approach that could have been taken is adding the columns "is_duplicate" (boolean) and "duplicate_of_request_id" (integer) to the existing table schema "requests". When staff marks B as a duplicate of A, for the row where request is B it will update the column is_duplicate to true and duplicate_of_request_id to A. When staff marks C as a duplicate of B, for the row where request is C it will update the column is_duplicate to true and duplicate_of_request_id to B. With this approach it is a simpiler design but to find the group of duplicates will need to make multiple select calls to find the chain of duplicates using the "duplicate_of_request_id" field, each time an export needs to be done. With the preferred approach, it is a bit more work to keep track of duplicates but each time an export needs to be done it will a be lot faster as the chain of duplicates has already been calculated.

### Question B.(ii)

Suppose staff want the ability to mark two CDS requests as (1) confidently distinct (i.e., **not** duplicates) or (2) potential duplicates for later discussion.

Describe how you would architect such a system. In your answer, describe alternatives you considered and the tradeoffs of each compared to your preferred design.

We do not expect you to have a perfect or fully fleshed out solution. Feel free to describe or draw out an answer. We just want to see your thought process!

#### Answer:
Use a relational database like Postgresql where in addition to the existing tables "requests" and "senators" another table called "connected_requests" will store request_id (integer), request_id_to (integer), and connection_type (character varying) which indicates they are connected (or not) and how they are connected. The table will have a unique constraint when combining both columns request_id and request_id_to. The connection_type can store either "potential duplicates" or "confidently distinct". When staff marks request 1 and request 2 as potential duplicates, the table will insert (request_id, request_id_to, connection_type ) values (1, 2, 'potential duplicates') after validating the unique contraint that the combination previously does not exist. It will also insert the inverse relation with same connection_type (request_id, request_id_to, connection_type ) values (2, 1, 'potential duplicates') after validating the unique contraint. Like below:

| request_id | request_id_to   |   connection_type    |
| ---------- | --------------- | -------------------- |
| 1          | 2               | potential duplicates |
| 2          | 1               | potential duplicates |
| 3          | 2               | confidently distinct |
| 2          | 3               | confidently distinct |

Another approach that could have been taken is adding the columns "is_potentially_connected" (boolean) and "request_id_to" (integer) to the existing table schema "requests". When staff marks request B as a potential duplicate of A, for the row where request is B it will update the column is_potentially_connected to true and request_id_to to A. The inverse will also be added for request A to be marked true as a potential duplicate of B. However, when the staff marks request B as confidently distinct from request A, for the row where request is B it will update the column is_potentially_connected to false and request_id_to to A. The inverse will also be added for request A to be marked false as a potential duplicate of B. This approach has a simple schema change of adding a boolean field to determine if there is no duplication or potential duplication. With this approach, however, is if in the future we mark 2 requests as definite duplicates then this approach will not be able to capture this information. We would need to add another boolean column as "is_duplicate" which can then get confusing with the column "is_potentially_connected". With the preferred approach, if we need to keep track of definite duplicates when can just add that relation as a new connection_type.