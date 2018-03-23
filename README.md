# Database Exercise 7 (Grouping and more joins)
This repository is a exercise project for Software development (PBA) Database course. Daniel (cph-dh136)

## Assignment

### 1. List driver info with wins
##### Query:
```sql
SELECT drivers.forename, drivers.surname, drivers.driverid, count(results.rank) as Wins FROM drivers
JOIN results USING(driverid) WHERE results.rank = 1 GROUP BY driverid ORDER BY Wins DESC;
```
##### Result:
- Results limited to 5 | Query has 31 rows.

| forename | surname |	driverid |	wins
|:--------:|:-------:|:---------:|:---------:|
Kimi |	Räikkönen |	8 |	41
Lewis|	Hamilton |	1 |	38
Sebastian |	Vettel |	20 |	31
Fernando |	Alonso |	4 |	22
Michael |	Schumacher |	30 |	21

### 2. Drivers and their cars
##### Query:
```sql
SELECT drivers.driverref, constructors.name, count(*) from drivers
JOIN results USING(driverid)
JOIN constructors USING(constructorid) 
GROUP BY drivers.driverref, constructors.name 
ORDER BY count DESC;
```
##### Result:
- Results limited to 5 | Query has 2093 rows

driverref |	name |	count
---------:|:----:|:---------
michael_schumacher |	Ferrari |	181
coulthard |	McLaren |	150
massa |	Ferrari |	140
button |	McLaren |	137
rosberg |	Mercedes |	136

### 3. Exclude instaces that didn't finish
##### Query:
```sql
SELECT drivers.driverref, constructors.name, count(constructors.name) from drivers
JOIN results USING (driverid) 
JOIN constructors USING (constructorid) 
WHERE results.statusid = 1 
GROUP BY drivers.driverref, constructors.name 
ORDER BY count DESC;
```
##### Results:
- Results limited to 5 | Query has 658 rows

driverref |	name |	count
---------:|:-----:|:-------
michael_schumacher |	Ferrari |	141
massa |	Ferrari |	112
rosberg |	Mercedes |	109
webber |	Red Bull |	100
vettel |	Red Bull |	97


### 4. Aggregate pitstop time per race and driver
##### Query:
- This query has been saved as a view with the name: **pit** as follows ```CREATE VIEW pit as```
```sql
SELECT driverid, raceid, sum(milliseconds) as ms FROM pitstops GROUP BY driverid, raceid;
```
##### Results:
- Results limited to 5 | Query has 2699 rows

driverid |	raceid	| ms
--------:|:--------:|:--------
815 |	969 |	22045
8 |	942 |	59281
20 |	898 |	24086
8 |	913 |	89060
807 |	894 |	47766


### 5. Putting it together (4 inside 3)

#### Solution A.
- Depending on how you interpret the task, it can be done in 2 ways. We want to keep the results, ie. We still want to know the total pitstop time even if it is non existing. eg. we still want to know michael schumacher has driven a ferrari 141 times, even if he never had a pitstop in one.
##### Query:
```sql
SELECT drivers.driverref, constructors.name, count(*), sum(ms)  from drivers
JOIN results USING (driverid) 
FULL OUTER JOIN pit USING (driverid, raceid)
JOIN constructors USING (constructorid)
WHERE results.statusid = 1 
GROUP BY drivers.driverref, constructors.name
ORDER BY count DESC
```
##### Results:
- Results limited to 10 | Query has 658 rows

driverref |	name |	count |	sum
---------:|:------:|:----:|:---------
michael_schumacher |	Ferrari |	141 |	None
massa |	Ferrari |	112 |	2787290
rosberg |	Mercedes |	109 |	11501262
webber |	Red Bull |	100 |	3017372
vettel |	Red Bull |	97 |	4927545
raikkonen |	Ferrari |	92 |	3608658
button |	McLaren |	87 |	8515623
alonso |	Ferrari |	87 |	3691856
coulthard |	McLaren |	85 |	None
hamilton |	Mercedes |	85 |	12147994

- Note: A Outer join will generate null values in the result set, as can be seen with the 141 races where michael schumacher completed the race, but never once had a pitstop in a ferrari, according to the data. (I realy thouroughly scourered the dataset, there realy is missing data on pitstops when michael schumacher was driving a ferrari, only in a Mercedes where he drove approx 38 races in it.). When not doing a outer join, this information will get exluded or removed out. So we wont know he drove a ferrari in 141 races any longer.

#### Solution B.
- Depending on how you interpret the task, it can be done in 2 ways. We don't neccesarily want to keep the results, ie. We dont want to know the total pitstop time if it is non existing. eg. we dont want to know if michael schumacher has driven a ferrari 141 times, if he never had a pitstop in one.
##### Query: 
```sql
SELECT drivers.driverref, constructors.name, count(*), sum(ms)  from drivers
JOIN results USING (driverid) 
JOIN pit USING (driverid, raceid)
JOIN constructors USING (constructorid)
WHERE results.statusid = 1
GROUP BY drivers.driverref, constructors.name
ORDER BY count DESC
```
##### Results:
- Results limited to 5 | Query has 76 rows

driverref |	name	| count |	sum
---------:|:-------:|:----:|:--------
rosberg |	Mercedes |	95 |	11501262
hamilton |	Mercedes |	85  |	12147994
button |	McLaren |	71 |	8515623
alonso |	Ferrari |	70 |	3691856
vettel |	Red Bull |	70 |	4927545


### 6. Who has the fastest pitstop on average.
- Solution B from task 5, has been chosen in this task.
##### Query:
```sql
SELECT drivers.driverref, constructors.name, count(*), sum(ms) as TotalPitStop, ROUND(sum(ms)/count(results.raceid), 2) as AveragePit  from drivers
JOIN results USING (driverid) 
JOIN pit USING (driverid, raceid)
JOIN constructors USING (constructorid)
WHERE results.statusid = 1
GROUP BY drivers.driverref, constructors.name
ORDER BY averagePit ASC
```

##### Results:
- Results limited to 10 | Query has 76 rows

driverref |	name |	count |	totalpitstop |	averagepit
------------:|:------:|:---:|:---------:|:----------
ambrosio |	Lotus F1 |	1 |	21962 |	21962.00
rosa |	HRT |	1 |	22272 |	22272.00
hulkenberg |	Renault |	5 |	186700 |	37340.00
glock |	Marussia |	2 |	82876 |	41438.00
bruno_senna |	Williams |	13 |	574413 |	44185.62
perez |	Sauber |	15 |	681053 |	45403.53
resta |	Force  India |	34 |	1563408 |	45982.59
ricciardo |	Toro Rosso |	22 |	1015872 |	46176.00
raikkonen |	Lotus F1 |	33 |	1532336 |	46434.42
maldonado |	Williams |	21 |	982946 |	46806.95
