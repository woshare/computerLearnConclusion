



moment_like
comment_like
resource_like

moments

moment_comments



rpcclient.MomentRecommendClient.RecallRecommend(ctx, cityCond, search)

MOMENT_RECALL_RECOMMEND_SVC_KEY             = "tantan-moment-recall-recommend-service"
	MOMO_MOMENT_RECALL_RECOMMEND_SVC_KEY        = "momo-moment-recall-recommend-service"


nearby.nearby_moments


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
 biz          | biz                         |           | not null | 
 comment_id         | bigint                      |           | not null | 0
 moment_id         | bigint                      |           | not null | 
 moment_owner_id         | bigint               |           | not null | 
 user_id           | integer                     |           | not null | 
 other_user_id     | integer                     |           | not null | 0
 created_time      | timestamp without time zone |           | not null | timezone('UTC'::text, now())
 
 biz:moment_comment 动态评论,moment_like 动态点赞

moment_like,brand_moment_like

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

comment_like
    Column    |            Type             | Collation | Nullable |           Default            
--------------:|:-----------------------------:|:-----------:|:----------:|:------------------------------
 comment_id   | bigint                      |           | not null | 
 message_id   | bigint                      |           |          | 
 user_id      | integer                     |           | not null | 
 created_time | timestamp without time zone |           | not null | timezone('UTC'::text, now())
 updated_time | timestamp without time zone |           | not null | timezone('UTC'::text, now())
 recalled     | boolean                     |           | not null | false
 status       | rel_8192_0.status           |           | not null | 'default'::rel_8192_0.status

      Column      |            Type             | Collation | Nullable |           Default            
------------------:|:-----------------------------:|:-----------:|:----------:|:------------------------------
 id               | bigint                      |           | not null | rel_8192_0.next_id()
 resource_type    | character varying(128)      |           | not null | 
 resource_id      | bigint                      |           | not null | 
 resource_user_id | integer                     |           | not null | 
 user_id          | integer                     |           | not null | 
 created_time     | timestamp without time zone |           | not null | timezone('UTC'::text, now())
 updated_time     | timestamp without time zone |           | not null | timezone('UTC'::text, now())
 recalled         | boolean                     |           | not null | false
 status           | rel_8192_0.status           |           | not null | 'default'::rel_8192_0.status



### 接口

g.GET("/users/:uid/moments", momentGetFunc) // and with
g.GET("/users/:uid/moments/:mid", momentSingleGetFunc)
g.POST("/users/:uid/moments", momentPostFunc)
g.POST("/users/:uid/moments/:mid", momentPostDispatchFunc)
g.DELETE("/users/:uid/moments/:mid", momentDeleteDeleteFunc)
g.PATCH("/users/:uid/moments/:mid", momentPatchFunc)
g.GET("/notify-users", momentNotifyUsersGetFunc)
g.GET("/cameraCategories", categoryGetFunc) // and with
g.GET("/cameraCategories/:cid", categoryGetFunc)
g.GET("/users/:uid/moments/:mid/likes", likeGetFunc)
g.PUT("/users/:uid/moments/:mid/likes/me", likePutFunc)
g.POST("/users/:uid/moments/:mid/likes/me", likePostFunc)
g.DELETE("/users/:uid/moments/:mid/likes/me", likeDeleteFunc)
g.POST("/users/:uid/moments/:mid/messages", commentPostFunc)
g.GET("/users/:uid/moments/:mid/messages", commentGetFunc)
g.GET("/users/:uid/moments/:mid/messages/:msgid", commentGetByIDFunc)
g.GET("/users/:uid/moments/:mid/messages/:msgid/subMessages", commentReplyGetFunc)
g.DELETE("/users/:uid/moments/:mid/messages/:msgid", commentDeleteFunc)
g.POST("/users/:uid/moments/:mid/messages/:msgid", commentDeleteFunc)
g.PUT("/users/:uid/moments/:mid/messages/:msgid/likes/me", commentLikePutFunc)
g.DELETE("/users/:uid/moments/:mid/messages/:msgid/likes/me", commentLikeDeleteFunc)
g.POST("/users/:uid/moments/:mid/messages/:msgid/likes/me", commentLikeDeleteFunc)
g.GET("/users/:uid/muted", settingMuteGetFunc)
g.POST("/users/:uid/muted/:oid", settingMutePostFunc)
g.PUT("/users/:uid/muted/:oid", feedsettingServer.ParamValidate(), settingMutePutFunc)
g.DELETE("/users/:uid/muted/:oid", feedsettingServer.ParamValidate(), settingMuteDeleteFunc)
g.GET("/users/:uid/hidden", settingHiddenGetFunc)
g.POST("/users/:uid/hidden/:oid", settingHiddenPostFunc)
g.PUT("/users/:uid/hidden/:oid", feedsettingServer.ParamValidate(), settingHiddenPutFunc)
g.DELETE("/users/:uid/hidden/:oid", feedsettingServer.ParamValidate(), settingHiddenDeleteFunc)
g.GET("/users/:uid/musics", musicServerGetFunc)
g.GET("/cameraStickers", cameraStickerGetFunc)
g.GET("/cameraFilters", cameraFilterGetFunc)
g.GET("/musicCategories", musicCategoryGetFunc)
g.GET("/musics", musicFeedGetFunc)
g.PUT("/users/:uid/musics/:music_id/favors/me", musicFavorPutFunc)
g.POST("/users/:uid/musics/:music_id/favors/me", musicFavorPostFunc)
g.DELETE("/users/:uid/musics/:music_id/favors/me", musicFavorDeleteFunc)
g.GET("/topics", topicFeedGetFunc)
g.GET("/topics/:topicId", topicSingleGetFunc)
g.GET("/search/topics", topicSearchFunc)
g.GET("/users/:uid/topics-search-histories", topicSearchHistoryGetFunc)
g.PUT("/users/:uid/topics-search-histories/:topicId", topicSearchHistoryPutFunc)
g.DELETE("/users/:uid/topics-search-histories", topicSearchHistoryDeleteFunc)
g.POST("/users/:uid/topics-search-histories/:topicId", topicSearchHistoryPostFunc)
g.GET("/topicCategories", server2.Wrapper(topicCategoryServer.GET, filter.WithTopic, filter.WithMoment, filter.WithUser))
g.POST("/users/:uid/topics/:tid/comments", topicCommentPostFunc)
g.GET("/users/:uid/topics/:tid/comments", topicCommentGetFunc)
g.DELETE("/users/:uid/topics/:tid/comments/:cid", topicCommentDeleteFunc)
g.POST("/users/:uid/topics/:tid/comments/:cid", topicCommentDeleteFunc)
g.GET("/users/:uid/topics/:tid/comments/:cid/subComments", topicCommentReplyGetFunc)
g.GET("/users/:uid/topics", userTopicGetFunc)
g.PUT("/users/:uid/topics/:tid/comments/:cid/likes/me", topicCommentLikePutFunc)
g.DELETE("/users/:uid/topics/:tid/comments/:cid/likes/me", topicCommentLikeDeleteFunc)
g.POST("/users/:uid/topics/:tid/comments/:cid/likes/me", topicCommentLikeDeleteFunc)
g.PUT("/users/:uid/topics/:tid/votes/:vid", topicCommentPutFunc)
g.GET("/followConfigs", serviceConfigGetFunc)
g.GET("/guideCounters", server2.Wrapper(serviceConfigServer.GETCounters))
g.GET("/operationGuides", operationGuideGetFunc)
g.GET("/users/:uid/moments/:mid/viewers", server2.Wrapper(momentViewerServer.GET, filter.WithUser))
g.POST("/moment-viewers", server2.Wrapper(momentViewerServer.POST))
g.GET("/pick-users", pickUserGetFunc)
g.GET("/pick-user-counters", server2.Wrapper(pickUserServer.GetCounter))
g.GET("/activity-users", activityUserGetFunc)
g.POST("/activity-users/:actorId", activityUserPostFunc)
g.DELETE("/activity-users/:actorId", activityUserDelFunc)
g.GET("/states", stateGetFunc)
g.GET("/momentTags", momentTagGetFunc)
g.POST("/users/:uid/states", userStatePostFunc)
g.GET("/users/:uid/states", userStateGetFunc)
g.GET("/users/:uid/states/:sid", userStateGetFunc)
g.POST("/users/:uid/states/:sid", userStateDeleteFunc)
g.DELETE("/users/:uid/states/:sid", userStateDeleteFunc)
g.PUT("/users/:uid/states/:sid/likes/me", stateLikePutFunc)
g.POST("/users/:uid/states/:sid/likes/me", stateLikePostFunc)
g.DELETE("/users/:uid/states/:sid/likes/me", stateLikeDeleteFunc)
g.PATCH("/users/:uid/state-counters", stateCounterPatchFunc)
g.GET("/users/:uid/state-counters", stateCounterGetFunc)
g.POST("/users/:uid/state-counters", stateCounterPostFunc)
g.GET("/users/:uid/hotLevel/:otherId", server2.Wrapper(hotLevelServer.GET))
g.GET("/users/:uid/moment-visitors", server2.Wrapper(visitorServer.GET, filter.WithUser))
g.GET("/users/:uid/visitor/counter", server2.Wrapper(visitorServer.GetCounter))
g.GET("/users/:uid/moment-settings", server2.Wrapper(momentSettingsServer.GET))
g.PATCH("/users/:uid/moment-settings", server2.Wrapper(momentSettingsServer.PATCH))
g.POST("/users/:uid/moment-settings", server2.Wrapper(momentSettingsServer.POST))
g.PUT("/moment-feedback", server2.Wrapper(feedbackRestServer.PUT))
g.POST("/moment-feedback", server2.Wrapper(feedbackRestServer.PUT))
g.PUT("/operation-moments", operationMomentPutFunc)
g.POST("/users/:uid/moment-views", momentViewPostFunc)
g.PATCH("/users/:uid/feed-counters", feedCounterPatchFunc)
g.POST("/users/:uid/feed-counters", feedCounterPostFunc)
g.GET("/users/:uid/latest-moments", server2.Wrapper(nearbyMomentServer.GET))
g.GET("/topics/:topicId/activity", server2.Wrapper(service.GET))
g.GET("/users/:uid/publishGuide", server2.Wrapper(service.GetPublishGuide))
g.POST("/users/:uid/award", receiveAwardFunc)
g.GET("/users/:uid/popWindows", popWindowFunc)
g.GET("/thirdshare/moments", thirdshareMomentServer.GET)
g.POST("/activities", activityPostFunc)
g.GET("/activities", activityGetFunc)
g.PATCH("/activities", activityPatchFunc)
g.DELETE("/activities", activityDeleteFunc)
g.DELETE("/activities/:activityId", activityDeleteByIDFunc)
g.POST("/activities/:activityId", activityDeleteByIDFunc) // IOS不支持DELETE，兼容
g.GET("/moments", feedGetFunc)


### 活跃接口
GET /v2/users/:uid/moments 200 798	1.43 K	1.32 K	1.37 K
GET /v2/operationGuides 200 472	859	781	805
GET /v2/topics 200 417	756	688	706
GET /v2/moments 200 390	719	666	678
GET /v2/followConfigs 200 382	1.00 K	701	649
POST /v2/users/:uid/moment-views 204 365	685	611	644
GET /v2/cameraCategories 200 319	590	530	564
GET /v2/users/:uid/moments/:mid 200 227	416	375	396
GET /v2/users/:uid/latest-moments 200 211	382	342	358
GET /v2/activities 200 188	377	328	328
PUT /v2/users/:uid/moments/:mid/likes/me 201 117	233	211	211
GET /v2/users/:uid/muted 200 84.5	173	144	148
GET /v2/notify-users 200 67.0	142	113	116
GET /v2/users/:uid/moment-settings 200 53.8	99.9	87.4	94.0
GET /v2/activity-users 200 25.4	52.2	42.9	47.7
GET /v2/users/:uid/moments 403 25.5	53.6	42.5	43.1
POST /v2/activities 204 16.0	31.7	26.1	28.4
GET /v2/users/:uid/moments/:mid/messages 200 15.2	28.0	23.3	25.1
GET /v2/search/topics 200 10.9	27.4	19.5	22.7
POST /v2/users/:uid/moments/:mid/messages 201 7.32	15.9	12.9	12.8
GET /v2/topicCategories 200 5.87	15.7	9.84	9.10
POST /v2/activity-users/:actorId 200 1.03	5.87	3.05	5.87
GET /v2/cameraFilters 200 2.30	5.57	4.10	4.60
POST /v2/users/:uid/feed-counters 204 2.37	5.97	4.25	4.57
PATCH /v2/users/:uid/feed-counters 204 2.30	6.30	4.88	4.33
GET /v2/users/:uid/moments/:mid 403 2.33	6.50	4.17	4.03
GET /v2/users/:uid/moments/:mid/messages/:msgid/subMessages 200 1.13	3.60	2.25	3.20
DELETE /v2/activity-users/:actorId 200 0.533	8.20	3.46	3.17
POST /v2/users/:uid/moments 201 1.70	4.00	2.69	3.13
GET /v2/users/:uid/topics 200 1.37	4.50	2.48	3.10
GET /v2/users/:uid/moments/:mid/messages/:msgid 200



