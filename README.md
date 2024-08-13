# HR-Analytics-in-Power-BI
I am using fictitious datasets from a Tech Company called Atlas Labs

**Primary Goal**: Atlas Labs HR team want to be able to monitor key metrics on employees
<br>
**Secondary Goal**: Understand what factors impact employee attrition

I will working with 1 Fact Table, and 5 Dimension Tables (```Employee```,```EducationLevel```,```RatingLevel```,```SatisfiedLevel``` & ```Date```)
The final data model will follow a snowflake schema.

## First Step: Loading and Preparing Dataset

### 1.Import dataset
Import all the dataset, rename the dataset by add either ```Fact``` or ```Dim``` at the beginning of each table name, depending on the type of table it is.
<br>
Review all the dataset, and ensure that the columns are correctly formatted as text, numbers, and dates where expected.


### 2.Create a dedicated date table
Create a new calculated table that uses the DAX code from ```Dimdate.text```.
```dax
DimDate = 
VAR _minYear = YEAR(MIN(DimEmployee[HireDate]))
VAR _maxYear = YEAR(MAX(DimEmployee[HireDate]))
VAR _fiscalStart = 4 

RETURN
ADDCOLUMNS(

    CALENDAR(

                DATE(_minYear,1,1),

                DATE(_maxYear,12,31)

),

"Year",YEAR([Date]),
"Year Start",DATE( YEAR([Date]),1,1),
"YearEnd",DATE( YEAR([Date]),12,31),
"MonthNumber",MONTH([Date]),
"MonthStart",DATE( YEAR([Date]), MONTH([Date]), 1),
"MonthEnd",EOMONTH([Date],0),
"DaysInMonth",DATEDIFF(DATE( YEAR([Date]), MONTH([Date]), 1),EOMONTH([Date],0),DAY)+1,
"YearMonthNumber",INT(FORMAT([Date],"YYYYMM")),
"YearMonthName",FORMAT([Date],"YYYY-MMM"),
"DayNumber",DAY([Date]),
"DayName",FORMAT([Date],"DDDD"),
"DayNameShort",FORMAT([Date],"DDD"),
"DayOfWeek",WEEKDAY([Date]),
"MonthName",FORMAT([Date],"MMMM"),
"MonthNameShort",FORMAT([Date],"MMM"),
"Quarter",QUARTER([Date]),
"QuarterName","Q"&FORMAT([Date],"Q"),
"YearQuarterNumber",INT(FORMAT([Date],"YYYYQ")),
"YearQuarterName",FORMAT([Date],"YYYY")&" Q"&FORMAT([Date],"Q"),
"QuarterStart",DATE( YEAR([Date]), (QUARTER([Date])*3)-2, 1),
"QuarterEnd",EOMONTH(DATE( YEAR([Date]), QUARTER([Date])*3, 1),0),
"WeekNumber",WEEKNUM([Date]),
"WeekStart", [Date]-WEEKDAY([Date])+1,
"WeekEnd",[Date]+7-WEEKDAY([Date]),
"FiscalYear",if(_fiscalStart=1,YEAR([Date]),YEAR([Date])+ QUOTIENT(MONTH([Date])+ (13-_fiscalStart),13)),
"FiscalQuarter",QUARTER( DATE( YEAR([Date]),MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1,1) ),
"FiscalMonth",MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1
)
```

Once a new table is created, connect 'DimDate' Tble with ```FactPerformanceRating``` table and ```DimEmployee``` Table. And it will shows like the figure belowed.
Now we can see that there are ten relationships between all six tables in our data model.
<div align="center">
  
![image](https://github.com/user-attachments/assets/e4c56992-b1aa-48a4-9060-0c016f98ef92)
  
</div>

### 3.Calculate the key measure
Now I am looking to have visibility on high-level metrics about the state of the employees. Trying to understand the attrition at the company. So I am going to explore the avaliable data and calculate this key measure that will be useful throughout the project.

<br>

Create a new table called ```_Measures```, and then create a ```TotalEmployees``` Measure inside the table, which takes the count of all employees.

The DAX code:
```dax
TotalEmployees = COUNT(DimEmployee[EmployeeID])
```
Create a new measure called ```Active Employees```
```dax
Active Employee = CALCULATE ([TotalEmployees], DimEmployee[Attrition] = "No")
```
<br>

Create a new measure called ```Inactive Employees```

```dax
Inactive Employee = CALCULATE ([TotalEmployees], DimEmployee[Attrition] = "Yes"
```
<br>
These two measures takes the count of all employees that are currently active or inactive, respectively. I used ```DimEmployee[Attrition]``` to determine whether an employeeis active or inactive.
<br>
Then I calculate the '%Attrition Rate' based on these measures. 
<div align="center">
  
![image](https://github.com/user-attachments/assets/59c3b291-7f2f-41a0-810c-f6ea2d62fb3a)

</div>


### 4. Activate relationships between tables.

Now I would like to start with analyzing company's hiring trends over time to see where Hiring Manager see the biggest growth in new employees. This will enable Hiring Team to be able to benchmark their HR metrics against organizations across their industry as well as understand how their employees are performing.
<br>
In this step, I am looking to activate relationships between tables. ```DimDate``` already has an active relationship with ```FactPerformanceRating```. Therefore I will need to utilize ```USERELATIONSHIP()``` to count the number of employees by date.
<br>
Create a new measure called ```TotalEmployeesDate``` that uses the ```CALCULATE()``` function on the previous measure and the ```USERELATIONSHIP()``` function in the filter.
```dax
TotalEmployeesDate =
CALCULATE([TotalEmployees],
USERELATIONSHIP (DimEmployee [HireDate], DimDate[Date]))
```

The figure shows like :

<div align="center">
  
![image](https://github.com/user-attachments/assets/7df49f8e-0c17-4789-8f92-62605f159067)
</div>

### 4. Analyzing Departments and Job Roles
It is essential to ask the HR team working with department managers to understand their teams and what type of typical roles they are hiring into the organization. This will enable every department to plan for new hiring requests in the future. 
<br>

Create a Culstered bar chart to show ```ActiveEmployees``` by ```Department```.
<div align="center">
  
![image](https://github.com/user-attachments/assets/59b8c437-5028-42c0-9dd2-6bd4e05ecdef)

</div>

Create a treemap to display ```ActiveEmployees``` by ```Department``` and ```JobRole```, and we can see that the Software Engineer is the most common job title for the Technology Team.

<div align="center">
  
![image](https://github.com/user-attachments/assets/305b3ced-45f3-4017-8cbd-9be40b30cd96)

</div>


### 5.key insights uncovered so far
1. Atlas Labs has employed over 1,470 people.
2. Atlas Labs currently employes over 1,200 people.
3. The largest department by far is Technology
4. The attrition rate for employees leaving the organization is 16%

In this case, I want to find out what factors impact employees attrition.


## Second Step: Find out next layer of key HR metrics that focus on Diversity and Inclusion

### 1.Demographics - Age & Gender
Create two card visuals to display the minimum and maximum values for ```Age```
<div align="center">
    
![image](https://github.com/user-attachments/assets/2f68f61f-d4e8-4775-9484-32c0f357f8d6)

</div>

Now I want ot create a conditional column called ```AgeBins``` that separates employees ages by bins in the following structure:
```<\20,20-29,30-39,40-49,50>```

The Conditional Column setting looks like:
<div align="center">
    
![image](https://github.com/user-attachments/assets/90fbd1be-f008-4656-bdcc-a90667c25328)
</div>

After that, I created two stacked column chart, and I set a page filter which only shows active employees in the company. And now you can see how employees is different in both age groups and gender
<div align="center">

![image](https://github.com/user-attachments/assets/54bd1b9c-12dc-429e-956d-de139a53400a)
</div>

### 2.Demographics - Marital Status and Ethnicity

Now I want to look at further employee information regarding marital status and ethnicity       
<br>

Firstly, I use a Donut Chart to display the counts of all employees by ```MaritalStatus```. After that, I create a new measure named ```Averagesalary```, and I use a Line and stacked column chart to display the count of all employees and their average salary by ```Ethnicity```.

The graph below shows the data for all active employees.
<div align="center">
    
![image](https://github.com/user-attachments/assets/8dcc7aef-1e79-4ecf-aca2-48c168ad7aba)

</div>


### 3. Performance Tracker: Part 1

Now I want to help the HR team to have a view where they can continually track an individual employee's performance scores based on their yearly performance reviews.

<br>

Firstly, I created a calculated column named ```FullName``` in the ```DimEmployee``` table that combines ```FirstName``` and ```LastName```

```dax
FullName = COMBINEVALUES(" ",DimEmployee[FirstName],DimEployee[LastName])
```

A slicer has been created, so we will be able to filter the report page based on the employee's full name. And I create a card visual that can shows the `HireDate` of selected Employee. Then, I create a new masure ```LastReviewDate``` that gets the last performance review for the selected individual.

```dax
LastReviewDate = IF(
MAX(FactPerformanceRating[ReviewDate]) = BLANK (), "No Review Yet",
Max(FactPerformanceRating[ReviewDate])
)
```

Next, I create a new measure called ```NextReviewDate``` that calculates when the next review is due. It should be 365 days after the ```LastReviewDate```, so here is the DAX Function:

```dax
Next Review Date = VAR reviewOrHire =
IF(
    MAX(FactPerformanceRating[ReviewDate]) = BLANK(),
    MAX(DimEmployee[HireDate]),
    MAX(FactPerformanceRating[ReviewDate])
)
RETURN
    reviewOrHire + 365
```

Once we created the new measure and create a new card visual on that, we can see the employee's start date, last review date and the next review date. As shown in the graph below.

<div align="center">
    
![image](https://github.com/user-attachments/assets/e391fb19-757e-4057-8455-07d86dd6309f)
</div>

### 4. Performance Tracker: Part 2
Now I want to look at the information on their individual review ratings

Firstly, I created a ```JobSatisfaction``` measure inside the ```_Measures``` tables, and it indicates the maximum value inside ```FactPerformanceRating[JobSatisfaction]``` level.
<br>
And now I found out that the other three satisfaction metrics do not have an active relationship to ```DimsSatisfiedLevel```. In this case, I will create three new measures ```EnvironmentSatisfaction```, ```RelationshipSatisfaction```, and ```WorkLifeBalance```.

```dax
EnvironmentSatisfaction =
CALCULATE(
    MAX(FactPerformanceRating[EnvironmentSatisfaction]),
    USERELATIONSHIP(FactPerformanceRating[EnvironmentSatisfaction], DimSatisfiedLevel[SatisfactionID])
)
```
```dax
RelationshipSatisfaction =
CALCULATE(
    MAX(FactPerformanceRating[RelationshipSatisfaction]),
    USERELATIONSHIP(FactPerformanceRating[RelationshipSatisfaction], DimSatisfiedLevel[SatisfactionID])
)
```
```dax
WorkLifeBalance =
CALCULATE(
    MAX(FactPerformanceRating[WorkLifeBalance]),
    USERELATIONSHIP(FactPerformanceRating[WorkLifeBalance], DimSatisfiedLevel[SatisfactionID])
)
```
<br>
Create two more measures, ```SelfRating``` and ```ManagerRating```, which provide the max rating id.

```dax
SelfRating = CALCULATE(
    MAX(FactPerformanceRating[SelfRating]),
    USERELATIONSHIP(FactPerformanceRating[SelfRating], DimRatingLevel[RatingID]))
)
```

```dax
SelfRating = CALCULATE(
    MAX(FactPerformanceRating[ManagerRating]),
    USERELATIONSHIP(FactPerformanceRating[ManagerRating], DimRatingLevel[RatingID]))
)
```
<div align="center">
![image](https://github.com/user-attachments/assets/f35c3195-5fe8-4cbc-92a0-787afc3cb1c9)
</div>

As you can see from the dashboard here, this Power BI dashboard tracks an employee's (For example, Estelle Chung) performance and satisfaction metrics from 2019 to 2022. It displays key dates like start date, last review, and next review. The dashboard features six line graphs showing trends in Work-Life Balance, Environment Satisfaction, Self Rating, Relationship Satisfaction, Job Satisfaction, and Manager Rating over time. Each metric is rated on a scale from 1 (Unacceptable/Very Dissatisfied) to 5 (Above and Beyond/Very Satisfied). This visual representation allows for quick assessment of the employee's performance trends and satisfaction levels across various aspects of their work experience, enabling easy identification of strengths, areas for improvement, and overall progress over time.
