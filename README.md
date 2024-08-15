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

### 5. Analyzing Departments and Job Roles
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


### 6.key insights uncovered so far
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


### 5. Key insights from Second Part
1. Majority of employees are between 20-29 years old.
2. Currently, Atlas Labs employe 2.7% more women than men.
3. Employees who identify as non-binary make up 8.5% of total employees.
4. Employees who identify as White have the highest average salary.
5. Employees who identify as mixed or multiple ethnic groups have one of the lowest average salaries.

Now, there is an interesting question that comes out of my mind from the insights I've uncovered:
    `Does our demographics insights impact employee attrition?`

## Final Step: Understanding Employee Attrition

### 1. Attrition rate by Department & Job role

Firstly,I've create a card visual that shows the `Attritoin Rate`, a stacked column chart to displays the `%Attrition Rate` for each department and job role.
<br>
Secondly,I want to understand the attrition rate based on `HireDate`. So I need to create a new measure called `InactiveEmployeesDate`. Then, I want to create a new measure called `%Attrition Rate Date` which calculates the rate of attrition based on `InactiveEmployeesDate` and `TotalEmployeesDate`. After that, I created a line chart to displays the `%Attrition Rate Date` over time.

```dax
InactiveEmployeesDate = CALCULATE([InactiveEmployees],
    USERELATIONSHIP(DimEmployee[HireDate], DimDate[Date])
)
```

```dax
%Attrition Rate Date = DIVIDE([InactiveEmployees], [TotalEmployeesDate])
```
<div align="center">
    
![image](https://github.com/user-attachments/assets/f0e65500-ef64-4ac4-bd3e-ee37e0b5a5ee)
</div>

As you can see from the graph,

Key Insights from Employee Attrition Analysis

1. Overall Attrition Rate
- The overall attrition rate is **16.1%**, representing the percentage of employees who have left the organization.

2. Attrition Rate by Department and Job Role
- The bar chart breaks down the attrition rate by department and job role:
  - **Sales**: Highest attrition rate with significant departures in roles like Analytics and Engineering.
  - **Human Resources**: Moderate attrition rate, with HR Executives accounting for a notable portion of the departures.
  - **Technology**: A high attrition rate distributed across various roles, including Engineering and Data Science.

3. Attrition by Hire Date
- The line chart visualizes the attrition rate based on hire dates from **2012 to the present**:
  - A **notable spike in 2016**, indicating a possible organizational event or external factor affecting employee turnover.
  - Post-2018 shows more stable attrition rates, with fewer extreme fluctuations, especially after 2020.

### 2. Other factors that may affect attrition

Since I want to discover more about what other factors may affect on employee attrition. I want to design more figure to explore. As you can see from the table below. There is a stacked column chart which shows the attrition by travel frequency, attrition by overtime requirement

<div align="center">
    
![image](https://github.com/user-attachments/assets/273edca2-c17b-49da-861c-edb4b9f409ca)
</div>

Key Insights from Employee Attrition Analysis

 1. Attrition by Travel Frequency
- **Frequent Travelers**: Highest attrition rate despite fewer employees in this group.
- **No Travel**: Lowest attrition rate, with "Some Travel" employees falling in between.

 2. Attrition by Overtime Requirement
- Employees required to work **overtime** show a **significantly higher attrition rate** compared to those who do not have overtime requirements.

 3. Attrition by Tenure
- **Employees with less than 2 years of tenure** show the highest attrition rates, especially those with around 1 year at the company.
- **Longer tenure** (6+ years) is associated with a lower attrition rate, indicating that employees who stay longer are less likely to leave.

I suggest that frequent travel, overtime, and shorter tenure are key factors contributing to higher employee turnover.


### 3. Format the Dashboard: Cleaning up the color, chart and title formatting

I've changed the format,size, color, theme of the graph. As you can see from the dashboard below.
<div align="center">
    
![image](https://github.com/user-attachments/assets/2a43c1c8-9cdd-4913-b89a-c2374b3031f4)
</div>

<div align="center">
    
![image](https://github.com/user-attachments/assets/0fbb938b-6c81-4db2-8604-e227c4075423)
</div>

<div align="center">
    
![image](https://github.com/user-attachments/assets/d690b0e8-9b82-4f51-a9ca-44a2c98a5b9e)
</div>

<div align="center">
    
![image](https://github.com/user-attachments/assets/d848ab30-dabd-4c61-a3be-2fafe9b3d699)
</div>

# Atlas Labs HR Analytics Dashboard

## Overview
This dashboard provides comprehensive insights into Atlas Labs' workforce, covering various aspects such as employee demographics, performance, and attrition.

## Key Metrics
- **Total Employees**: 1470
- **Active Employees**: 1233
- **Inactive Employees**: 237
- **Attrition Rate**: 16.1%

## Employee Demographics
- **Age Range**: 18 to 51 years old
- **Largest Age Group**: 20-29 years
- **Gender Distribution**: Majority male, with female representation increasing in younger age groups
- **Marital Status**: 
  - 42.45% married 
  - 37.35% single 
  - 20.2% divorced

## Performance Insights
- **Job Satisfaction**: Declining trend from 2020 to 2022
- **Relationship Satisfaction**: Sharp decline from 2020 to 2022
- **Work-Life Balance**: Fluctuating, with a significant dip in 2021 but recovery in 2022
- **Manager Ratings**: Steady decline from 2020 to 2022

## Attrition Analysis
- **Highest Attrition**: Sales Representatives and Facilities roles
- **Travel Impact**: Frequent travelers have the highest attrition rate
- **Overtime**: Employees working overtime show higher attrition rates
- **Tenure**: Highest attrition observed in the first two years of employment

## Business Insights
- **Retention Focus**: Prioritize retention strategies for employees in their first two years, particularly in sales and facilities roles.
- **Work-Life Balance**: Implement policies to improve work-life balance, especially for frequent travelers and those working overtime.
- **Management Training**: Address the declining manager ratings through leadership development programs.
- **Diversity Initiatives**: Focus on increasing diversity, particularly in older age groups and senior positions.
- **Employee Satisfaction**: Investig
