



      Column        |               Type               | Collation | Nullable |                   Default                    
---------------------+----------------------------------+-----------+----------+----------------------------------------------
 id                  | bigint                           |           | not null | rel_8192_510.next_id()
 value               | character varying(2000)          |           | not null | 
 location            | json                             |           |          | 
 user_id             | integer                          |           | not null | 
 liked_user_ids      | integer[]                        |           |          | 
 created_time        | timestamp without time zone      |           | not null | timezone('UTC'::text, now())
 updated_time        | timestamp without time zone      |           | not null | timezone('UTC'::text, now())
 status              | rel_8192_510.status              |           | not null | 'default'::rel_8192_510.status
 visibility          | rel_8192_510.visibility          |           |          | 
 muted               | boolean                          |           |          | 
 moment_type         | rel_8192_510.moment_type         |           |          | 
 topic_ids           | bigint[]                         |           |          | 
 group_id            | bigint                           |           |          | 
 hyperlink           | character varying(1024)          |           |          | 
 user_set_visibility | rel_8192_510.user_set_visibility |           | not null | 'everyone'::rel_8192_510.user_set_visibility
 activity_id         | bigint                           |           |          | 
 additional_data     | json                             |           |          | 
 operation_topic_ids | bigint[]                         |           |          | 
 share_my_vote       | boolean                          |           | not null | false
 allow_forward       | rel_8192_510.forward_setting     |           | not null | 'allow'::rel_8192_510.forward_setting
 notify_users        | json                             |           |          | 
Indexes:
    "moments_pkey" PRIMARY KEY, btree (id)
    "moments_user_id_idx" btree (user_id)



## 查看生成id的next_id方法

select * from pg_proc where proname='next_id';


 SELECT (FLOOR(EXTRACT(EPOCH FROM clock_timestamp()) * 1000) /* now_millis */ +
                   - 1314220021721 /* our_epoch */)::bigint << 23                   +
             | (1 + 0 /* shard_id */ << 10)                                         +
             | (nextval('rel_8192_0.rel_id_seq') % 1024 /* seq_id */);              +
 
                                                                                    +
       SELECT (FLOOR(EXTRACT(EPOCH FROM clock_timestamp()) * 1000) /* now_millis */ +
                   - 1314220021721 /* our_epoch */)::bigint << 23                   +
             | (1 + 1 /* shard_id */ << 10)                                         +
             | (nextval('rel_8192_1.rel_id_seq') % 1024 /* seq_id */);              +








 Column       |            Type             | Collation | Nullable |           Default            
-------------------:|:-----------------------------:|:-----------:|:----------:|:------------------------------
 id                | bigint                      |           | not null |
 comment_id         | bigint                      |           | not null | 
 moment_id         | bigint                      |           | not null | 
 moment_owner_id         | bigint               |           | not null | 
 user_id           | integer                     |           | not null | 
 other_user_id     | integer                     |           | not null | 
 created_time      | timestamp without time zone |           | not null | timezone('UTC'::text, now())
 


 Column     |            Type             | Collation | Nullable |           Default            
----------------:|:-----------------------------:|:-----------:|:----------:|:------------------------------
 id             | bigint                      |           | not null | rel_8192_0.next_id()
 moment_id      | bigint                      |           | not null | 
 moment_user_id | integer                     |           | not null | 
 user_id        | integer                     |           | not null | 
 created_time   | timestamp without time zone |           | not null | timezone('UTC'::text, now())
 updated_time   | timestamp without time zone |           | not null | timezone('UTC'::text, now())
 recalled       | boolean                     |           | not null | false
 status         | rel_8192_0.status           |           | not null | 'default'::rel_8192_0.status
 attitude_id    | integer                     |           |          | 

    Column    |            Type             | Collation | Nullable |           Default            
--------------:|:-----------------------------:|:-----------:|:----------:|:------------------------------
 comment_id   | bigint                      |           | not null | 
 message_id   | bigint                      |           |          | 
 user_id      | integer                     |           | not null | 
 created_time | timestamp without time zone |           | not null | timezone('UTC'::text, now())
 updated_time | timestamp without time zone |           | not null | timezone('UTC'::text, now())
 recalled     | boolean                     |           | not null | false
 status       | rel_8192_0.status           |           | not null | 'default'::rel_8192_0.status
