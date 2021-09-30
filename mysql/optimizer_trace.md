-- set optimizer_trace="enabled=ON";

-- ALTER TABLE daily_task_word add  INDEX `cuw` (`client_time`,`uid`,`word_status`);
-- ALTER TABLE daily_task_word drop index `cuw`;
-- ALTER TABLE daily_task_word add  INDEX `uwc` (`uid`,`word_status`,`client_time`);
-- ALTER TABLE daily_task_word drop index `uwc`;
explain select * from daily_task_word where uid='1007168' and word_status=1 and client_time BETWEEN '2020-03-18 04:00:00' and '2020-03-19 03:59:59' order  by client_time desc;

-- explain select * from daily_task_word FORCE INDEX (cuw) where uid='1007168' and word_status=1 and client_time BETWEEN '2020-03-18 04:00:00' and '2020-03-19 03:59:59' order  by client_time desc

-- select * from information_schema.OPTIMIZER_TRACE;
-- 
-- SET OPTIMIZER_trace="enabled=OFF"

## 优化器执行计划

```
{
	"steps": [
		{
			"join_preparation": {
				"select#": 1,
				"steps": [
					{
						"expanded_query": "/* select#1 */ select `daily_task_word`.`id` AS `id`,`daily_task_word`.`uid` AS `uid`,`daily_task_word`.`word_status` AS `word_status`,`daily_task_word`.`word` AS `word`,`daily_task_word`.`belong_type` AS `belong_type`,`daily_task_word`.`belong` AS `belong`,`daily_task_word`.`meaning_cn` AS `meaning_cn`,`daily_task_word`.`client_time` AS `client_time`,`daily_task_word`.`server_time` AS `server_time` from `daily_task_word` where ((`daily_task_word`.`uid` = '1007168') and (`daily_task_word`.`word_status` = 1) and (`daily_task_word`.`client_time` between '2020-03-18 04:00:00' and '2020-03-19 03:59:59')) order by `daily_task_word`.`client_time` desc"
					}
				]
			}
		},
		{
			"join_optimization": {
				"select#": 1,
				"steps": [
					{
						"condition_processing": {
							"condition": "WHERE",
							"original_condition": "((`daily_task_word`.`uid` = '1007168') and (`daily_task_word`.`word_status` = 1) and (`daily_task_word`.`client_time` between '2020-03-18 04:00:00' and '2020-03-19 03:59:59'))",
							"steps": [
								{
									"transformation": "equality_propagation",
									"resulting_condition": "((`daily_task_word`.`uid` = '1007168') and (`daily_task_word`.`client_time` between '2020-03-18 04:00:00' and '2020-03-19 03:59:59') and multiple equal(1, `daily_task_word`.`word_status`))"
								},
								{
									"transformation": "constant_propagation",
									"resulting_condition": "((`daily_task_word`.`uid` = '1007168') and (`daily_task_word`.`client_time` between '2020-03-18 04:00:00' and '2020-03-19 03:59:59') and multiple equal(1, `daily_task_word`.`word_status`))"
								},
								{
									"transformation": "trivial_condition_removal",
									"resulting_condition": "((`daily_task_word`.`uid` = '1007168') and (`daily_task_word`.`client_time` between '2020-03-18 04:00:00' and '2020-03-19 03:59:59') and multiple equal(1, `daily_task_word`.`word_status`))"
								}
							]
						}
					},
					{
						"table_dependencies": [
							{
								"table": "`daily_task_word`",
								"row_may_be_null": false,
								"map_bit": 0,
								"depends_on_map_bits": []
							}
						]
					},
					{
						"ref_optimizer_key_uses": [
							{
								"table": "`daily_task_word`",
								"field": "uid",
								"equals": "'1007168'",
								"null_rejecting": false
							}
						]
					},
					{
						"rows_estimation": [
							{
								"table": "`daily_task_word`",
								"range_analysis": {
									"table_scan": {
										"rows": 730,
										"cost": 153.1
									},
									"potential_range_indices": [
										{
											"index": "PRIMARY",
											"usable": false,
											"cause": "not_applicable"
										},
										{
											"index": "uid",
											"usable": true,
											"key_parts": [
												"uid",
												"word",
												"client_time"
											]
										},
										{
											"index": "server_time",
											"usable": true,
											"key_parts": [
												"client_time",
												"belong_type",
												"belong",
												"id"
											]
										},
										{
											"index": "cuw",
											"usable": true,
											"key_parts": [
												"client_time",
												"uid",
												"word_status",
												"id"
											]
										}
									],
									"setup_range_conditions": [],
									"group_index_range": {
										"chosen": false,
										"cause": "not_group_by_or_distinct"
									},
									"analyzing_range_alternatives": {
										"range_scan_alternatives": [
											{
												"index": "uid",
												"ranges": [
													"1007168 <= uid <= 1007168"
												],
												"index_dives_for_eq_ranges": true,
												"rowid_ordered": false,
												"using_mrr": false,  //MRR 是 MySQL 针对特定查询的一种优化手段。假设一个查询有二级索引可用，读完二级索引后要回表才能查到那些不在当前二级索引上的列值，由于二级索引上引用的主键值不一定是有序的，因此就有可能造成大量的随机 IO，如果回表前把主键值给它排一下序，那么在回表的时候就可以用顺序 IO 取代原本的随机 IO。
												"index_only": false,
												"rows": 22,
												"cost": 27.41,
												"chosen": true
											},
											{
												"index": "server_time",
												"ranges": [
													"2020-03-18 04:00:00.000 <= client_time <= 2020-03-19 03:59:59.000"
												],
												"index_dives_for_eq_ranges": true,
												"rowid_ordered": false,
												"using_mrr": false,
												"index_only": false,
												"rows": 10,
												"cost": 13.01,
												"chosen": true
											},
											{
												"index": "cuw",
												"ranges": [
													"2020-03-18 04:00:00.000 <= client_time <= 2020-03-19 03:59:59.000"
												],
												"index_dives_for_eq_ranges": true,
												"rowid_ordered": false,
												"using_mrr": false,
												"index_only": false,
												"rows": 10,
												"cost": 13.01,
												"chosen": false,
												"cause": "cost"
											}
										],
										"analyzing_roworder_intersect": {
											"usable": false,
											"cause": "too_few_roworder_scans"
										}
									},
									"chosen_range_access_summary": {
										"range_access_plan": {
											"type": "range_scan",
											"index": "server_time",
											"rows": 10,
											"ranges": [
												"2020-03-18 04:00:00.000 <= client_time <= 2020-03-19 03:59:59.000"
											]
										},
										"rows_for_plan": 10,
										"cost_for_plan": 13.01,
										"chosen": true
									}
								}
							}
						]
					},
					{
						"considered_execution_plans": [
							{
								"plan_prefix": [],
								"table": "`daily_task_word`",
								"best_access_path": {
									"considered_access_paths": [
										{
											"access_type": "ref",
											"index": "uid",
											"rows": 22,
											"cost": 19.4,
											"chosen": true
										},
										{
											"access_type": "range",
											"rows": 8,
											"cost": 15.01,
											"chosen": true,
											"use_tmp_table": true
										}
									]
								},
								"cost_for_plan": 15.01,
								"rows_for_plan": 8,
								"sort_cost": 8,
								"new_cost_for_plan": 23.01,
								"chosen": true
							}
						]
					},
					{
						"attaching_conditions_to_tables": {
							"original_condition": "((`daily_task_word`.`word_status` = 1) and (`daily_task_word`.`uid` = '1007168') and (`daily_task_word`.`client_time` between '2020-03-18 04:00:00' and '2020-03-19 03:59:59'))",
							"attached_conditions_computation": [],
							"attached_conditions_summary": [
								{
									"table": "`daily_task_word`",
									"attached": "((`daily_task_word`.`word_status` = 1) and (`daily_task_word`.`uid` = '1007168') and (`daily_task_word`.`client_time` between '2020-03-18 04:00:00' and '2020-03-19 03:59:59'))"
								}
							]
						}
					},
					{
						"clause_processing": {
							"clause": "ORDER BY",
							"original_clause": "`daily_task_word`.`client_time` desc",
							"items": [
								{
									"item": "`daily_task_word`.`client_time`"
								}
							],
							"resulting_clause_is_simple": true,
							"resulting_clause": "`daily_task_word`.`client_time` desc"
						}
					},
					{
						"refine_plan": [
							{
								"table": "`daily_task_word`",
								"pushed_index_condition": "(`daily_task_word`.`client_time` between '2020-03-18 04:00:00' and '2020-03-19 03:59:59')",
								"table_condition_attached": "((`daily_task_word`.`word_status` = 1) and (`daily_task_word`.`uid` = '1007168'))",
								"access_type": "range"
							}
						]
					},
					{
						"reconsidering_access_paths_for_index_ordering": {
							"clause": "ORDER BY",
							"index_order_summary": {
								"table": "`daily_task_word`",
								"index_provides_order": true,
								"order_direction": "desc",
								"index": "server_time",
								"plan_changed": false
							}
						}
					}
				]
			}
		},
		{
			"join_execution": {
				"select#": 1,
				"steps": []
			}
		}
	]
}


```
