
* ### Connect to mysql database
```bash
mysql -Smysql_socket -hmysql_host -umysql_user -pmyssql_password database_name
```

***

* ### Dump mysql database
```bash
mysqldump -Smysql_socket -hmysql_host -umysql_user -pmyssql_password database_name > dump.sql
```

***

* ### Restore mysql database from dump
```bash
mysql -Smysql_socket -hmysql_host -umysql_user -pmyssql_password database_name < dump.sql
```

* ### Swap MySQL tables
```sql
RENAME TABLE operations_deposits TO tmp_operations_deposits
             __operations_deposits TO operations_deposits
             tmp_operations_deposits TO __operations_deposits;
```

* ### MySQL query performance troubleshooting
```sql
EXPLAIN FORMAT=JSON SELECT accounts.uid, MAX(refs.confirmed_at) AS confirmed_at
         FROM barong_production.accounts
                  INNER JOIN barong_production.accounts AS refs
                             ON accounts.uid = refs.referrer_id_0 OR accounts.uid = refs.referrer_id_1 OR
                                accounts.uid = refs.referrer_id_2 OR accounts.uid = refs.referrer_id_3 OR
                                accounts.uid = refs.referrer_id_4
         WHERE ((accounts.banned_until IS NULL OR accounts.banned_until < NOW()) AND accounts.role="member")
           AND accounts.level >= 1
           AND accounts.confirmed_at IS NOT NULL
           AND refs.level >= 5
           AND refs.confirmed_at IS NOT NULL
           AND refs.confirmed_at >= '2021-04-27 16:00:00'
         GROUP BY accounts.uid;
```

**Before optimization**

```json
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "21198.05"
    },
    "grouping_operation": {
      "using_temporary_table": true,
      "using_filesort": true,
      "cost_info": {
        "sort_cost": "963.80"
      },
      "nested_loop": [
        {
          "table": {
            "table_name": "refs",
            "access_type": "ALL",
            "possible_keys": [
              "search_referrer_0",
              "search_referrer_1",
              "search_referrer_2",
              "search_referrer_3",
              "search_referrer_4"
            ],
            "rows_examined_per_scan": 3213,
            "rows_produced_per_join": 963,
            "filtered": "30.00",
            "cost_info": {
              "read_cost": "546.84",
              "eval_cost": "192.76",
              "prefix_cost": "739.60",
              "data_read_per_join": "14M"
            },
            "used_columns": [
              "id",
              "confirmed_at",
              "level",
              "referrer_id_0",
              "referrer_id_1",
              "referrer_id_2",
              "referrer_id_3",
              "referrer_id_4"
            ],
            "attached_condition": "((`barong_production`.`refs`.`level` >= 5) and (`barong_production`.`refs`.`confirmed_at` is not null))"
          }
        },
        {
          "table": {
            "table_name": "accounts",
            "access_type": "ALL",
            "possible_keys": [
              "index_accounts_on_uid",
              "search_1"
            ],
            "rows_examined_per_scan": 3213,
            "rows_produced_per_join": 963,
            "filtered": "0.03",
            "using_join_buffer": "Block Nested Loop",
            "cost_info": {
              "read_cost": "916.30",
              "eval_cost": "192.76",
              "prefix_cost": "20234.25",
              "data_read_per_join": "14M"
            },
            "used_columns": [
              "id",
              "uid",
              "confirmed_at",
              "role",
              "level",
              "banned_until"
            ],
            "attached_condition": "((isnull(`barong_production`.`accounts`.`banned_until`) or (`barong_production`.`accounts`.`banned_until` < <cache>(now()))) and (`barong_production`.`accounts`.`role` = 'member') and (`barong_production`.`accounts`.`level` >= 1) and (`barong_production`.`accounts`.`confirmed_at` is not null) and ((`barong_production`.`accounts`.`uid` = `barong_production`.`refs`.`referrer_id_0`) or (`barong_production`.`accounts`.`uid` = `barong_production`.`refs`.`referrer_id_1`) or (`barong_production`.`accounts`.`uid` = `barong_production`.`refs`.`referrer_id_2`) or (`barong_production`.`accounts`.`uid` = `barong_production`.`refs`.`referrer_id_3`) or (`barong_production`.`accounts`.`uid` = `barong_production`.`refs`.`referrer_id_4`)))"
          }
        }
      ]
    }
  }
}
```

**After optimization**

```json
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "2489.09"
    },
    "grouping_operation": {
      "using_temporary_table": true,
      "using_filesort": true,
      "cost_info": {
        "sort_cost": "74.33"
      },
      "nested_loop": [
        {
          "table": {
            "table_name": "refs",
            "access_type": "range",
            "possible_keys": [
              "search_referrer_0",
              "search_referrer_1",
              "search_referrer_2",
              "search_referrer_3",
              "search_referrer_4",
              "tmp_confirmed_at"
            ],
            "key": "tmp_confirmed_at",
            "used_key_parts": [
              "confirmed_at"
            ],
            "key_length": "6",
            "rows_examined_per_scan": 223,
            "rows_produced_per_join": 74,
            "filtered": "33.33",
            "index_condition": "((`barong_production`.`refs`.`confirmed_at` is not null) and (`barong_production`.`refs`.`confirmed_at` >= '2021-04-27 16:00:00'))",
            "cost_info": {
              "read_cost": "298.34",
              "eval_cost": "14.87",
              "prefix_cost": "313.21",
              "data_read_per_join": "1M"
            },
            "used_columns": [
              "id",
              "confirmed_at",
              "level",
              "referrer_id_0",
              "referrer_id_1",
              "referrer_id_2",
              "referrer_id_3",
              "referrer_id_4"
            ],
            "attached_condition": "(`barong_production`.`refs`.`level` >= 5)"
          }
        },
        {
          "table": {
            "table_name": "accounts",
            "access_type": "ALL",
            "possible_keys": [
              "index_accounts_on_uid",
              "search_1",
              "tmp_confirmed_at"
            ],
            "rows_examined_per_scan": 3214,
            "rows_produced_per_join": 74,
            "filtered": "0.03",
            "using_join_buffer": "Block Nested Loop",
            "cost_info": {
              "read_cost": "736.57",
              "eval_cost": "14.87",
              "prefix_cost": "2414.76",
              "data_read_per_join": "1M"
            },
            "used_columns": [
              "id",
              "uid",
              "confirmed_at",
              "role",
              "level",
              "banned_until"
            ],
            "attached_condition": "((isnull(`barong_production`.`accounts`.`banned_until`) or (`barong_production`.`accounts`.`banned_until` < <cache>(now()))) and (`barong_production`.`accounts`.`role` = 'member') and (`barong_production`.`accounts`.`level` >= 1) and (`barong_production`.`accounts`.`confirmed_at` is not null) and ((`barong_production`.`accounts`.`uid` = `barong_production`.`refs`.`referrer_id_0`) or (`barong_production`.`accounts`.`uid` = `barong_production`.`refs`.`referrer_id_1`) or (`barong_production`.`accounts`.`uid` = `barong_production`.`refs`.`referrer_id_2`) or (`barong_production`.`accounts`.`uid` = `barong_production`.`refs`.`referrer_id_3`) or (`barong_production`.`accounts`.`uid` = `barong_production`.`refs`.`referrer_id_4`)))"
          }
        }
      ]
    }
  }
}
```
