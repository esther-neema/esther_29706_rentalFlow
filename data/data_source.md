# Data Source

**Project:** Car rental business in Rwanda
**Date checked:** 18 November 2026

## 1. Chosen dataset

| Item | Detail |
|------|--------|
| Name | Cornell Car Rental Dataset (Version 2) |
| Creator | Kushlesh Kumar |
| Where | Kaggle: https://www.kaggle.com/datasets/kushleshkumar/cornell-car-rental-dataset |
| Collected | July 2020, from car rental listings in major US cities. The creator scraped the web and built the dataset himself. |
| Licence | Creative Commons Attribution 4.0 (CC BY 4.0) |
| Size | 5,851 records (one row per rental car), 15 columns |
| Our copy | [data/raw/car_rental_cars.csv](raw/car_rental_cars.csv) (5,851 rows, 13 columns, with personal-risk columns removed, see section 3) |

**APA citation:**
Kumar, K. (2020). *Cornell car rental dataset* (Version 2) [Data set]. Kaggle. https://www.kaggle.com/datasets/kushleshkumar/cornell-car-rental-dataset

## 2. Why this dataset fits our problem

Our system must show **which cars people rent most** and **how much money the business earns**. This dataset has real data for both:

- `renterTripsTaken` is the number of times each car was rented. In total the data has 195,876 rentals.
- `rate.daily` is the price per day for each car (from 20 to 1,500 US dollars, middle value 69).
- It also describes each car: make, model, type, year, fuel type, rating, and number of reviews.

Some real facts we can already see:

- Most common makes: Tesla (598 cars), Toyota (591), BMW (456), Ford (436), Chevrolet (431).
- Most rented models (total rentals): Tesla Model 3 (10,187), Ford Mustang (5,940), Toyota Corolla (5,862), Jeep Wrangler (4,979).
- Car types: car 3,659, SUV 1,714, minivan 232, truck 191, van 55.
- Fuel: gasoline 4,810, electric 622, hybrid 274, diesel 70, missing 75.

## 3. Checklist against the requirements

| Requirement | Result | Evidence |
|-------------|--------|----------|
| A few hundred records or more | Yes | 5,851 records |
| Licence allows educational use | Yes | CC BY 4.0 allows anyone to use, share, and change the data for any purpose, if we give credit to the creator. We give credit in section 1. |
| No personal information that identifies real people | Yes, after cleaning | The data has no names, phone numbers, or emails. But it had three columns that could point to a real car owner: `owner.id` (the owner's ID on the rental website) and `location.latitude` / `location.longitude` (the exact place where the car is parked, which may be the owner's home). **We removed these three columns** in our copy. We added `car_no` (1 to 5,851) as a simple row number. |

## 4. Columns in our copy

| Column | Meaning | Example |
|--------|---------|---------|
| car_no | Row number we added | 1 |
| fuelType | Fuel type | ELECTRIC |
| rating | Average customer rating (out of 5) | 5.0 |
| renterTripsTaken | Number of times the car was rented | 13 |
| reviewCount | Number of reviews | 12 |
| location.city | City | Seattle |
| location.country | Country | US |
| location.state | US state | WA |
| rate.daily | Price per day (US dollars) | 135 |
| vehicle.make | Make | Tesla |
| vehicle.model | Model | Model X |
| vehicle.type | Type | suv |
| vehicle.year | Year made | 2019 |

**Data quality notes (to fix in the cleaning step):**

- `rating` is missing for 501 cars (cars with no reviews yet).
- `fuelType` is missing for 75 cars.
- No duplicate rows.
- Car years range from 1955 to 2020, so a few very old cars may be outliers.

## 5. How the data maps to our system

| Our table | Source |
|-----------|--------|
| CARS | From the dataset: make, model, type, year, fuel type, daily rate. |
| BOOKINGS | The dataset gives the real number of rentals per car (`renterTripsTaken`). We will create booking rows so that each car has the same number of bookings as in the real data. This keeps the real pattern of which cars are rented most. |
| PAYMENTS | Calculated from each booking: rental time multiplied by the car's real daily rate. |
| CUSTOMERS | Made-up sample customers. This is on purpose: real customer data would break the "no personal information" rule. |

**Changes for Rwanda:**

- Prices are in US dollars. We will convert them to Rwandan francs (RWF) using the National Bank of Rwanda (BNR) exchange rate on the day we load the data, and write that rate in our notes.
- The US city and state columns are not needed for a business in Rwanda. We can replace them with Rwandan branch locations (for example Kigali or Musanze), or drop them.
- Hourly and weekly prices are not in the data. We will set them from the daily rate (for example, the hourly price = daily rate / 8), and say so clearly in the report.

## 6. Other datasets we checked and did not choose

| Dataset | Why not chosen |
|---------|----------------|
| SAR Rental / YourCabs bookings (Kaggle: erigor/sar-rental-car-rental-service, 10,000 bookings, listed as MIT) | It has real booking records, including hourly rental packages (3 to 12 hours). But the page says San Francisco, while the coordinates in the data are in Bangalore, India. It is a re-upload of the YourCabs data, and the original licence is not known. Because the licence chain is unclear, we did not use it. |
| Car Rental Demand, Bangalore (Kaggle: ashwinshetgaonkar/analytics-vidhya-hackathon-april-2022, CC0) | Only hourly demand counts, with no car details. The uploader says the data is owned by Analytics Vidhya, so the CC0 licence may not be the uploader's to give. |
| Munich shared mobility trips (Zenodo, ODbL, ShareNow: 1,727 cars, 1,285,865 trips) | Clear licence, but the "trips" are GPS movements (median about 5 minutes). They are not real bookings of hours, days, or weeks. The vehicle IDs also look like number plates. |
| ReachNow EV driving data (US Department of Energy, about 49,000 rentals) | The download is on the Livewire platform, which needs an account, and we could not confirm the licence. |
| Turo Rental Car Pricing Info (Kaggle, CC0, 8,475 cars) | One 347 MB JSON file. It is hard to use, and it gives no more than the Cornell dataset for our needs. |
| Rwanda data (NISR, HDX, TUMI Hub) | Only national totals, such as the number of registered vehicles. No Rwandan dataset has car rental records. |
