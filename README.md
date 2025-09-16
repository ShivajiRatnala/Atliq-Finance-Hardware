 Project Atliq Hardware Analysis

 Tools:                                                                                                                                                                                   
        Excel                                                                                                                                                              
        Mysql                                                                                                                                                                      
			  powerpoint                                                                                                                                                                 
Excel Used to visualize the data.                                                                                                                                                  
Sql used to Quering and data cleaning operations                                                                                                                                   
Power Point use to make reports.                                                                                                                                                   

Way of importing data                                                                                                                                                                                                                                                                                                                                                 -->     Open Sql and click a sever option and click import data select the file location.                                                                                               

Some Queries Examples 

customer Analysis 
		 
		 	SELECT DC.CUSTOMER,
			SUM(CASE WHEN YEAR(FSM.DATE) = 2019 THEN FSM.SOLD_QUANTITY * FGP.GROSS_PRICE END) AS YEAR_2019,
			SUM(CASE WHEN YEAR(FSM.DATE) = 2020 THEN FSM.SOLD_QUANTITY * FGP.GROSS_PRICE END) AS YEAR_2020,
			SUM(CASE WHEN YEAR(FSM.DATE) = 2021 THEN FSM.SOLD_QUANTITY * FGP.GROSS_PRICE END) AS YEAR_2021,
			CONCAT(TRUNCATE(
        (SUM(CASE WHEN YEAR(FSM.DATE) = 2021 THEN FSM.SOLD_QUANTITY * FGP.GROSS_PRICE END) /
         SUM(CASE WHEN YEAR(FSM.DATE) = 2020 THEN FSM.SOLD_QUANTITY * FGP.GROSS_PRICE END)
        ) * 100, 1),"%"
    			) AS 'YOY2021-2020'
			FROM DIM_CUSTOMER AS DC 
			JOIN FACT_SALES_MONTHLY AS FSM ON DC.CUSTOMER_CODE = FSM.CUSTOMER_CODE
			JOIN FACT_GROSS_PRICE AS FGP ON FSM.PRODUCT_CODE = FGP.PRODUCT_CODE
			WHERE YEAR(FSM.DATE) in (2019,2020,2021)
			GROUP BY DC.CUSTOMER ORDER BY DC.CUSTOMER ASC;


 Top 3 Products in each division                                                                                                                                                                                                                                                                                                                        

				WITH CTE1 AS (
				SELECT DP.PRODUCT,DP.DIVISION,DP.PRODUCT_CODE,
				SUM(FSM.SOLD_QUANTITY) AS TOTAL_QUANTITY,
				DENSE_RANK() OVER (PARTITION BY DP.DIVISION ORDER BY SUM(FSM.SOLD_QUANTITY) DESC) AS RANKS
				FROM DIM_PRODUCT AS DP 
				JOIN FACT_SALES_MONTHLY AS FSM ON DP.PRODUCT_CODE = FSM.PRODUCT_CODE
				WHERE FSM.FISCAL_YEAR = 2021
				GROUP BY DP.PRODUCT,DP.DIVISION,DP.PRODUCT_CODE)
				
				SELECT * FROM CTE1 WHERE RANKS <=3;


Total Perentage Contribution in Evey channel

						SELECT DC.CHANNEL,ROUND(SUM(FGP.GROSS_PRICE * FSM.SOLD_QUANTITY) / 1000000, 2) AS GROSS_SALES_MLN, 
						CONCAT
						(
						ROUND
						( (SUM(FGP.GROSS_PRICE*FSM.SOLD_QUANTITY)*100)/
						SUM(SUM(FGP.GROSS_PRICE*FSM.SOLD_QUANTITY)) OVER(),2),'%'
						) AS 'PERCENT%'
						FROM DIM_CUSTOMER AS DC 
						JOIN 
						FACT_SALES_MONTHLY AS FSM ON DC.CUSTOMER_CODE = FSM.CUSTOMER_CODE
						JOIN
						FACT_GROSS_PRICE AS FGP ON FGP.PRODUCT_CODE = FSM.PRODUCT_CODE
						WHERE FSM.FISCAL_YEAR = 2021
						GROUP BY DC.CHANNEL ORDER BY  GROSS_SALES_MLN DESC;






