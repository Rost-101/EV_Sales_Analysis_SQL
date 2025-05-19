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
  GROUP BY Region, Brand),
RegionTotals AS (
  SELECT Region, SUM(Revenue) AS Region_Total_Revenue
  FROM `gentle-respect-458915-s5.12.EV_cars`
  GROUP BY Region),
RankedBrands AS (
  SELECT 
    rbr.Region,
    rbr.Brand,
    rbr.Total_Revenue AS Top_Revenue,
    rt.Region_Total_Revenue,
    ROW_NUMBER() OVER (PARTITION BY rbr.Region ORDER BY rbr.Total_Revenue DESC) AS rank
  FROM RevenueByBrandRegion rbr
  JOIN RegionTotals rt ON rbr.Region = rt.Region)
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
#### ✅ Результат: 
<img src="https://github.com/user-attachments/assets/c488cac1-e01b-46d2-ba94-c89ae4d88e9a" width="600">

Бренд Toyota демонструє найбільшу частку доходу (22.35%) у Африці з доходом 1,248,52791, тоді як Kia лідирує в Азії з 19.12% (590,0556). У Північній Америці Hyundai має 15.81% (11,474,486), а BYD у Південній Америці — 17.19% (3,065,970), що відображає регіональну специфіку популярності брендів.

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
#### ✅ Результат: 
<img src="https://github.com/user-attachments/assets/ca4b7bef-9e33-4cfa-be95-f6580abb02ee" width="600">

Kia у Північній Америці має найвищу середню знижку (11.36%) з доходом 113144270, тоді як Toyota у Південній Америці пропонує найбільшу знижку (16.5%) при доході 10727248. У Європі Volkswagen демонструє знижку 12.5% з доходом 16700280, а BYD в Африці — 11.0% з доходом 24763916, що підкреслює регіональні відмінності у стратегіях знижок та доходах брендів

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
#### ✅ Результат: 
<img src="https://github.com/user-attachments/assets/a2e468b8-e0f6-4a48-a404-ff8c87d03cfc" width="600">

У січні 2023 року BMW лідирує з ринковою часткою 19.77% (2393 одиниці), за ним йдуть Ford з 17.91% (2168 одиниць) та Kia з 17.08% (2068 одиниць). Tesla та BYD мають найменші частки — 3.03% (367 одиниць) та 3.68% (445 одиниць) відповідно, що відображає значну перевагу європейських та корейських брендів у цьому місяці. Результати охоплюють багато місяців, але тут наведено приклад лише за січень 2023 року.

### Завдання 5: Аналіз ефективності знижок
**Мета**: Проаналізувати, чи впливають знижки (Discount_Percentage) на продажі (Units_Sold), порівнюючи середні продажі для різних рівнів знижок у кожному регіоні (Region).

```sql
WITH DiscountLevels AS (
    SELECT 
        Region,
        CASE 
            WHEN Discount_Percentage = 0 THEN 'No Discount'
            WHEN Discount_Percentage BETWEEN 1 AND 10 THEN 'Low (1-10%)'
            WHEN Discount_Percentage BETWEEN 11 AND 20 THEN 'Medium (11-20%)'
            ELSE 'High (>20%)'
        END AS Discount_Level,
        AVG(Units_Sold) AS Avg_Units_Sold,
        AVG(Discount_Percentage) AS Avg_Discount
    FROM `12.EV_cars`
    GROUP BY Region, 
        CASE 
            WHEN Discount_Percentage = 0 THEN 'No Discount'
            WHEN Discount_Percentage BETWEEN 1 AND 10 THEN 'Low (1-10%)'
            WHEN Discount_Percentage BETWEEN 11 AND 20 THEN 'Medium (11-20%)'
            ELSE 'High (>20%)'END)
SELECT 
    Region,
    Discount_Level,
    ROUND(Avg_Discount, 2) AS Average_Discount,
    ROUND(Avg_Units_Sold, 2) AS Average_Units_Sold
FROM DiscountLevels
ORDER BY Region, Avg_Units_Sold DESC;
```
#### ✅ Результат: 
<img src="https://github.com/user-attachments/assets/5aea626a-9319-44d2-86ba-3f85f63a12fa" width="600">

У Африці середні продажі найбільші при середній знижці 14.82% (237.33 одиниць), тоді як у Європі середня знижка 15.58% забезпечує 253.58 одиниць. У Північній Америці найбільші продажі (223.02 одиниці) спостерігаються при знижці 15.5%, тоді як у Азії при відсутності знижок продажі нижчі (201.0 одиниць), що вказує на різний вплив знижок на продажі в регіонах.

### Завдання 6: Середній дохід на одиницю за типом транспортного засобу
**Мета**: Підрахувати середній дохід на одиницю (Revenue / Units_Sold) для кожного типу транспортного засобу (Vehicle_Type) у регіонах, де продажі перевищують 200 одиниць, щоб оцінити прибутковість.

```sql
SELECT 
    Vehicle_Type,
    Region,
    ROUND(SUM(Revenue) / SUM(Units_Sold), 2) AS Avg_Revenue_Per_Unit,
    SUM(Units_Sold) AS Total_Units_Sold
FROM `12.EV_cars`
GROUP BY Vehicle_Type, Region
HAVING SUM(Units_Sold) > 200
ORDER BY Avg_Revenue_Per_Unit DESC;
```
#### ✅ Результат: 
<img src="https://github.com/user-attachments/assets/8b986b78-d5b0-455b-8205-83bc9999debd" width="600">

Найвищий середній дохід на одиницю (23855.28) спостерігається для SUV в Азії (2467 одиниць), тоді як у Північній Америці SUV має дохід 23554.05 (4067 одиниць). У Африці кросовери досягають 24449.52 (2186 одиниць), а в Океанії хетчбеки — 22288.63 (12096 одиниць), що вказує на різну прибутковість типів транспортних засобів у регіонах.

### Завдання 7: Вплив сегменту клієнтів на середню знижку
**Мета**: Проаналізувати, як сегмент клієнтів (Customer_Segment) впливає на середню знижку (Discount_Percentage), щоб зрозуміти стратегії ціноутворення для різних аудиторій.

```sql
SELECT 
    Customer_Segment,
    ROUND(AVG(Discount_Percentage), 2) AS Avg_Discount_Percentage,
    COUNT(*) AS Total_Records,
    SUM(Units_Sold) AS Total_Units_Sold
FROM `12.EV_cars`
GROUP BY Customer_Segment
ORDER BY Avg_Discount_Percentage DESC;
```
#### ✅ Результат: 
<img src="https://github.com/user-attachments/assets/a6ecb474-9f99-42f2-b9f6-636e1b1b6796" width="600">

Найвищу середню знижку (9.6%) отримують клієнти сегменту High Income (27960 одиниць), що може вказувати на стратегію залучення преміум-аудиторії. Eco-Conscious сегмент має знижку 9.16% (21127 одиниць), тоді як Budget Conscious (8.69%, 29449 одиниць) та Middle Income (8.48%, 15919 одиниць) отримують менші знижки, відображаючи диференційований підхід до ціноутворення залежно від сегменту.

### Завдання 8: Топ-5 брендів із найбільшим доходом та їхній внесок у загальний дохід
**Мета**: Визначити топ-5 брендів (Brand) із найбільшим сумарним доходом (Revenue) і розрахувати їхню частку у загальному доході, а також показати кумулятивний дохід по мірі зростання.

```sql
SELECT 
    Brand,
    Total_Revenue,
    ROUND((Total_Revenue / SUM(Total_Revenue) OVER ()) * 100, 2) AS Revenue_Share_Percentage,
    SUM(Total_Revenue) OVER (ORDER BY Total_Revenue DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS Cumulative_Revenue
FROM (
    SELECT
        Brand,
        SUM(Revenue) AS Total_Revenue
    FROM `12.EV_cars`
    GROUP BY Brand) AS BrandTotals
ORDER BY Total_Revenue DESC
LIMIT 5;
```

#### ✅ Результат: 
<img src="https://github.com/user-attachments/assets/74444b7a-64f5-4501-b584-fed311628fc4" width="600">

Kia демонструє найбільший сумарний дохід у розмірі 382151251$, що становить 14.63% загального доходу, займаючи провідну позицію. BMW (358120737$, 13.71%) і Hyundai (351716271$, 13.47%) також мають значний внесок, тоді як Ford (295197328$, 11.3%) і Nissan (291035195$, 11.14%) завершують топ-5. Кумулятивний дохід досягає 1678220782$ для цих п’яти брендів, що відображає їхній домінуючий вплив на загальну виручку.

### Короткий загальний висновок
🌟 Аналіз продажів електромобілів за допомогою SQL виявив ключові тенденції: Kia та BMW демонструють лідерство в доходах і ринковій частці, особливо в Північній Америці та Азії, з ефективним використанням знижок для сегменту High Income. 📈 Знижки значно впливають на продажі, особливо в Європі та Африці, де середні продажі зростають при знижках 15-16%. 🚗 SUV і кросовери є найбільш прибутковими типами транспортних засобів, особливо в Азії та Африці. 🌍 Регіональні відмінності підкреслюють необхідність адаптивних маркетингових стратегій для різних сегментів і ринків. 🎯




