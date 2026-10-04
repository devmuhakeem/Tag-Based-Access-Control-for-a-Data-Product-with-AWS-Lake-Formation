# Tag-Based Access Control for a Data Product with AWS Lake Formation

A hands-on AWS lab where I used Lake Formation tags (LF-tags) to control exactly which columns of a shared dataset two different customer tiers could see — then proved it by logging in as each consumer and querying the data myself.

## Scenario
Acting as a data engineer, the task was to treat a movies dataset as a product with two customer tiers — standard and enterprise — and ensure each tier could see only the data they were entitled to, enforced through policy rather than separate copies of the data.

## What I did

### 1. Validated the underlying AWS Glue job
Reviewed an 8-node Glue ETL job (`transform-movies`) that filled missing rating values, applied column mappings, and appended new rows to the dataset on a 10-minute schedule. Let the scheduled trigger run it, then queried the result in Athena to confirm 4,609 total movie records.

### 2. Defined LF-tags
Created three tag keys with their own value sets:
- `Environment`: Development, Production
- `Customer`: Regular, Enterprise
- `Confidential`: True, False

### 3. Applied tags across database, table, and column levels
- Tagged the database `Environment=Production`
- Tagged the table `Confidential=False` (inheriting `Environment=Production` from the database)
- Tagged most columns `Customer=Regular`, but tagged two sensitive columns (`rank`, `rating_filled`) `Customer=Enterprise` — restricting them to enterprise customers only, at the column level

### 4. Revoked default IAM access and granted tag-based permissions instead
Revoked the default `IAMAllowedPrincipals` access to the database and table, then granted permissions purely through LF-tag combinations:
- **Consumer_A** (regular customer): access where `Confidential=False` AND `Customer=Regular`
- **Consumer_B** (enterprise customer): access where `Confidential=False` AND `Customer` is Regular OR Enterprise

### 5. Verified each consumer's actual view
Logged in as each test user separately and ran the identical query in Athena:
- **Consumer_A** saw 11 of 13 columns — `rank` and `rating_filled` were invisible
- **Consumer_B** saw all 13 columns, including the enterprise-only fields

## Key takeaways
- Tag-based access control (TBAC) scales far better than per-resource permission grants — tagging a column once and writing policy against the tag means new resources can inherit the right access automatically just by being tagged consistently
- Combining multiple LF-tag conditions in a single grant is an AND, not an OR — getting that logic backwards would either over- or under-grant access silently
- The only real proof that column-level security works is logging in as the actual restricted user and running the query yourself — trusting the policy configuration alone isn't verification

## Tools
AWS Lake Formation (LF-TBAC), AWS Glue, Amazon Athena, Amazon S3, IAM

---
*Completed as an AWS hands-on lab, including three hands-on challenge tasks.*
