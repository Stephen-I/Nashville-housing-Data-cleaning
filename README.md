# Nashville-housing-Data-cleaning

This is the Code I used in SQL to cleam the data.

After looking over the data I was provided, I selected the coloumns I would need. 

Then proceeded to organise specific columns by first standardizing the date sale date fromat

```
USE Nashville_Housing

SELECT * 
FROM [dbo].[Nashville Housing]



/*Standardize Sale Date Format*/

SELECT SaleDate, CONVERT(Date,SaleDate) 
FROM [dbo].[Nashville Housing]

UPDATE [dbo].[Nashville Housing]
SET SaleDate = CONVERT(Date,SaleDate) 
```
Next, I had to Remove the null values from the Propertyaddress column

```
/*Populate Property address Data*/

SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM [dbo].[Nashville Housing] a
JOIN [dbo].[Nashville Housing] b
ON a.ParcelID = b.ParcelID
AND a.UniqueID <> b.UniqueID
WHERE a.PropertyAddress is Null

UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM [dbo].[Nashville Housing] a
JOIN [dbo].[Nashville Housing] b
ON a.ParcelID = b.ParcelID
AND a.UniqueID <> b.UniqueID

```
The next objective was to sepereate the Address column into three seperate columns as shown, to make it clearer to read

```

/* Seperate Address into three columns (Address, City, State)*/

SELECT PropertyAddress 
FROM [dbo].[Nashville Housing]

SELECT 
SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1) AS Address,
SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress)) AS City
FROM [dbo].[Nashville Housing ]

Alter TABLE [dbo].[Nashville Housing ]
Add PropertySplitAddress Nvarchar(255)

UPDATE [dbo].[Nashville Housing]
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1)

Alter TABLE [dbo].[Nashville Housing]
Add PropertySplitCity Nvarchar(255)

UPDATE [dbo].[Nashville Housing Data]
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress))



/* Seperate OwnerAddress into three columns (Address, City, State)*/

SELECT OwnerAddress 
FROM [dbo].[Nashville Housing]


SELECT 
ParseName(REPLACE(OwnerAddress, ',','.'), 3) AS Address,
ParseName(REPLACE(OwnerAddress, ',','.'), 2) AS City,
ParseName(REPLACE(OwnerAddress, ',','.'), 1) AS State
FROM [dbo].[Nashville Housing]

Alter TABLE [dbo].[Nashville Housing]
Add OwnerSplitAddress Nvarchar(255)

UPDATE [dbo].[Nashville Housing]
SET OwnerSplitAddress = ParseName(REPLACE(OwnerAddress, ',','.'), 3)

Alter TABLE [dbo].[Nashville Housing]
Add OwnerSplitCity Nvarchar(255)

UPDATE [dbo].[Nashville Housing]
SET OwnerSplitCity = ParseName(REPLACE(OwnerAddress, ',','.'), 2)

Alter TABLE [dbo].[Nashville Housing]
Add OwnerSplitState Nvarchar(255)

UPDATE [dbo].[Nashville Housing]
SET OwnerSplitState = ParseName(REPLACE(OwnerAddress, ',','.'), 1)



/*Change Y and N to Yes and no in 'sold as vacant'*/

SELECT DISTINCT(SoldAsVacant), count(SoldAsVacant)
FROM [dbo].[Nashville Housing]
Group by SoldAsVacant
Order By SoldAsVacant

ALTER TABLE [dbo].[Nashville Housing]
ALTER COLUMN SoldAsVacant Varchar;



SELECT *,
Case WHEN SoldAsVacant = '1' THEN 'YES'
WHEN SoldAsVacant = '0' THEN 'NO'
ELSE SoldAsVacant
END AS SoldAsVacantCoverted
From [dbo].[Nashville Housing]
```
