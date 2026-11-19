# RentalFlow: Car Rental Management System

**Student name:** Esther Neema\
**Student ID:** 29706\
**Course:** Database Development with PL/SQL (INSY8311)\
**Lecturer:** Eric M.\
**Database:** Oracle Database with PL/SQL

## 1. Problem statement

Car rental is growing in Rwanda. The number of registered vehicles grew from 268,537 in 2021 to 366,549 in 2024 (NISR, 2025), and visits to national parks grew from 107,976 in 2022 to 135,869 in 2023 (NISR, n.d.). Many visitors and local people need a car for a few hours, a few days, or a few weeks.

But many small rental businesses still keep their records on paper and check if a car is free by phone. This causes double bookings, lost records, and mistakes in payments (Hasan & Mat Nawi, 2026). Existing systems also have gaps:

- They rent cars by full day or half day, not by hour, day, and week (Hasan & Mat Nawi, 2026; Europcar, n.d.).
- They accept credit cards, cash, or QR codes, but not mobile money, which many people in Rwanda use (Europcar, n.d.).
- They give the owner weak reports, so the owner cannot easily see which cars are rented most or how much money the business earns (Hasan & Mat Nawi, 2026; Ogbiti & Aaron, 2024).

There is also no public Rwandan data on car rental (NISR, 2025), so each business must keep good records of its own.

## 2. Objectives

### Main objective

To build an Oracle PL/SQL database system that helps a Rwandan car rental business manage cars, customers, bookings, and payments, and gives the owner clear reports to make better decisions.

### Specific objectives

1. Keep accurate records of cars, customers, bookings, and payments in one database.
2. Support rentals by the **hour, day, or week**, with the price calculated automatically.
3. Stop double bookings, so the same car cannot be booked twice for the same time.
4. Record payments made by **mobile money (MTN MoMo, Airtel Money)**, cash, or card.
5. Give the owner reports on the most rented cars, income per month, and car use.
6. Protect the data with rules, checks, and an audit trail of every change.

## 3. Data source

- **Dataset:** Cornell Car Rental Dataset (Kumar, 2020), from Kaggle.
- **Size:** 5,851 real rental cars with make, model, type, year, fuel type, daily price, rating, and number of times rented (195,876 rentals in total).
- **Licence:** CC BY 4.0 (allows educational use with credit to the creator).
- **Privacy:** We removed the owner ID and the exact car location, so the data has no information that identifies real people.
- **Use in this project:** The cars and prices come from the dataset. Bookings follow the real number of rentals per car. Payments are calculated from real prices. Customers are sample (made-up) records. Prices are converted from US dollars to Rwandan francs (RWF).

Full details: [data/data_source.md](data/data_source.md). Data file: [data/raw/car_rental_cars.csv](data/raw/car_rental_cars.csv).

## 4. Innovation

What makes this system different from the systems we studied:

1. **Flexible rental time:** one booking can be by hour, day, or week, and the database picks the right price.
2. **Mobile money ready:** payments store the method (MoMo, Airtel Money, cash, card) and the transaction reference.
3. **No double booking by design:** a trigger checks the dates before a booking is saved, so the rule works for every app that uses the database.
4. **Owner reports inside the database:** analytic (window) functions rank the most rented cars, show income per month with running totals, and show how busy each car is.
5. **Full audit trail:** every insert, update, and delete on key tables is recorded with the user and the time.

**Innovation option and tool:** a **user interface** built with **Python (Streamlit)**. Staff use it to book cars, record payments, and return cars, and the owner views reports. Every action calls RentalFlow's PL/SQL procedures on the real Oracle database, so the business rules stay in PL/SQL.

## 5. Planned tables

| Table | Purpose | Main columns |
|-------|---------|--------------|
| BRANCHES | Places where cars are picked up and returned | branch_id (PK), branch_name, district, phone |
| CAR_CATEGORIES | Car groups and their prices | category_id (PK), category_name, hourly_rate, daily_rate, weekly_rate |
| CARS | Each car in the fleet | car_id (PK), plate_no (UNIQUE), make, model, car_year, fuel_type, category_id (FK), branch_id (FK), status |
| CUSTOMERS | People who rent cars | customer_id (PK), full_name, national_id_or_passport (UNIQUE), phone, email, licence_no |
| BOOKINGS | Each rental | booking_id (PK), customer_id (FK), car_id (FK), start_time, end_time, rental_unit (HOUR/DAY/WEEK), total_amount, status |
| PAYMENTS | Money received for bookings | payment_id (PK), booking_id (FK), amount, method (MOMO/AIRTEL/CASH/CARD), reference_no, paid_at |
| MAINTENANCE | Car service records, so cars under repair are not rented | maintenance_id (PK), car_id (FK), start_date, end_date, cost, description |
| PUBLIC_HOLIDAYS | Rwanda public holidays, used by the business rules | holiday_date (PK), holiday_name |
| AUDIT_LOG | Record of every change to key tables | audit_id (PK), table_name, action, done_by, done_at, status, details |

Main rules (constraints): end_time must be after start_time, amounts must be above zero, and status and method columns only accept listed values (CHECK constraints).

## 6. Planned PL/SQL features

| Feature | How it will be used |
|---------|--------------------|
| **Procedures** | `make_booking`, `cancel_booking`, `return_car`, `record_payment` |
| **Functions** | `calc_rental_price` (price for hour, day, or week), `is_car_available`, `customer_balance` |
| **Cursors** | Loop through bookings to build the monthly income report and the list of late returns |
| **Packages** | `rental_pkg` (booking and payment logic) and `report_pkg` (owner reports) |
| **Triggers** | Block double bookings; update car status when a car is rented or returned; write to AUDIT_LOG |
| **Compound trigger** | Audit all changes on BOOKINGS and PAYMENTS in one place |
| **Business rule** | Block changes to data on weekdays and on public holidays (using PUBLIC_HOLIDAYS) as the course requires, with every blocked try saved in AUDIT_LOG |
| **Exception handling** | Custom errors such as "car not available" and "payment more than amount due" |
| **Window functions** | `RANK()` for the most rented cars, `SUM() OVER` for running income, `LAG()` to compare months |
| **Sequences and indexes** | Automatic IDs and faster searches on bookings by car and date |

## 7. Project folders

```
docs/research/       Research log and sources
docs/proposal/       Project proposal
docs/presentation/   Presentation slides
docs/business_process/  business_process_diagram and business_process_explanation
data/                Dataset and data source notes
```

Later phases will add more folders (for example, SQL scripts).

## 8. business_process_diagram
https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&dark=auto#R7V1Zc6O4Fv41rpp5iAsQmx87zkx3V01PdyW363aebikg25pg5AE5S%2F%2F6K7HY5kCMwIg4M8lDAkIIOMuns0mZoPn66WOCN6svLCTRxDLCpwm6mliW7xjit2x4zhtc28sblgkN8yZz33BDf5KisbhvuaUhSSsdOWMRp5tqY8DimAS80oaThD1Wuy1YVH3qBi9JreEmwFG99b805CvwWbL9E6HLVflk0yiurHHZuWhIVzhkjwdN6LcJmieM8fxo%2FTQnkaRdSZf8vt9fuLp7sYTEXOWG25nz1buxvbvr22j9%2FdMVSbzrC6c%2BSjFwyp9LGqSPdB3hWJxdrlhCf7KY40hcMkTD44pycrPBgez7KERAduJredkUh%2FlYDzjaFmPNtylna5IUF0jCydPBc4tX%2F0hEH548iy6rQ%2BraBS0f96wwbb9oLMYp%2BxQCV5IfF4Kw3A29p5U4KMjVgXR%2BnXTHSHxIT56w%2B504WVWKCTHZyH7rp6VUqOkiYo%2FBCid8mnLx%2B3%2FWC0TfkISKDxOURVfi7YV%2BkG%2F7pjojejEA1emPqtRHgPqOJuqbRm%2FyJ2wbhyQs6N0iwEXnA1I2y%2FU1%2BXtLUi4%2FVaIRFgLuRuK9LkP6IA6X8vAX%2FixYaxlscdhtHmIuWucbGtxvN7%2BW9wmqHNzai1tug7ZYVXaZZpVfpi5tmeXjkrAGt535R%2BLwg0R2cRZEOE1p0MiyAqBCssDbiP%2BB70h0iYP7ZXZ5ziImeZnD%2F2Ukr37DYUjjZXafZdg7qgu1W5Jjr1gSDWjXceaUM0hCIszpQ5UqBxxzjnCkeMI3RmO%2B56lrT12%2FwlUbjJGybRKQ4rbDKQOMtJOWF8bJKVMbJxOQ3WefoOPW2xWaM5EF061y0OsrCWAcEw6kWRRs%2B7zg%2FhuWPAzJhqVUkkl%2BsGVck8wyEmYmIboQ23aP6%2BRgiO2eGcm%2Fp6ScMg0ch5m68G0Sy%2B%2BQLGDyiK9kp3Cb%2Fc7mVT1c8GZjccE5FQMzDVfg86uAJXmi%2FIccdWo4XnF%2BK69PnZlfnF89FQ%2FKTp6LkxFgtuD1hTGdmbMW5BX0yiCh7LCRSJgeAWbDAMjsHgdg2L90NPfClb9C8%2B0zYOfN%2Bs4D8DVmxrgTgVuXfr0O6yVj91JmLePrYkGDET3X3Zw7vPOEFDHEHQfar0lApD5K1dy5URnEByyKSCAVyQh2kQMx73JMo1QXuEOnCGnjg9cX3OuM%2BXcYuJ1hFtqtdhfUhHf3Rk34Fv64oIlUTbmR9P1mu1yWWs6EyZbsDDuWnGK4NYSnoG4jB0y7unQbuafqdpvhhl7VcIsFkX4Uw2YnudnmOeX53mzLzkq7rbT4jIq1ZzkKxl57IAaNgE17q9AwZl4LXtWgwALyZ8KImyqmIGCIQWNBN6QUX17kXfBdKYKqcVnUMIUJVcmEsSq0OKLLWEq0GDCLZUtUoAGOPhQX1jQM5RiXCUnpz%2BJVpNgUs4MY3LmcOFeihcYCbSj%2FD3nixeiDzoT77IaBlwkh6iZKo%2FQdiJrrqota2YUtFinRE47p7ZWqgpt9Dl6pMfUROsApc2q6ngJQjYQ%2FQiaGdksdAE8XMKnY0t%2FtYl45fvVmaBWpQiF8CQfm%2FHT7pNZ5mVfzFQnudybVxJqnEifL4JhAbJ7gYO9jrfI%2F7CHDrXvyLL0tziou10COFkgWeh5gnLYoGjoVr14DhiJ6hx8YDa%2FZludXGtk9PMSomzO%2BfZyDqioMx7kYOdXk9ZcHLSr8IZQqKayUJMsTr%2BTXp00p5ZTkcZJM1TOlXWNBIxLjOJDaTmWuOSZEPHT01DIIo2iLkXv2W9TunQdlTHc%2B023FZWp2oCZqvpA3hi80XJzGmjlNtsDLmWnQv6w0UQzswDqVvqaHaQEbBsYKdeNWXdD1hsN3Kc4vOBbi3c802PmuHaLhCCYahovCqlZyeeNgf9V8ww%2BYCgShEeXP%2BqB6JKQu02jdkbpO%2B39HwLseQBoIuSBwjRySPoHxyYqt77Zpu8JlpZSf45SG5CVd%2B5xCRZNP%2Bb2Xovl1RQOQBmmuTc%2BQqkdapy7PQmOHZGyicl7CWqpPzLLJZUGjCDR1iN1V1LXGqD9Zz5mmxhO3yhMEaty01Qsj8y1i3xvwQZE7DCLCccb2QcvH%2FXOU9rZDALyj1sJqGG05evuEiUpL7hRnhRIH1RB3eYXKL2uc7M3GhKQkecicfj02o9VWETocB3oH70qWtCYb6j7TK5TAVRKinkpCdKw8g234LaDcOc8AZ922PENzJbpq9wuzOn5LeAD5ILgAfU%2FlzETtPUY2s93%2Bc4quSi%2BWhNkLPa9J8c0j1OfWODhcauHdrNNj1sGUQG8lrOUWxo7RqVZbjhVfilia11oWobw8%2F7fd5HVXuQGhKdBkoSqy6rMavN71V%2B96eXxJEXLA7Ni3hMm0gRVfG0m3ZjaEz1VFIl9i2xBwEsdCXm64VKaqezWRbov8qfli4spiUVxpZLaGlbp16uuLsI%2B%2BzPxDEAil47inhdEUy2tLULjayGc3LHo4Smftq98SFpA0PTTj6pnpL1%2B%2FfJ2UK%2BMucXw%2Fcv4ZLpSDlvdw7PH7zjQN%2FHqfao4UiJkwOKs607hWy0CaJ5r9diE9NHjodMdOZ410Gwg9XmwjfSkPSHltWug0zDGqJD7T8Km2nIcHF%2FHp2qXD6Z3vNVWXr5aq9cqFwmW47nayj%2BSdQejOQ8d2YOkVunNh4Hd2PBbnQxCv9m8JxUH8qBXYKEcBwGuMnd1xT1eF84lOn1Fw2rAcd2AJNy2Aj8g9LuL1G7qIuAm3WRGPmxoHP1Y%2Fia%2B91QUaWebRrC7imrcXy5JfNE43JOCsXxzL6lGd5muza1xV568ktm7n73NO3Em5%2F4YYX2vMEG5doG%2Brjd6rsRtI%2F%2B7HVXaNGKrc1njdqjVXtVS0SRkH9OOu8FrKRLFeOslCjbo8uBrN9dXxq3rJDdQ9UxdOXwWMacJVb7qcOE%2FVs347bNHmWdcWvWgsTLIUpqvS2mrzohvMsteewc7bt5arbzssyS4LS0BZgtN3Z0ZYq2iPvB%2Bfo1KTpSp8ZxHC6eLg9t78Iv%2FO1rVb5eL3kVxo33GHdqG9mVuRT6%2Blvgv2t9Hx%2Fn5tN7Quy79cYNF4CMydyvVdMBY2dl6hEJTeu3CUevyWd%2BFIGBcCz%2BKX5oB8r068xDRWnvWPbcch9HyG2hRmzN04vCEtgdLFeY%2BnN0GljWZmC%2Bc7Rxtru4MZLdWtJlyz0v0O1%2BwWoWyOSO4eb4L6YGV%2FHr6XgwSNARgPh6FXm%2B3j3XecBCb%2B8dcsIh8XvntxRFf0xCg%2F7QewjLk8oHHfbSWNpgCaY1Tl9vQVTuJ0%2F48qcsLv%2F9sH%2Bu3%2F

## 9. References

Europcar. (n.d.). *Car hire at Kigali Intl Airport (self drive)*. Retrieved November 18, 2026, from https://www.europcar.co.uk/en-gb/places/car-hire-rwanda/kigali/kigali-intl-airport-self-drive

Hasan, N. A. N., & Mat Nawi, R. (2026). Traveller-friendly vehicle rental web-based application (TWB): A web-based application to optimize business efficiency. *Applied Information Technology and Computer Science, 7*(1), 390-409. https://doi.org/10.30880/aitcs.2026.07.01.021

Kumar, K. (2020). *Cornell car rental dataset* (Version 2) [Data set]. Kaggle. https://www.kaggle.com/datasets/kushleshkumar/cornell-car-rental-dataset

National Institute of Statistics of Rwanda. (2025). *Rwanda statistical yearbook 2025*. https://statistics.gov.rw/sites/default/files/documents/2025-12/RWANDA_STATISTICAL_YEARBOOK_2025.pdf

National Institute of Statistics of Rwanda. (n.d.). *Tourism and transport*. Retrieved November 18, 2026, from https://statistics.gov.rw/statistical-publications/ict-water-sanitation-energy-tourism-and-transport/tourism-transport

Ogbiti, J. T., & Aaron, W. (2024). Development of a web-based car rental management system. *Science World Journal, 19*(3), 797-807. https://doi.org/10.4314/swj.v19i3.27

Oracle. (n.d.). Analytic functions. In *Oracle Database SQL language reference* (Release 19c). Retrieved November 18, 2026, from https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Analytic-Functions.html

Oracle. (n.d.). Data integrity. In *Oracle Database concepts* (Release 19c). Retrieved November 18, 2026, from https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/data-integrity.html

More details on each source: [docs/research/research_log.md](docs/research/research_log.md).
