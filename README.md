# Data-Cleaning-SQL
****** Script for SelectTopNRows command from SSMS  ******/
SELECT TOP (1000) [UniqueID ]
      ,[ParcelID]
      ,[LandUse]
      ,[PropertyAddress]
      ,[SaleDate]
      ,[SalePrice]
      ,[LegalReference]
      ,[SoldAsVacant]
      ,[OwnerName]
      ,[OwnerAddress]
      ,[Acreage]
      ,[TaxDistrict]
      ,[LandValue]
      ,[BuildingValue]
      ,[TotalValue]
      ,[YearBuilt]
      ,[Bedrooms]
      ,[FullBath]
      ,[HalfBath]
  FROM [project].[dbo].[Housing]


  --converting date from month day,year(ef:april 21,2000) to YYYY-MM-DD format
  select CONVERT(Date,SaleDate) as newdate from Housing;

  Alter Table Housing  add newdate date;

  update Housing
  SET  newdate=CONVERT(Date,SaleDate)  

  select [UniqueID ],ParcelID,PropertyAddress from Housing
   order by ParcelID;

   --replacing null propertyaddress by the same propertyaddress by using self join which has similar parcelID's
   select a.UniqueID,a.ParcelID,a.PropertyAddress,b.ParcelID,b.PropertyAddress from Housing a 
   JOIN Housing b on a.ParcelID=b.ParcelID
   order by a.ParcelID;

   select a.UniqueID,a.ParcelID,a.PropertyAddress,b.ParcelID,b.PropertyAddress,ISNULL(a.PropertyAddress,b.PropertyAddress) from Housing a 
   JOIN Housing b on a.ParcelID=b.ParcelID
   AND a.[UniqueID ] <> b.[UniqueID ]
  order by a.ParcelID;

  --updating propertyaddress
  update a
  set propertyaddress=ISNULL(a.PropertyAddress,b.PropertyAddress)
  from Housing a 
  JOIN Housing b on a.ParcelID=b.ParcelID
   AND a.[UniqueID ] <> b.[UniqueID ]
  order by a.ParcelID;

  --no propertyaddress is blank now
  select [UniqueID ],ParcelID,PropertyAddress from Housing
  where PropertyAddress is null;

  --replacing Y as yes and N as No in soldasvacant column
  select distinct soldasvacant from Housing;
   select soldasvacant,
   case 
   when soldasvacant='Y' then 'Yes'
   when soldasvacant='N' then 'No' 
   else soldasvacant end
   from Housing;

   update Housing set 
   SoldAsVacant= 
   case 
   when soldasvacant='Y' then 'Yes'
   when soldasvacant='N' then 'No' 
   else soldasvacant end
   from Housing;

   --Split the owneraddress into seperate lines as line1,line2,line3 and add them in seperate columns
   select owneraddress from housing;
   select distinct(parsename(replace(OwnerAddress,',','.'),1)),
   parsename(replace(OwnerAddress,',','.'),2),
   parsename(replace(OwnerAddress,',','.'),3)from housing;

   alter table housing add line1_owneraddress varchar(100)

   update housing set 
   line1_owneraddress=parsename(replace(OwnerAddress,',','.'),3)from housing;

   alter table housing add line2_owneraddress varchar(100)

   update housing set 
   line2_owneraddress=parsename(replace(OwnerAddress,',','.'),2)from housing;

   alter table housing add line3_owneraddress varchar(25)

   update housing set 
   line3_owneraddress=parsename(replace(OwnerAddress,',','.'),1)from housing;
--To find out duplicates
--using CTE giving row number of each row by grouping them and delete the row number which occured morethan once.
   with rownum as(
   select * ,row_number() over(partition by ParcelID,propertyaddress,saledate order by uniqueid) row_num
   from Housing
   )
   select * from rownum
   where row_num >1;


    with rownum as(
   select * ,row_number() over(partition by ParcelID,propertyaddress,saledate order by uniqueid) row_num
   from Housing
   )
   Delete from rownum
   where row_num >1;


  --Drop the unused columns
   Alter table Housing 
   DROP column Taxdistrict,landuse;
