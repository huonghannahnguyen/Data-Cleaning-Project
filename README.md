# Data-Cleaning-Project

PostgreSQL - Nashville Housing Data Cleaning

Nashville Housing Dataset link https://www.kaggle.com/datasets/bvanntruong/housing-sql-project

-- Create table 'nashvillehousing' and import data into postgreSQL

	CREATE TABLE nashvillehousing (uniqueID integer, parcelID integer, land_use text, property_address varchar, sale_date date, sale_price integer, legal_ref char, sold_as_vacant text, owner_name varchar, owner_address varchar, acreage decimal, tax_district text, land_value integer, building_value integer, total_value integer, year_build integer, bedroom integer, fullbath integer, halfbath integer);

	ALTER TABLE nashvillehousing
 	ALTER COLUMN sale_date TYPE varchar,
 	ALTER COLUMN legal_ref TYPE varchar,
 	ALTER COLUMN parcelID TYPE varchar,
 	ALTER COLUMN sale_price TYPE money;

-- Import Nashville Housing CSV file into postgreSQL, and overview the table

	SELECT * FROM nashvillehousing;

-- CLEANING DATASET

-- Standardize date format

	SELECT CAST(sale_date AS date)
 	FROM nashvillehousing;

	UPDATE nashvillehousing
 	SET sale_date = CAST(sale_date AS date);

-- Match null property address values with correct property addresses

	SELECT a.parcelid, a.property_address, 
 	       b.parcelid, b.property_address, COALESCE(a.property_address, b.property_address)
 	FROM nashvillehousing AS a
  	JOIN nashvillehousing AS b
   	     ON a.parcelid = b.parcelid
	     AND a.uniqueid != b.uniqueid
	WHERE a.property_address IS NULL

-- Update null property address

	UPDATE nashvillehousing a
 	SET property_address = b.property_address
  	FROM nashvillehousing b
   	WHERE a.uniqueID != b.uniqueID
    	      AND a.parcelid = b.parcelid
     	      AND a.property_address IS NULL;

-- Break property address into individual column (Address, City)

	ALTER TABLE nashvillehousing
 	ADD COLUMN property_numberstreet text,
  	ADD COLUMN property_city text;

	UPDATE nashvillehousing 
 	SET property_numberstreet = SPLIT_PART(property_address,',',1),     
   	    property_city = SPLIT_PART(property_address,',',2);

-- Break owner address into individual column (Address, city, state)

	ALTER TABLE nashvillehousing
 	ADD COLUMN owner_numberstreet text,
	ADD COLUMN owner_city text,
	ADD COLUMN owner_state text;

	UPDATE nashvillehousing
 	SET owner_numberstreet = SPLIT_PART(owner_address,',',1),
  	    owner_city = SPLIT_PART(owner_address,',',2),
   	    owner_state = SPLIT_PART(owner_address,',',3);

-- Change Y and N to Yes and No in sold_as_vacant field

	UPDATE nashvillehousing
 	SET sold_as_vacant = 
  	    CASE 
	    WHEN sold_as_vacant = 'N' THEN 'No'
	    WHEN sold_as_vacant = 'Y' THEN 'Yes'
	    ELSE sold_as_vacant 
	    END 

-- Remove unused columns

	ALTER TABLE nashvillehousing
 	DROP COLUMN property_address, 
	DROP COLUMN owner_address;

-- Rename columns

	ALTER TABLE nashvillehousing
	RENAME COLUMN property_numberstreet TO property_address;

	ALTER TABLE nashvillehousing
	RENAME COLUMN owner_numberstreet TO owner_address;

-- Nashville Housing table overview

	SELECT uniqueid, parcelid, land_use, property_address, property_city, sale_date, sale_price,
	       legal_ref, sold_as_vacant, owner_name, owner_address, owner_city, owner_state, acreage,
	       tax_district, land_value, building_value, total_value, year_build, bedroom, fullbath, halfbath
	FROM nashvillehousing;

