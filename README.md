# Data Analytics Power BI Report

I was recently approached by a medium-sized international retailer keen on elevating its business intelligence practices. With operations spanning different regions, they've accumulated large amounts of sales from disparate sources over the years.

Recognising the value of this data, they aim to transform it into actionable insights for better decision-making. I aim to use Microsoft Power BI to design a comprehensive Quarterly report. This will involve extracting and transforming data from various origins, designing a robust data model rooted in a star-based schema, and constructing a multi-page report.

The report will present a high-level business summary tailored for C-suite executives, give insights into their highest value customers segmented by sales region, provide a detailed analysis of top-performing products categorised by type against their sales targets, and a visually appealing map visual that spotlights the performance metrics of their retail outlets across different territories.

## Data Loading and Preparation

I connected to an Azure SQL database, a Microsoft Azure Storage Account, and web-hosted CSV files to import useful data for this dataset. I cleaned and organised the data by removing irrelevant columns, splitting date-time details, and ensuring data consistency. I also renamed columns to fit Power BI conventions.

I connected to the Azure SQL Database and imported the orders_powerbi table using the Import option in Power BI. 

The Orders table is the main fact table. It contains information about each order, including the order and shipping dates, the customer, store and product IDs for associating with dimension tables, and the amount of each product ordered. Each order in this table consists of an order of a single product type, so there is only one product code per order. 

I used the Power Query Editor and deleted the column named [Card Number] to ensure data privacy.

I used the Split Column feature to separate the [Order Date] and [Shipping Date] columns into two distinct columns each: one for the date and another for the time

I filtered out and removed any rows where the [Order Date] column has missing or null values to maintain data integrity

I renamed the columns in my dataset to align with Power BI naming conventions (e.g. capitalising first letters using spaces instead of underscores), ensuring consistency and clarity in my report.

I downloaded the Products.csv file  and then used Power BI's Get Data option to import the file into my project.

The Products table contains information about each product the company sells, including the product code, name, category, cost price, sale price, and weight.

In the Data view, I used the Remove Duplicates function on the product_code column to ensure each product code is unique.

In Power Query Editor, I used the Column From Examples feature to generate two new columns from the weight column - one for the weight values and another for the units (e.g. kg, g, ml). 

For the newly created units column, I replaced any blank entries with kg using the Replace Values feature.
I converted the data type of the values column to a decimal number
I replaced error values with the number 1

In the Data view, I created a new calculated column, such that if the unit in the units column is not kg, divide the corresponding value in the values column by 1000 to convert it to kilograms.

I returned to the Power Query Editor and deleted any columns that were no longer needed.

I renamed the columns in your dataset to match Power BI naming conventions, ensuring a consistent and clear presentation in my report.

I used Power BI's Get Data option to connect to Azure Blob Storage and import the Stores table into my project. The Stores table contains information about each store, including the store code, store type, country, region, and address.

I downloaded the Customers.zip file - inside the zip file is a folder containing three CSV files, each with the same column format, one for each of the regions in which the company operates.

I used the Get Data option (using the Folder data connector) in Power BI to import the Customers folder into my project. I selected Combine and Transform to import the data; Power BI automatically appended the three files into one query.

Once the data are loaded into Power BI, I create a Full Name column by combining the [First Name] and [Last Name] columns using the DAX language's CONCATENATE function.

I deleted any unused columns (e.g. index columns) and renamed the remaining columns to align with Power BI naming conventions.

## Creating the Data Model

### Generating the date table

I created a date table running from the start of the year containing the earliest date in the Orders['Order Date'] column to the end of the year containing the latest date in the Orders['Shipping Date'] column using the DAX formula:

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

### Building the Star Schema data model

I created relationships between the tables to form a star schema. The relationships are as follows:

Orders[product_code] to Products[product_code]
Orders[Store Code] to Stores[store code]
Orders[User ID] to Customers[User UUID]
Orders[Order Date] to Date[date]
Orders[Shipping Date] to Date[date]

I ensured that the relationship between Orders[Order Date] and Date[date] is the active relationship and that all relationships are one-to-many, with a single filter direction from the one side to the many side

### Creating a Measures table

Creating a separate table for measures is a best practice that will help keep the data model organized and easy to navigate. 

I created a new table in the data Model View with Power Query Editor. It is generally better to use the latter approach, as it makes the measures table visible in the Query Editor, which is useful for debugging and troubleshooting.

### Create key measures

I created some of the key measures used in the report. These give me a starting point for building the analysis:

I created a measure called Total Orders that counts the number of orders in the Orders table:

Total Orders = COUNTROWS(Orders)

I created a measure called Total Revenue that multiplies the Orders[Product Quantity] column by the Products[Sale Price] column for each row and then sums the result:

Total Revenue = SUMX(Orders, Orders[Product Quantity] * RELATED(Products[Sale Price]))

I created a measure called Total Profit, which performs the following calculation: For each row, subtract the Products[Cost_Price] from the Products[Sale_Price], and then multiply the result by the Orders[Product Quantity]. Then, it sums the result for all rows:

Total Profit = SUMX(Orders, (RELATED(Products[Sale Price]) - RELATED(Products[Cost Price])) * Orders[Product Quantity])

I created a measure called Total Customers that counts the number of unique customers in the Orders table:

Total Customers = DISTINCTCOUNT(Orders[User ID])

I created a measure called Total Quantity that counts the number of items sold in the Orders table:

Total Quantity = SUM(Orders[Product Quantity]) 

I created a measure called Profit YTD that calculates the total profit for the current year:

Profit YTD = TOTALYTD(SUMX(Orders, (RELATED(Products[Sale Price]) - RELATED(Products[Cost Price])) * Orders[Product Quantity]), Orders[Order Date])

I created a measure called Revenue YTD that calculates the total revenue for the current year:

Revenue YTD = TOTALYTD(SUMX(Orders, RELATED(Products[Sale Price]) * Orders[Product Quantity]), Orders[Order Date])


### Create Date and Georgraphy hierarchies

Hierarchies allow me to drill down into the data and perform granular analysis within the report. I created two hierarchies: one for dates to facilitate drill-down in my line charts and one for geography to allow me to filter the data by region, country and province/state.

I created a date hierarchy using the following levels:

Start of Year

Start of Quarter

Start of Month

Start of Week

Date

I created a new calculated column in the Stores table called Country that creates a full country name for each row, based on the Stores[Country Code] column, according to the following scheme:
GB: United Kingdom
US: United States
DE: Germany

Here is the DAX language formula:

Country = SWITCH([Country Code], 
"GB", "United Kingdom",
"US", "United States",
"DE", "Germany")

In addition to the country hierarchy, a full geography column  makes mapping more accurate. I created a new calculated column in the Stores table called Geography that creates a full geography name for each row, based on the Stores[Country Region] and Stores[Country] columns, separated by a comma and a space:

Geography = CONCATENATE(CONCATENATE(Stores[Country Region], ", "), Stores[Country])

I ensured that the following columns have the correct data category assigned, as follows:

World Region: Continent

Country: Country

Country Region: State or Province

I created a Geography hierarchy using the following levels:

World Region

Country

Country Region

![](Data_Model_Screenshot.png)

## Building the Customer Detail Page

I created a report page focussed on customer-level analysis.

I created headline card visuals: I created two rectangles as the backgrounds for the card visuals, added a card visual for the [Total Customers] measure I created earlier, and renamed the field Unique Customers.

I created a new measure in my Measures Table called [Revenue per Customer], which is the [Total Revenue] divided by the [Total Customers]:

Revenue per Customer = DIVIDE([Total Revenue], [Total Customers])

I added a card visual for the [Revenue per Customer] measure.

I created summary charts: I added a Donut Chart visual showing the total customers for each country, using the Users[Country] column to filter the [Total Customers] measure:

![](donut_chart_filters_screenshot.png)

I added a Column Chart visual showing the number of customers who purchased each product category, using the Products[Category] column to filter the [Total Customers] measure:

![](column_chart_setup_screenshot.png)

I added a Line Chart visual to the top of the page. It shows [Total Customers] on the Y axis and uses the created Date Hierarchy for the X axis. Users are allowed to drill down to the month level but not to weeks or individual dates.

![](line_chart_setup_screenshot.png)

I added a trend line and a forecast for the next 10 periods with a 95% confidence interval:

![](line_chart_forecast_setup_screenshot.png)

I created a new table that displays the top 20 customers filtered by revenue. The table shows each customer's full name, revenue, and number of orders.

![](top_20_customers_filters_screenshot.png)

I added conditional formatting to the revenue column to display data bars for the revenue values.

![](top_20_customers_filters_and_conditional_formatting_screenshot.png)

I created  a set of three card visuals that provide insights into the top customer by revenue. They display the top customer's name, the number of orders made by the customer, and the total revenue generated by the customer.

I added a date slicer to allow users to filter the page by year using the "between" slicer style.

![](Customer_Detail_Page_Screenshot.png)

## Creating an Executive Summary Page

I created a page to give an overview of the company's performance so that C-suite executives can quickly get insights and check outcomes against key targets. 

I created card visuals for my Total Revenue, Total Orders and Total Profit measures. I used the Format > Callout Value pane to ensure no more than 2 decimal places in the case of the revenue and profit cards and only 1 decimal place in the case of the Total Orders measure.

I added a revenue trending line chart by copying the line graph from my Customer Detail page and changing the Y-axis to Total Revenue.

I added a pair of donut charts, showing Total Revenue broken down by Store[Country] and Store[Store Type] respectively.

I added  a bar chart showing the number of orders by product category. I copied the Total Customers by Product Category donut chart from the Customer Detail page.

I went to the visual's "Build a visual" pane to change the visual type to a Clustered bar chart.

I changed  the X-axis field from Total Customers to Total Orders.

With the Format pane open, I clicked on one of the bars to bring up the Colors tab and select a colour.

###  KPI Visuals

I created KPIs for Quarterly Revenue, Orders and Profit. 

I created a set of new measures for the quarterly targets:

Previous Quarter Profit

Previous Quarter Revenue

Previous Quarter Orders

Target Profit, Revenue, and Orders, equal to 5% growth in each measure compared to the previous quarter, e.g.

Target Profit = 1.05 * [Previous Quarter Profit]

I was then able to create a  KPI visual for the revenue:

The Value field is Total Revenue.

The Trend Axis is Start of Quarter

The Target is Target Revenue

In the Format pane, I set the Trend Axis to On, expand the associated tab, and set the values as follows:

Direction: High is Good
Bad Colour: red
Transparency: 15%

I formatted the Callout Value to only show 1 decimal place.

I duplicated the card twice and set the appropriate values for the Profit and Orders cards.

![](executive_summary_page_screenshot.png)

## Product Detail Page

### Gauge Visuals

I added a set of three gauges, showing the current-quarter performance of Orders, Revenue and Profit against a quarterly target. The CEO told me they targeted 10% quarter-on-quarter growth in all three metrics.

In my measures table, I defined DAX measures for the three metrics and quarterly targets for each metric. I used the TOTALQTD and CALCULATE functions to calculate the cumulative values and the target values based on the 10% growth rate.

I created three gauge visuals and assigned the measures I had created. In each case, the maximum value of the gauge was set to the target, so the gauge showed as full when the target was met.

I applied conditional formatting to the callout value (the number in the middle of the gauge), so it showed as red if the target was not yet met and black otherwise.

I arranged my gauges to be evenly spaced along the report's top. I left another similarly-sized space for the card visuals that displayed which slicer states were currently selected. I used the alignment and distribution tools to adjust the position and size of my visuals.

### Filter State Cards

To the left of the gauges, I put some placeholder shapes for the cards, which will show the filter state. Using a colour in keeping with my theme, I added two rectangle shapes. I will sort out the values for these once I have added the slicer panel.

### Area Chart of Revenue by Product Category

I wanted to add an area chart showing how the different product categories were performing in terms of revenue over time.

I added a new area chart and applied the following fields:

X axis is Dates[Start of Quarter]
Y axis is Total Revenue
Legend is Products[Category]

### Top 10 Product Table

I created a table of the Top 10 Products ranked by Total Orders. The table has the following fields:


Product Description
Total Revenue
Total Customers
Total Orders
Profit per Order

I created a new measure of Profit per Order = DIVIDE([Total Profit], [Total Orders])

### Scatter Graph

The products team wanted to know which items to suggest to the marketing team for a promotional campaign. They wanted a visual that allowed them to quickly see which product ranges were both top-selling items and also profitable. A scatter graph is ideal for this job.

I created a new calculated column called [Profit per Item] in my Products table, using a DAX formula to work out the profit per item:

Profit per Item = [Sale Price] -  [Cost Price]

I added a new Scatter Chart to the page and configured it as follows:

Values are Products[Description]
X-Axis is Products[Profit per Item]
Y-Axis is Products[Total Quantity]
Legend is Products[Category]

![](product_detail_page_screenshot.png)

### Slicer Toolbar

Slicers were a handy way for me to control how the data on a page were filtered, but adding multiple slicers cluttered up the layout of my report page.

A professional-looking solution to this issue was to use Power BI's bookmarks feature to create a pop-out toolbar, which could be accessed from the navigation bar on the left-hand side of my report.

I downloaded a collection of custom icons. I used these for all of my navigation bar icons.

I added a new blank button to the top of my navigation bar, set the icon type to Custom in the Format pane, and chose Filter_icon.png as the icon image. I also set the tooltip text to Open Slicer Panel.


I added two new slicers. One was set to Products[Category], and the other to Stores[Country]. I changed the titles to Product Category and Country, respectively.

They are in Vertical List slicer style.
It is possible to select multiple items in the Product Category slicer, but only one option at a time is selectable in the Country slicer.
I configured the Country slicer so that there is a Select All option.
I ensured the formatting was neat and fit with my  report style.
I grouped the slicers with my slicer toolbar shape using the Selection pane.

I needed to add a Back button to hide the slicer toolbar when it is not in use.

I added a new button and selected the Back button type. I positioned it somewhere sensible, in the top-right corner of the toolbar.
I dragged the back button into the group with the slicers and toolbar shape in the Selection pane.

I opened the Bookmarks pane and added two new bookmarks: I created one bookmark with the toolbar group hidden in the Selection pane and one with it visible. I named them Slicer Bar Closed and Slicer Bar Open. I right-clicked each bookmark in turn and ensured that Data was unchecked. This prevents the bookmarks from altering the slicer state when I open and close the toolbar.

The final step was to assign the actions on my buttons to the bookmarks. I opened the Format pane and turned the Action setting on for both my Filter and Back buttons. For each button, I set the Type to Bookmark and selected the appropriate bookmark. 

![](slicer_bar_closed_screenshot.png)

![](slicer_bar_open_screenshot.png)

Finally, I tested my buttons and slicers (I remembered I needed to Ctrl-Click to use buttons while designing the report in Power BI Desktop).

## Stores Map Page

The regional managers requested a report page that allows them to easily check on the stores under their control, allowing them to see which stores they are responsible for are most profitable and which are on track to reach their quarterly profit and revenue targets. 

The best way to do this is using a map visual.

### Map Visual

I added a new map visual on the Stores Map page. It takes up most of the page, just leaving a narrow band at the top for a slicer. I set the style to my satisfaction in the Format pane and ensured Show Labels was set to On.

I set the controls of my map as follows:
Auto-Zoom: On
Zoom buttons: Off
Lasso button: Off

I assigned my Geography hierarchy to the Location field and ProfitYTD to the Bubble size field.

### Country Slicer

I added a slicer above the map, set the slicer field to Stores[Country], and in the Format section, I set the slicer style as Tile and the Selection settings to Multi-select with Ctrl/Cmd and Show "Select All" as an option in the slicer.

![](stores_map_screenshot.png)

### Stores Drillthrough Page

I wanted to make it easy for the region managers to check on the progress of a given store, so I created a drillthrough page summarising each store's performance. This included the following visuals:

A table showing the top 5 products, with columns: Description, Profit YTD, Total Orders, Total Revenue
A column chart showing Total Orders by product category for the store
Gauges for Profit YTD against a profit target of 20% year-on-year growth vs. the same period in the previous year. The target used the Target field, not the Maximum Value field, as the target changed as we moved through the year.
A Card visual showing the currently selected store

I created a new page named Stores Drillthrough. I opened the "format" pane and expanded the "Page information" tab. I set the Page type to Drillthrough and set Drill through when to "Used as category". I set "Drill through from" to the country region.

I needed some measures for the gauges as follows:

Profit YTD and Revenue YTD: I had already created this earlier in the project

Previous YTD Profit and Previous YTD Revenue, e.g.

Previous YTD Profit = CALCULATE(SUMX(Orders, (RELATED(Products[Sale Price]) - RELATED(Products[Cost Price])) * Orders[Product Quantity]), DATESYTD(SAMEPERIODLASTYEAR(Dates[Date])))

Profit Goal and Revenue Goal, which were a 20% increase on the previous year's year-to-date profit or revenue at the current point in the year, e.g.

Profit Goal = [Previous YTD Profit] * 1.2

I added the visuals to the drillthrough page.

![](stores_drillthrough_screenshot.png)

### Stores Tooltip Page

I want users to see each store's year-to-date profit performance against the profit target just by hovering the mouse over a store on the map. To do this, I created a custom tooltip page and copied over the profit gauge visual, then set the tooltip of the visual to the tooltip page I had created.

![](stores_tooltip_screenshot.png)

## Cross-Filtering

Power BI has cross-highlighting set up by default on most visuals, so clicking on a region in one visual will automatically filter the others. This can be good for some situations, but in others, it can confuse the user and obscure the message I want to get across. So, I wanted to change the visual interaction settings of my Power BI report pages.

### Executive Summary Page

I ensured the Product Category bar chart and the Top 10 Products table did not filter the card visuals or KPIs. I selected the Product Category bar chart or the Top 10 Products table to make it active.
From the Format tab of the ribbon, I selected Edit Interactions to display the "filter" and "highlight" icons by the other visuals on the report page. To turn off the cross-filtering or cross-highlighting effect of the selected visual, I clicked on the None icon on the card visuals or KPIs I didn't want to be affected.
I repeated the same process for the other visual that I wanted to change the interaction behaviour.
To save the changes, I selected Edit Interactions again from the Format tab of the ribbon.

### Customer Detail Page

I turned off the cross-filtering effect of the Top 20 Customers table on the other visuals. I prevented the Total Customers by Product Column Chart from affecting the Customers' line graph. I enabled the cross-filtering effect of the Total Customers by Country donut chart on the Total Customers by Product donut chart.

### Product Detail Page

I turned off the visual interactions of the Orders vs. Profitability scatter graph and the Top 10 Products table with the other visuals.

## Navigation Bar

I added navigation buttons for the individual report pages.

For each page, a custom icon was available in the custom icons collection I had downloaded earlier in the project. There were two colour variants for each icon. I used a white version for the default button appearance and an orange version so that the button changed colour when I hovered over it with the mouse pointer. 

In the sidebar of the Executive Summary page, I added four new blank buttons, and in the Format > Button Style pane, I made sure the Apply settings to field was set to Default and set each button icon to the relevant white PNG in the Icon tab.

For each button, I went to Format > Button Style > Apply settings to and set it to On Hover, then selected the alternative orange colour of the relevant button under the Icon tab.

For each button, I turned on the Action format option, selected the type as Page navigation, and then selected the appropriate page under Destination.

Finally, I grouped the buttons and copied them to the other pages. I ensured that each button was linked to the relevant other page.

![](navigation_bar_screenshot.png)

## Creating Metrics For Users Outside the Company Using SQL

In industry, clients often don't have direct access to specialised visualisation tools like Power BI. To ensure that data insights could still be extracted and shared with a broader audience, I used SQL queries to extract and disseminate key data without relying solely on visualisation platforms.

I connected to a Postgres database server hosted on Microsoft Azure and ran queries from VSCode using the SQLTools extension.

### Checking the Table and Column Names

This database's table and column names differ from those I used in Power BI.

I printed a list of the tables in the database using a SQL query:

SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES

And saved the result to a CSV file for quick referencing:

I printed a list of the columns in the orders table: 

SELECT COLUMN_NAME
FROM INFORMATION.SCHEMA.COLUMNS
WHERE TABLE_NAME = 'orders'

![](SQL_column_name_query_screenshot.png)

and saved the result to a CSV file called orders_columns.csv

I repeated the same process for each other table in the database, saving the results to CSV files.

### Querying the Database

I wrote SQL queries to answer the following questions. In each case, I exported the result to a CSV file and uploaded it to my Github repository, along with the query as a .sql file. So, for example, question 1 has files named question_1.csv and question_1.sql.

#### Question 1: How many staff were there in all of the UK stores?
For my query, I used the SUM function on staff_numbers from the dim_store table, filtering by the country of 'UK':
SELECT SUM(staff_numbers)
FROM dim_store
WHERE country = 'UK'

#### Question 2: Which month in 2022 had the highest revenue?
My query changes dates in the country_region table to date format. The subsequent query creates a common table expression to calculate the total revenue grouped by month and year. My query ranks the monthly revenues in descending order and then filters to select only the month in 2022 with the highest revenue.

#### Question 3: Which German store type had the highest revenue for 2022?
My query filters the dim_store table to get only the German stores in 2022. It then groups the orders by the sale year and calculates the total revenue for each store type using the SUM function. It then ranks the store types by revenue in descending order using the RANK function and filters the result to select only the store type with the highest rank.

#### Question 4: Create a view where the rows are the store types, and the columns are the total sales, the percentage of total sales, and the number of orders.
My query joins the orders and dim_store tables on the store_code column and uses the SUM, AVG and COUNT functions to calculate the metrics and the ROUND function to format the percentage values.

#### Question 5: Which product category generated the most profit for the "Wiltshire, UK" region in 2021?
My query joins the orders, dim_product, dim_store, and dim_date tables on the product_id, store_id, and date_id columns and filters the result to get only the orders from the "Wiltshire, UK" region in 2021. It then groups the orders by the category column and calculates the total profit for each category using the sale_price and cost columns. It then ranks the categories by profit in descending order using the RANK function and filters the result to get only the category with the highest-ranking profit.
