// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

```
Table users {
  id varchar
}

Table files {
  uuid varchar [primary key]
  user_id integer
  created_at timestamp
  updated_at timestamp
}

// index file_id, total_comments
Table files_threads {
  id varchar [primary key]
  total_comments number 
  resolved boolean
  file_id varchar
}

// index files_threads_id
// index files_threads_id, order
// index files_threads_id, resolved
// index files_threads_id, resolved, user_id
// index files_threads_id, user_id
Table files_threads_comments {
  id varchar [primary key]
  files_threads_id varchar
  resolved boolean
  order number 
  text varchar
  user_id varchar
}

// index files_threads_comments_id
Table comments_mentions {
  id varchar [primary key]
  files_threads_comments_id varchar 
  user_id varchar
}

Ref: users.id > comments_mentions.user_id
Ref: files.uuid > files_threads.id
Ref: files_threads_comments.files_threads_id > files_threads.id
Ref: comments_mentions.files_threads_comments_id > files_threads_comments.id
```
