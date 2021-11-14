# Task 1
Add City, State, Count, row limit to 10
Filter Facility Type is equal to HELIPORT, descending order for count

Add Count, State, pivot Facility Type, row limit to 10, descending order for count

Add Origin City, Origin State, Flight Count, Cancelled Count
Filter Flight Count > 10000
Custom Table Calculation ${flights.cancelled_count}/${flights.count} Percent with 3 decimals, in descending order

# Task 2
Add Aircraft Origin City, Aircraft Origin State, Aircraft Origin Code, and Flights Count from Flights, row limit to 10
Add State, City, and Code from Airports
Filter Control Tower, Is Major, and Joint Use to be Yes for all
