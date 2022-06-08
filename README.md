# Data-Cleanning-in-SQL
Data Cleanning in SQL with NashVilleHousing
/* 
Cleaning data in SQL Queries
*/


select *
from NashvilleHousing
------------------------------------------------------------------------------------------------------------------------
-- Standardize data format

select SaleDateConverted, Convert(Date, SaleDate)
from NashvilleHousing

update NashvilleHousing
Set SaleDate = Convert(Date, SaleDate) -- Didn't work

alter table NashvilleHousing
add SaleDateConverted date

Update NashvilleHousing
set SaleDateConverted = Convert(Date, SaleDate) -- It did work, at the end delet SaleDate

------------------------------------------------------------------------------------------------------------------------
-- Populate propoty adress data

select [UniqueID ], ParcelID, PropertyAddress
from NashvilleHousing
where PropertyAddress is null
Order by ParcelID

select  T1.ParcelID, T1.PropertyAddress, T2.ParcelID, T2.PropertyAddress, ISNULL(T1.PropertyAddress, T2.PropertyAddress)
from NashvilleHousing AS T1
Join NashvilleHousing AS T2
ON T1.ParcelID = T2.ParcelID
And T1.[UniqueID ] <> T2.[UniqueID ]
where T1.PropertyAddress is null

Update T1
SET PropertyAddress = ISNULL(T1.PropertyAddress, T2.PropertyAddress)
from NashvilleHousing AS T1
Join NashvilleHousing AS T2
ON T1.ParcelID = T2.ParcelID
And T1.[UniqueID ] <> T2.[UniqueID ]
where T1.PropertyAddress is null
--------------------------------------------------------------------------------------------------------------------
-- Brinking out Address into into individual colomns (Address, City, State)

select  PropertyAddress
from NashvilleHousing

select  PropertyAddress,
	SUBSTRING(PropertyAddress,1, CHARINDEX(',',PropertyAddress)-1) As Address,
	SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress)+1, LEN(PropertyAddress)) As Address
from NashvilleHousing

Alter table NashvilleHousing
Add PropertySplitAdress nvarchar(255)

Update NashvilleHousing
Set PropertySplitAdress = SUBSTRING(PropertyAddress,1, CHARINDEX(',',PropertyAddress)-1)

alter table NashvilleHousing
add PropertySplitCity nvarchar(50)

update NashvilleHousing
set PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress)+1, LEN(PropertyAddress))

select OwnerAddress,
	PARSENAME(Replace(OwnerAddress,',','.'),3),
	PARSENAME(Replace(OwnerAddress,',','.'),2),
	PARSENAME(Replace(OwnerAddress,',','.'),1)
from NashvilleHousing

Alter table NashvilleHousing
Add OwnerSplitAddress nvarchar(255)

Alter table NashvilleHousing
Add OwnerSplitCity nvarchar(255)

Alter table NashvilleHousing
add OwnerSplitState nvarchar(255)

update NashvilleHousing
set OwnerSplitAddress = PARSENAME(Replace(OwnerAddress,',','.'),3)

update NashvilleHousing
set OwnerSplitCity = PARSENAME(Replace(OwnerAddress,',','.'),2)

update NashvilleHousing
set OwnerSplitState = PARSENAME(Replace(OwnerAddress,',','.'),1)

select *
from NashvilleHousing
----------------------------------------------------------------------------------------------------
-- Change Y and N to Yes and No in 'SoldAsVacant' field

Select Distinct(SoldAsVacant), count(SoldAsVacant)
from NashvilleHousing
group by SoldAsVacant
order by 2

Select 
	SoldAsVacant,
	Case 
		When SoldAsVacant = 'Y' then 'Yes'
		When SoldAsVacant = 'N' then 'No'
		Else SoldAsVacant
	End
from NashvilleHousing

Update NashvilleHousing
set SoldAsVacant = Case 
		When SoldAsVacant = 'Y' then 'Yes'
		When SoldAsVacant = 'N' then 'No'
		Else SoldAsVacant
	End
--Check with the query before 
------------------------------------------------------------------------------------------------------------------------
-- Remove Duplicates

Select *,
	Row_Number() Over(
		PARTITION BY ParcelID,
			PropertyAddress,
			SaleDate,
			LegalReference
			Order by UniqueID) AS RowNumm
From NashvilleHousing


With NumCTE AS (
Select *,
	Row_Number() Over(
		PARTITION BY ParcelID,
			PropertyAddress,
			SaleDate,
			LegalReference
			Order by UniqueID) AS RowNumm
From NashvilleHousing
)

--Select * then Delete then Select* to check
select *
From NumCTE
where RowNumm > 1

-----------------------------------------------------------------
-- Delete Innecessary colomns

Alter Table NashvilleHousing
Drop column TaxDistrict

Select * 
From NashvilleHousing

------------------------------------------------------------------------------------------------------------------------
