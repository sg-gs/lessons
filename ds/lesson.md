## Design fundamentals
-   **Infrastructure debt**  - Schema decisions are the most expensive to change.
	- Example: `files` table, removing `deleted` & `removed` fields. **May 21' added > Aug 25', still there.**
-   **Clear semantics**  - Names and types that express exactly what they represent.
	- Example: `users` table, `userId` field, what does it mean? `id` field already exists. This harms maintainability and hardens efforts to understand the underlying data.
-   **Access patterns first**  - Design for how you'll read, not just how you'll write.
	- Example: `workspace_logs` is optimised for reads, writes are not the main concern there, as it is a history of the workspace user actions: write once, read forever.
	

## Efficiency and Trade-offs

-   **Operation frequency**  - Optimize for the 80% most common cases.
-   **Strategic denormalization**  - Pre-calculate what's expensive to compute in real-time.
	- Example: Drive's account usage calculus is not efficient, now [accounting](https://one.eu.newrelic.com/nr1-core/apm-features/databases/NDI4NDc0NHxBUE18QVBQTElDQVRJT058NDc2ODc3NjE2?account=4284744&duration=1800000&state=81561e8e-5b5d-0362-b770-80cd1c7fbb11) for the 61% of the database time. If it was pre-computed, it would be the 5-10%, as the other frequent operations. 
-   **Intentional indexes**  - Each index costs on writes, make sure it's worth it
	- Example: files name index is destroying the performance of the table ([ovh Postgres stats](https://www.ovh.com/manager/#/public-cloud/pci/projects/3f3b87a3727a4dc7848d88101591ad99/databases-analytics/operational/services/0f8b4183-c8a5-408f-ba4d-d0564a483b28/queries) by std dev & files tables accounting for 449G and the indexes account for 160G of that size, ideally it should be 10% or less, the less, the better, as long as the performance of the queries are fine).

## Scalability and Concurrency

-   **Contention points**  - Identify where multiple users will modify the same data
-   **Partitioning strategies**  - How you'll distribute when it grows
-   **Consistency vs availability**  - Which operations need to be immediately consistent

## Integrity and Maintenance

-   **DB-level constraints**  - Don't rely only on application for integrity
-   **Soft delete strategy**  - For data you need to preserve but hide
-   **Basic audit fields**  - created_at, updated_at as minimum
-   **Schema versioning**  - How you'll migrate without downtime

## Product Considerations

-   **Hidden non-functional requirements**  - Reports, analytics, future integrations
-   **Regulations and compliance**  - GDPR, audits, data retention
-   **Internationalization**  - If you'll need multiple languages/locales


## Exercise: Collaborative Comments and Review System (10')
### Product Context

Your company wants to add a  **collaborative document review system**  similar to Google Docs comments, but with advanced capabilities:

-   Users can leave comments on files (only text)
-   Comments can be conversation threads (nested replies)
-   Comments can have @mentions to other users
-   Threads can be resolved/reopened
-   Need to show recent activity per file and per workspace
-   Users want to see "all my unresolved threads" across all workspaces.
-   Analytics: which files are most discussed? most active users in reviews?

### Business Requirements
1.  **High volume**: An enterprise workspace can have 10,000+ active comments.
2.  **Frequent reads**:
    -   View file comments (95% of operations).
    -   "My pending comments" dashboard (queried 20+ times/day per user).
    -   Workspace activity feed (real-time).
3.  **Moderate writes**: 100-500 new comments per day in large workspaces.
