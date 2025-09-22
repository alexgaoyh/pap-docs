```sql
-- 省市区 json 结构查询.
SELECT
	JSON_ARRAYAGG(
	CASE
			WHEN EXISTS ( SELECT 1 FROM t_data_area T2 WHERE T2.AREA_PARENT_ID = T1.AREA_ID ) THEN
			JSON_OBJECT(
				'AREA_NAME',
				T1.AREA_NAME,
				'AREA_ID',
				T1.AREA_ID,
				'children',
				(
				SELECT
					JSON_ARRAYAGG(
					CASE
							WHEN EXISTS ( SELECT 1 FROM t_data_area T3 WHERE T3.AREA_PARENT_ID = T2.AREA_ID ) THEN
							JSON_OBJECT(
								'AREA_NAME',
								T2.AREA_NAME,
								'AREA_ID',
								T2.AREA_ID,
								'children',
								(
								SELECT
									JSON_ARRAYAGG(
									CASE
											WHEN EXISTS ( SELECT 1 FROM t_data_area T4 WHERE T4.AREA_PARENT_ID = T3.AREA_ID ) THEN
											JSON_OBJECT(
												'AREA_NAME',
												T3.AREA_NAME,
												'AREA_ID',
												T3.AREA_ID,
												'children',
												(
												SELECT
													JSON_ARRAYAGG( JSON_OBJECT( 'AREA_NAME', T4.AREA_NAME, 'AREA_ID', T4.AREA_ID ) ) 
												FROM
													t_data_area T4 
												WHERE
													T4.AREA_PARENT_ID = T3.AREA_ID 
												ORDER BY
													T4.AREA_ID ASC 
												) 
											) ELSE JSON_OBJECT( 'AREA_NAME', T3.AREA_NAME, 'AREA_ID', T3.AREA_ID ) 
										END 
										) 
									FROM
										t_data_area T3 
									WHERE
										T3.AREA_PARENT_ID = T2.AREA_ID 
									ORDER BY
										T3.AREA_ID ASC 
									) 
								) ELSE JSON_OBJECT( 'AREA_NAME', T2.AREA_NAME, 'AREA_ID', T2.AREA_ID ) 
							END 
							) 
						FROM
							t_data_area T2 
						WHERE
							T2.AREA_PARENT_ID = T1.AREA_ID 
						ORDER BY
							T2.AREA_ID ASC 
						) 
					) ELSE JSON_OBJECT( 'AREA_NAME', T1.AREA_NAME, 'AREA_ID', T1.AREA_ID ) 
				END 
				) AS test 
			FROM
				t_data_area T1 
			WHERE
				T1.AREA_PARENT_ID IS NULL 
		ORDER BY
	T1.AREA_ID ASC;
```