// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

```
Table users {
  uuid varchar [primary key]
}

Table files {
  id varchar [primary key]
  user_id integer
  created_at timestamp
  updated_at timestamp
}

Table files_threads {
  id varchar [primary key]
  file_id varchar
  resolved boolean
  // agg
  total_comments number
  total_mentions number
  involved_users number 
  created_at timestamp
  updated_at timestamp
}

Table thread_comments {
  id varchar [primary key]
  order number
  thread_id varchar
  posted_by varchar
  mentions_id varchar
  content varchar
  resolved boolean
}

Table thread_comments_mention {
  id varchar [primary key]
  thread_comments_id varchar
  mentioned_id varchar 
} 

// Ref user_posts: posts.user_id > users.id // many-to-one

Ref: files.id < files_threads.file_id

Ref threads_comments: thread_comments.thread_id > files_threads.id

Ref: thread_comments.posted_by > users.uuid

Ref: thread_comments_mention.thread_comments_id > thread_comments.id

Ref: thread_comments_mention.mentioned_id > users.uuid
```
