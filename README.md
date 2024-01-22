# Data Analytics Power BI Report

In this fictious scenario I have recently been approached by a medium-sized international retailer who is keen on elevating their business intelligence practices. With operations spanning across different regions, they've accumulated large amounts of sales from disparate sources over the years.

Recognizing the value of this data, they aim to transform it into actionable insights for better decision-making. My goal is to use Microsoft Power BI to design a comprehensive Quarterly report. This will involve extracting and transforming data from various origins, designing a robust data model rooted in a star-based schema, and then constructing a multi-page report.

The report will present a high-level business summary tailored for C-suite executives, and also give insights into their highest value customers segmented by sales region, provide a detailed analysis of top-performing products categorised by type against their sales targets, and a visually appealing map visual that spotlights the performance metrics of their retail outlets across different territories.

## Data Loading and Preparation

I connected to an Azure SQL database, a Microsfot Azure Storage Account, and web-hosted CSV files to import useful data for this dataset. I cleaned and organized the data by removing irrelevant columns, splitting date-time details, and ensuring data consistency. I also renamed columns to fit Power BI conventions.

I connected to the Azure SQL Database and imported the orders_powerbi table using the Import option in Power BI. 

The Orders table is the main fact table. It contains information about each order, including the order and shipping dates, the customer, store and product IDs for associating with dimension tables, and the amount of each product ordered. Each order in this table consists of an order of a single product type, so there is only one product code per order. 

I used the Power Query Editor and delete the column named [Card Number] to ensure data privacy.

I used the Split Column feature to separate the [Order Date] and [Shipping Date] columns into two distinct columns each: one for the date and another for the time

I filtered out and removed any rows where the [Order Date] column has missing or null values to maintain data integrity

I renamed the columns in my dataset to align with Power BI naming conventions (e.g. captilaisign first letters, using spaces instead of underscores), ensuring consistency and clarity in your report

I downloaded the Products.csv file  and then used Power BI's Get Data option to import the file into my project.

The Products table contains information about each product sold by the company, including the product code, name, category, cost price, sale price, and weight.

In the Data view, I used the Remove Duplicates function on the product_code column to ensure each product code is unique.

In Power Query Editor, I used the Column From Examples feature to generate two new columns from the weight column - one for the weight values and another for the units (e.g. kg, g, ml). 

For the newly created units column, replaced any blank entries with kg using the Replace Values feature.
I converted the data type of the values column to a decimal number
I replaced error values with the number 1

In the Data view, I created a new calculated column, such that if the unit in the units column is not kg, divide the corresponding value in the values column by 1000 to convert it to kilograms.

I returned to the Power Query Editor and delete any columns that are no longer needed.

I renamed the columns in your dataset to match Power BI naming conventions, ensuring a consistent and clear presentation in my report.

I used Power BI's Get Data option to connect to Azure Blob Storage and import the Stores table into my project. The Stores table contains information about each store, including the store code, store type, country, region, and address.

I downloaded the Customers.zip file - inside the zip file is a folder containing three CSV files, each with the same column format, one for each of the regions in which the company operates.

I used the Get Data option (using the Folder data connector) in Power BI to import the Customers folder into my project. I selected Combine and Transform to import the data; Power BI automatically appended the three files into one query.

Once the data are loaded into Power BI, I create a Full Name column by combining the [First Name] and [Last Name] columns using the DAX language's CONCATENATE function.

I delete any obviously unused columns (eg. index columns) and renamed the remaining columns to align with Power BI naming conventions.

## Creatign the Data Model

### generate the date table

I created a date table running from the start of the year containing the earliest date in the Orders['Order Date'] column to the end of the year containing the latest date in the Orders['Shipping Date'] column usign the DAX formula:

Dates = CALENDAR(
STARTOFYEAR(Orders[Order Date]), 
ENDOFYEAR(Orders[Shipping Date])
)

Day Of Week = WEEKDAY(Dates[Date],2)

Month = MONTH(Dates[Date])

Month Name = FORMAT(DATE(1, Dates[Month], 1), "MMMM")

Quarter = QUARTER(Dates[Date])

Year = YEAR(Dates[Date])

Start of Year = STARTOFYEAR(Dates[Date])

Start of Quarter = STARTOFQUARTER(Dates[Date])

Start of Month = STARTOFMONTH(Dates[Date])

Start of Week = Dates[Date] - WEEKDAY(Dates[Date],2) + 1

### Build the Star Schema data model

I created relationships between the tables to form a star schema. The relationships are as follows:

Orders[product_code] to Products[product_code]
Orders[Store Code] to Stores[store code]
Orders[User ID] to Customers[User UUID]
Orders[Order Date] to Date[date]
Orders[Shipping Date] to Date[date]

i ensured that the relationship between Orders[Order Date] and Date[date] is the active relationship, and that all relationships are one-to-many, with a single filter direction from the one side to the many side

add a screenshot image of my data model

### Create a Measures table

Creating a separate table for measures is a best practice that will help keep the data model organized and easy to navigate. 

I created a new table in the data Model View with Power Query Editor. It is generally better to use the latter approach, as it makes the measures table visible in the Query Editor, which is useful for debugging and troubleshooting.

### Create key measures

I created some of the key measures that will be used in the report. These give me a starting point for building the analysis:

I created a measure called Total Orders that counts the number of orders in the Orders table:

Total Orders = COUNTROWS(Orders)

I created a measure called Total Revenue that multiplies the Orders[Product Quantity] column by the Products[Sale Price] column for each row, and then sums the result:

Total Revenue = SUMX(Orders, Orders[Product Quantity] * RELATED(Products[Sale Price]))

I created a measure called Total Profit which performs the following calculation: For each row, subtract the Products[Cost_Price] from the Products[Sale_Price], and then multiply the result by the Orders[Product Quantity]. then, it sums the result for all rows:

Total Profit = SUMX(Orders, (RELATED(Products[Sale Price]) - RELATED(Products[Cost Price])) * Orders[Product Quantity])

I created a measure called Total Customers that counts the number of unique customers in the Orders table:

Total Customers = DISTINCTCOUNT(Orders[User ID])

I created a measure called Total Quantity that counts the number of items sold in the Orders table:

Total Quantity = SUM(Orders[Product Quantity]) 

I created a measure called Profit YTD that calculates the total profit for the current year:

Profit YTD = TOTALYTD(SUMX(Orders, (RELATED(Products[Sale Price]) - RELATED(Products[Cost Price])) * Orders[Product Quantity]), Orders[Order Date])

I created a measure called Revenue YTD that calculates the total revenue for the current year:

Revenue YTD = TOTALYTD(SUMX(Orders, RELATED(Products[Sale Price]) * Orders[Product Quantity]), Orders[Order Date])



the DAX formulas I used to create key measures and calculated columns. 
