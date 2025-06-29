# 🧹 Nashville Housing Data Cleaning – SQL Transformation & Standardization

This project demonstrates a comprehensive data cleaning process for a real estate dataset using SQL. The focus is on standardizing date formats, filling in missing address data, splitting composite columns, cleaning inconsistent categorical values, and identifying duplicates — all using native SQL operations in SQL Server.

---

## 📌 Project Objectives

- Convert inconsistent date formats into a standard `DATE` type column  
- Fill missing property addresses using self-joins based on shared parcel identifiers  
- Break down composite address fields (`PropertyAddress`, `OwnerAddress`) into individual parts  
- Standardize "SoldAsVacant" column values from `'Y'/'N'` to `'Yes'/'No'`  
- Identify and remove duplicate records based on key business logic  
- Drop redundant or unneeded columns after cleaning is complete  

---

## 📁 Data Source

**Table**: `PortfolioProject.dbo.NashvilleHousing`  
A raw housing sales dataset with attributes such as:

- `SaleDate`: Original transaction date (string format)  
- `PropertyAddress`, `OwnerAddress`: Full address strings  
- `SoldAsVacant`: Inconsistent boolean-like field (`Y/N`, etc.)  
- `ParcelID`, `UniqueID`: Keys for identifying and linking records  
- `TaxDistrict`, `LegalReference`: Legal and administrative info  

---

## 🔍 Key Features & SQL Queries

### 🗓 1. Date Standardization  
**Query Name**: `convert_sale_date_to_date_type`

```sql
ALTER TABLE NashvilleHousing
ADD SaleDateConverted DATE;

UPDATE NashvilleHousing
SET SaleDateConverted = CONVERT(DATE, SaleDate);
```

---

### 🏠 2. Address Imputation Using Self Join  
**Query Name**: `fill_missing_property_address`

```sql
UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleHousing a
JOIN NashvilleHousing b
  ON a.ParcelID = b.ParcelID AND a.[UniqueID ] <> b.[UniqueID ]
WHERE a.PropertyAddress IS NULL;
```

---

### 🧱 3. Splitting Composite Columns  
**Query Names**: `split_property_address`, `split_owner_address`

```sql
-- Split PropertyAddress
UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1),
    PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress));

-- Split OwnerAddress
UPDATE NashvilleHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3),
    OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2),
    OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1);
```

---

### ✅ 4. Standardizing Boolean-like Fields  
**Query Name**: `normalize_sold_as_vacant_values`

```sql
UPDATE NashvilleHousing
SET SoldAsVacant = CASE 
    WHEN SoldAsVacant = 'Y' THEN 'Yes'
    WHEN SoldAsVacant = 'N' THEN 'No'
    ELSE SoldAsVacant
END;
```

---

### 🧹 5. Identifying & Removing Duplicates  
**Query Name**: `remove_duplicate_rows`

```sql
WITH RowNumCTE AS (
  SELECT *, ROW_NUMBER() OVER (
    PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference
    ORDER BY UniqueID
  ) AS row_num
  FROM NashvilleHousing
)
SELECT *
FROM RowNumCTE
WHERE row_num > 1;
```

---

### 🧯 6. Dropping Redundant Columns  
**Query Name**: `drop_unused_columns`

```sql
ALTER TABLE NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate;
```

---

## 📊 Sample Insights Enabled by Cleaning

- Property transactions are now tied to clean, structured addresses  
- Date fields are properly cast, enabling time-based trend analysis  
- Categorical variables are normalized, supporting accurate filtering and grouping  
- Duplicates are detected and can be eliminated to improve data reliability

---

## 💻 Tech Stack

- **SQL Server** – Core platform used for data transformation  
- **SSMS (SQL Server Management Studio)** – Used to run and test queries  
- **Raw Dataset** – Pre-cleaned `NashvilleHousing` table in SQL Server  

---

## 🧠 Why This Project?

Data rarely comes clean. This project simulates real-world data wrangling challenges that analysts and BI developers face daily — from missing values and text parsing to field normalization and deduplication — all essential for preparing reliable datasets for analytics and reporting.
