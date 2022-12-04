ssh cmomdji@144.24.14.145

# Load data into Oracle
wget https://www.dropbox.com/s/3xt3up4fve78me8/dolthub_us-housing-prices_main_sales_REDUCED2.csv?dl=0

# rename file
mv dolthub_us-housing-prices_main_sales_REDUCED2.csv?dl=0 us-housing-prices.csv 

# Make sure csv file is in Oracle server
ls -al

# Upload the file to hdfs
hdfs dfs -mkdir HousingData
hdfs dfs -ls
hdfs dfs -put us-housing-prices.csv HousingData/
hdfs dfs -ls HousingData/

# NOTE: Make sure to delete file in Oracle server after uploading it to hdfs
rm -rf us-housing-prices.csv

# Open separate terminal to use Hive
beeline 

use cmomdji;

# Create the table of all the housing data in the file
DROP TABLE IF EXISTS housing_data;
CREATE EXTERNAL TABLE housing_data(state STRING, property_zip5 STRING, property_street_address STRING, property_city STRING, property_county STRING, 
property_id STRING, sale_datetime STRING, property_type STRING, sale_price double, seller_1_name STRING, 
buyer_1_name STRING, building_num_units double, building_year_built double, source_url STRING, book STRING, page STRING, 
transfer_deed_type STRING, property_township STRING, property_lat STRING, property_lon STRING, sale_id STRING, deed_date STRING, 
building_num_stories double, building_num_beds double, building_num_baths double, building_area_sqft STRING, building_assessed_value double,
building_assessed_date STRING, land_area_acres STRING, land_area_sqft STRING, land_assessed_value STRING, seller_2_name STRING, 
buyer_2_name STRING, land_assessed_date STRING, seller_1_state STRING, seller_2_state STRING, buyer_1_state STRING, buyer_2_state STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE LOCATION '/user/cmomdji/HousingData/' 
TBLPROPERTIES("skip.header.line.count"="1");

# See if table was created
show tables;

# Make a test query
select state, property_street_address, property_city, property_county, sale_price from housing_data where state='CA' and sale_price > 10000 limit 10;

# create new table that filters our data from only the state of CA AND from 2019 or later
DROP TABLE IF EXISTS california_housing_records;
CREATE TABLE california_housing_records 
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE LOCATION '/user/cmomdji/HousingData/california_housing_records/' 
AS 
SELECT state, property_street_address, property_city, property_county, 
sale_price, property_type, sale_datetime
FROM housing_data 
WHERE state='CA' 
AND sale_datetime >= '2019-01-01 00:00:00' 
AND sale_price > 100000 
AND (property_type LIKE '%RESIDENCE%' OR property_type LIKE '%CONDO%');

# check to see if the file was saved to HDFS
hdfs dfs -ls HousingData/california_housing_records

# download the file from HDFS to Oracle
hdfs dfs -get /user/cmomdji/HousingData/california_housing_records/000000_0 california_housing_records.csv

# verify that the file is there
ls -al

# download the file to your computer (from a new terminal)
scp cmomdji@144.24.14.145:/home/cmomdji/california_housing_records.csv .

# the rest of the tutorial involves using Excel and PowerBI


