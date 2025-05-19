# Аналіз продажів електромобілів за допомогою SQL

Цей репозиторій містить проєкт, який демонструє аналіз продажів електромобілів (EV) та їхнього впровадження за допомогою SQL. Початковий аналіз був виконаний у Google Sheets, а потім перенесений до SQL для більш ефективної обробки даних та отримання глибших інсайтів.

## Огляд

Проєкт демонструє мої навички аналізу даних продажів електромобілів, які спочатку були зібрані та проаналізовані в Google Sheets, а потім переведені в SQL для поглибленого аналізу. Набір даних включає різні атрибути, такі як регіони продажів, марки автомобілів, типи, ємність батарей, сегменти клієнтів та метрики доходів. Цей README описує структуру даних та підхід до аналізу в SQL.

## Набір даних

Набір даних, використаний у цьому проєкті, включає такі поля:

- **Date**: Місяць у форматі РРРР-ММ  
- **Region**: Географічний регіон, де відбувалися продажі  
- **Brand**: Автомобільна марка (наприклад, Toyota, BYD, Tesla...)  
- **Vehicle_Type**: Категорія (Crossover, Truck, Sedan, Hatchback...)  
- **Battery_Capacity_kWh**: Ємність батареї в кіловат-годинах  
- **Discount_Percentage**: Застосована знижка у відсотках  
- **Customer_Segment**: Сегмент клієнтів ( High Income, Middle Income, Budget Conscious)  
- **Fast_Charging_Option**: Чи підтримує транспортний засіб швидку зарядку  
- **Units_Sold**: Загальна кількість проданих одиниць  
- **Revenue**: Загальний дохід від проданих одиниць  

Дані з Google Sheets були імпортовані до бази даних SQL для подальшого аналізу. Використання SQL дозволило:
- Виконувати складні запити для агрегації даних (наприклад, загальний дохід за регіонами або марками).
- Фільтрувати дані за різними критеріями (наприклад, сегменти клієнтів або типи транспортних засобів).
- Аналізувати тренди продажів за місяцями та регіонами.

## Виконані завдання

Нижче наведено приклади SQL-запитів, які я виконав для аналізу даних, разом із їхньою метою.

### Завдання 1: Аналіз доходу за брендами в Північній Америці
**Мета**: Визначити загальний дохід (Revenue) для кожного бренду (Brand) у регіоні Північна Америка (North America), де знижка перевищує 15% і транспортний засіб підтримує швидку зарядку.

```sql
SELECT Brand, SUM(Revenue) AS Total_Revenue
FROM `gentle-respect-458915-s5.12.EV_cars`
WHERE Region = 'North America' AND Discount_Percentage > 15 AND Fast_Charging_Option = TRUE
GROUP BY Brand
ORDER BY Total_Revenue DESC;
```
#### ✅ Результат: 
<img src="https://github.com/user-attachments/assets/db06f651-2f7d-4ff2-a7dd-40da3314b502" width="350">

Бренд Kia лідирує з доходом 2,470,858 у Північній Америці за умов знижки понад 15% і наявності швидкої зарядки, що свідчить про успішну маркетингову стратегію. Ford (1,722,709) і Nissan (848,784) також показують високі результати, тоді як Toyota з доходом 357,124 займає останню позицію

### Завдання 2: Найприбутковіший бренд у кожному регіоні
**Мета**: Для кожного регіону (Region) знайти бренд (Brand), який має найбільший сумарний дохід (Revenue), і визначити, яку частку (у відсотках) цей дохід становить від загального доходу регіону.

```sql
WITH RevenueByBrandRegion AS (
  SELECT Region, Brand, SUM(Revenue) AS Total_Revenue
  FROM `gentle-respect-458915-s5.12.EV_cars`
  GROUP BY Region, Brand
),
RegionTotals AS (
  SELECT Region, SUM(Revenue) AS Region_Total_Revenue
  FROM `gentle-respect-458915-s5.12.EV_cars`
  GROUP BY Region
),
RankedBrands AS (
  SELECT 
    rbr.Region,
    rbr.Brand,
    rbr.Total_Revenue AS Top_Revenue,
    rt.Region_Total_Revenue,
    ROW_NUMBER() OVER (PARTITION BY rbr.Region ORDER BY rbr.Total_Revenue DESC) AS rank
  FROM RevenueByBrandRegion rbr
  JOIN RegionTotals rt ON rbr.Region = rt.Region
)
SELECT 
  Region,
  Brand,
  Top_Revenue,
  Region_Total_Revenue,
  ROUND((Top_Revenue / Region_Total_Revenue) * 100, 2) AS Revenue_Share_Percentage
FROM RankedBrands
WHERE rank = 1
ORDER BY Region;
```

### Завдання 3: Бренди з найвищою середньою знижкою в регіонах
**Мета**: Визначити бренди (Brand), які мають найвищу середню знижку (Discount_Percentage) у кожному регіоні (Region), враховуючи лише бренди зі знижкою понад 10%. Для таких брендів підрахувати середню ємність батареї (Battery_Capacity_kWh) та загальний дохід (Revenue).

```sql
WITH CTE AS (
  SELECT Region, Brand,
    AVG(Discount_Percentage) AS avg_discount_percentage,
    ROUND(AVG(Battery_Capacity_kWh), 2) AS avg_battery_capacity,
    SUM(Revenue) AS total_revenue
  FROM `gentle-respect-458915-s5.12.EV_cars`
  GROUP BY Region, Brand
  HAVING AVG(Discount_Percentage) > 10
),
CTE2 AS (
  SELECT Region, Brand, avg_discount_percentage, avg_battery_capacity, total_revenue,
    ROW_NUMBER() OVER (PARTITION BY Region ORDER BY avg_discount_percentage DESC) AS rank
  FROM CTE
)
SELECT Region, Brand,
  avg_discount_percentage AS Avg_Discount_Percentage,
  avg_battery_capacity AS Avg_Battery_Capacity_kWh,
  total_revenue AS Total_Revenue
FROM CTE2
WHERE rank = 1
ORDER BY total_revenue DESC;
```

### Завдання 4: Ринкова частка брендів за місяцями
**Мета**: Для кожного місяця (Date) і бренду (Brand) підрахувати кількість проданих одиниць (Units_Sold), загальну кількість проданих одиниць у місяці та ринкову частку бренду (у відсотках).

```sql
SELECT
    date,
    Brand,
    SUM(Units_Sold) AS Brand_Sold,
    SUM(SUM(Units_Sold)) OVER (PARTITION BY date) AS Total_Month_Sold,
    ROUND(
        SUM(Units_Sold) * 100.0 / SUM(SUM(Units_Sold)) OVER (PARTITION BY date), 
        2
    ) AS Brand_Market_Share
FROM `12.EV_cars`
GROUP BY date, Brand
ORDER BY date, Brand_Market_Share DESC;
```

### Завдання 5: Типи автомобілів із падінням продажів
**Мета**: Визначити типи автомобілів (Vehicle_Type), у яких продажі (Units_Sold) знизилися на 30% або більше порівняно з попереднім місяцем, та показати відсоток падіння.

```sql
WITH SalesWithLag AS (
    SELECT
        Date,
        Vehicle_Type,
        SUM(Units_Sold) AS Total_Sold,
        LAG(SUM(Units_Sold)) OVER (PARTITION BY Vehicle_Type ORDER BY Date) AS Previous_Units_Sold
    FROM `12.EV_cars`
    GROUP BY Date, Vehicle_Type
)
SELECT
    Date,
    Vehicle_Type,
    Previous_Units_Sold AS Previous_Month_Sales,
    Total_Sold AS Current_Month_Sales,
    ROUND(((Previous_Units_Sold - Total_Sold) * 100.0 / Previous_Units_Sold), 2) AS Drop_Percent
FROM SalesWithLag
WHERE Previous_Units_Sold IS NOT NULL
  AND ROUND(((Previous_Units_Sold - Total_Sold) * 100.0 / Previous_Units_Sold), 2) >= 30
ORDER BY Date, Vehicle_Type;
```
