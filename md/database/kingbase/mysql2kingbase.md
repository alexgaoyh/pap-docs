# [人大金仓]Mysql数据库迁移Kingbase数据库-自增序列

## 背景

&ensp;&ensp;在一些老版本的Kingbase数据库，使用 KingbaseDTS 进行迁移的时候，老版本会出现自增序列的问题，故编写此文进行解决。

## 思路

&ensp;&ensp;首先在Mysql中将自增序列进行删除，之后进行恢复。Mysql自增序列删除之后同步至Kingbase之后，需要在Kingbase中对相关表再次维护序列。

&ensp;&ensp;脚本1： 删除自增设置；
&ensp;&ensp;脚本2： 回退删除的自增设置；
&ensp;&ensp;脚本3： 在Kingbase数据库中对原先的自增列做序列维护；

## SQL脚本

```sql
    -- 1. Mysql环境下，取消自增设置的 SQL 语句（保留字段注释）
    SELECT
    CONCAT(
    '-- 取消自增设置并保留字段注释\n',
    'ALTER TABLE ', TABLE_SCHEMA, '.', TABLE_NAME,
    ' MODIFY ', COLUMN_NAME, ' ', COLUMN_TYPE,
    (CASE WHEN IS_NULLABLE = 'NO' THEN ' NOT NULL' ELSE '' END),
    (CASE WHEN COLUMN_COMMENT IS NOT NULL AND COLUMN_COMMENT != ''
    THEN CONCAT(' COMMENT ''', REPLACE(COLUMN_COMMENT, '''', ''''''), '''')
    ELSE '' END),
    ';'
    ) AS cancel_auto_increment_sql
    FROM information_schema.COLUMNS
    WHERE TABLE_SCHEMA = 'alexgaoyh' AND EXTRA LIKE '%auto_increment%';

    -- 2. Mysql环境下，还原自增设置的 SQL 语句（保留字段注释）
    SELECT
        CONCAT(
                '-- 查询当前列的最大值\n',
                'SET @max_value = (SELECT COALESCE(MAX(', COLUMN_NAME, '), 0) FROM ', TABLE_SCHEMA, '.', TABLE_NAME, ');\n',
                '-- 动态生成还原自增的 SQL 并执行\n',
                'SET @auto_increment_sql = CONCAT(\n',
                '    "ALTER TABLE ', TABLE_SCHEMA, '.', TABLE_NAME,
                ' MODIFY ', COLUMN_NAME, ' ', COLUMN_TYPE,
                (CASE WHEN IS_NULLABLE = 'NO' THEN ' NOT NULL AUTO_INCREMENT ' ELSE ' AUTO_INCREMENT ' END),
                (CASE WHEN COLUMN_COMMENT IS NOT NULL AND COLUMN_COMMENT != ''
        THEN CONCAT(' COMMENT ''', REPLACE(COLUMN_COMMENT, '''', ''''''), '''')
                      ELSE '' END),
                ';"\n',
                ');\n',
                'PREPARE stmt FROM @auto_increment_sql;\n',
                'EXECUTE stmt;\n',
                'DEALLOCATE PREPARE stmt;\n'
        ) AS restore_auto_increment_sql
    FROM information_schema.COLUMNS
    WHERE TABLE_SCHEMA = 'alexgaoyh' AND EXTRA LIKE '%auto_increment%';

    -- 3. Kingbase 环境中生成序列
    SELECT
        CONCAT(
            '-- 查询当前列的最大值\n',
            'DO $$\n',
            'DECLARE\n',
            '    max_value BIGINT;\n',
            '    sequence_name TEXT;\n',
            'BEGIN\n',
            '    -- 获取当前列的最大值\n',
            '    SELECT COALESCE(MAX(', COLUMN_NAME, '), 0) + 1 INTO max_value FROM ', TABLE_NAME, ';\n',
            '    -- 获取序列名\n',
            '    SELECT pg_get_serial_sequence(''', TABLE_NAME, ''', ''', COLUMN_NAME, ''') INTO sequence_name;\n',
            '    -- 检查序列是否绑定到字段\n',
            '    IF sequence_name IS NULL THEN\n',
            '        -- 序列未绑定，重新创建序列并绑定\n',
            '        RAISE NOTICE \'序列未绑定，正在创建新序列并绑定到字段: %.%\', ''', TABLE_NAME, ''', ''', COLUMN_NAME, ''';\n',
            '        sequence_name := \'', TABLE_NAME, '_', COLUMN_NAME, '_seq\';\n',
            '        DROP SEQUENCE IF EXISTS ', TABLE_NAME, '_', COLUMN_NAME, '_seq;\n',
            '        EXECUTE format(\n',
            '            \'CREATE SEQUENCE %I START WITH %s INCREMENT BY 1 NO MINVALUE NO MAXVALUE CACHE 1;\',\n',
            '            sequence_name, max_value\n',
            '        );\n',
            '        EXECUTE format(\n',
            '            \'ALTER TABLE %I ALTER COLUMN %I SET DEFAULT nextval(%L);\',\n',
            '             ''', TABLE_NAME, ''', ''', COLUMN_NAME, ''', sequence_name\n',
            '        );\n',
            '    END IF;\n',
            '    -- 设置序列值\n',
            '    PERFORM setval(sequence_name, max_value, true);\n',
            'END;\n',
            '$$;\n'
        ) AS restore_auto_increment_sql
        FROM information_schema.COLUMNS
        WHERE TABLE_SCHEMA = 'alexgaoyh' AND EXTRA LIKE '%auto_increment%';
```

## 参考

1. https://github.com/pkumod/gStore
2. http://pap-docs.pap.net.cn/
3. https://gitee.com/alexgaoyh








