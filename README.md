# sql-murder

> https://mystery.knightlab.com/

_A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a **​murder​** that occurred sometime on ​**Jan.15, 2018​** and that it took place in **​SQL City​**. Start by retrieving the corresponding **crime scene report** from the police department’s database._

[Schema](https://mystery.knightlab.com/schema.png)

![https://mystery.knightlab.com/schema.png](assets/img/schema.png)

> **Knowns**
>
> - Crime: murder
> - Date: Jan.15, 2018
> - City: SQL City
> 
> **Unknowns**
> 
> 1. Crime scene report
> 2. Person(s) of interest

Explore the `crime_scene_report` table.

```sql
SELECT * 
FROM crime_scene_report
ORDER BY date
LIMIT 10
```

date| type | description | city
----| ---- | ----------- | ----
20170101 | fraud | forgotten to ask.’ | Long Beach
20170101 | fraud | Alice did not wish to offend the Dormouse again, so she began very | Oceanside
20170101 | theft | ‘Yes,’ said Alice, ‘we learned French and music.’ | Chicago
20170101 | smuggling | ‘Now tell me, Pat, what’s that in the window?’ | Gilbert
20170101 | assault | twist it up into a sort of knot, and then keep tight hold of its right | Nashville
20170102 | arson | with a smile. There was a dead silence. | Las Vegas
20170102 | bribery | glass table. ‘Now, I’ll manage better this time,’ she said to herself, | Riverside
20170102 | theft | days and days.’ | Canton
20170102 | smuggling | ‘Yes,’ said Alice, ‘we learned French and music.’ | Alexandria
20170103 | arson | witness.’ And he added in an undertone to the Queen, ‘Really, my dear, | Rochester

Explore the **murders** on **Jan.15, 2018** in **SQL City**

```sql
SELECT *
FROM crime_scene_report
WHERE 
    type = 'murder'
    AND date = 20180115
    AND LOWER(city) = 'sql city'
```

date | type | description | city
---- | ---- | ----------- | ----
20180115 | murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". | SQL City

> **Knowns**
>
> - `crime_scene_report.date=20180115`
> - `crime_scene_report.city='SQL City'`
> - Witnesses:
>   1. **Last house** of **Northwestern Dr**
>   2. **Annabel**, somewhere on **Franklin Ave**
> 
> Unknowns
> 1. Witness identities

Explore the `person` table:

```sql
SELECT *
FROM person
LIMIT 10
```

id | name | license_id | address_number | address_street_name | ssn
-- | ---- | ---------- | -------------- | ------------------- | ---
10000 | Christoper Peteuil | 993845 | 624 | Bankhall Ave | 747714076
10007 | Kourtney Calderwood | 861794 | 2791 | Gustavus Blvd | 477972044
10010 | Muoi Cary | 385336 | 741 | Northwestern Dr | 828638512
10016 | Era Moselle | 431897 | 1987 | Wood Glade St | 614621061
10025 | Trena Hornby | 550890 | 276 | Daws Hill Way | 223877684
10027 | Antione Godbolt | 439509 | 2431 | Zelham Dr | 491650087
10034 | Kyra Buen | 920494 | 1873 | Sleigh Dr | 332497972
10039 | Francesco Agundez | 278151 | 736 | Buswell Dr | 861079251
10095 | Leslie Thate | 729987 | 2772 | Camellia Park Circle | 127944356
10122 | Alva Conkel | 779002 | 116 | Diversey Circle | 148521773

Search for witness #1

```sql
SELECT *
FROM person
WHERE LOWER(address_street_name) = 'northwestern dr'
ORDER BY address_number DESC
LIMIT 1
```

id | name | license_id | address_number | address_street_name | ssn
-- | ---- | ---------- | -------------- | ------------------- | ---
14887 | Morty Schapiro | 118009 | 4919 | Northwestern Dr | 111564949

Search for witness #2:

```sql
SELECT * FROM person
WHERE 
    LOWER(name) LIKE 'annabel%'
    AND LOWER(address_street_name) = 'franklin ave'
```

id | name | license_id | address_number | address_street_name | ssn
-- | ---- | ---------- | -------------- | ------------------- | ---
16371 | Annabel Miller | 490173 | 103 | Franklin Ave | 318771143

> **Knowns**
>
> - `crime_scene_report.date=20180115`
> - `crime_scene_report.city='SQL City'`
> - Witness 1: `person.id = 14887`
> - Witness 2: `person.id = 16371`
> 
> **Unknowns**
> 1. Witness interviews

Witness interviews:
```sql
SELECT 
    p.name,
    i.transcript
FROM 
    person p
    LEFT JOIN interview i ON p.id = i.person_id
WHERE p.id IN (14887, 16371)
```

name | transcript
---- | ----------
Morty Schapiro | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".
Annabel Miller | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.

> **Knowns**
>
> - `crime_scene_report.date=20180115`
> - `crime_scene_report.city='SQL City'`
> - Witness 1: `person.id = 14887`
> - Witness 2: `person.id = 16371`
> - Observations:
>   - Gym membership number starting with '48Z'
>   - Gold status membership bag
>   - Attended gym Jan 9th 2018
>   - Car registration plate included 'H42W'
> 
> **Unknowns**
> 1. Person(s) of interest matching gym membership
> 2. Car registrations matching 'H42W'

Search by membership prefix:

```sql
SELECT *
FROM get_fit_now_member
WHERE 
    id LIKE '48Z%'
    AND membership_status = 'gold'
```

id | person_id | name | membership_start_date | membership_status
-- | --------- | ---- | --------------------- | -----------------
48Z7A | 28819 | Joe Germuska | 20160305 | gold
48Z55 | 67318 | Jeremy Bowers | 20160101 | gold

Search by gym attendance on Jan 9th:

```sql
SELECT 
    a.check_in_date,
    a.check_in_time,
    a.check_out_time,
    b.name,
    b.person_id,
    a.membership_id
FROM 
    get_fit_now_check_in a
    LEFT JOIN get_fit_now_member b ON (a.membership_id = b.id)
WHERE 
    a.check_in_date = 20180109
    AND a.membership_id LIKE '48Z%'
```

check_in_date | check_in_time | check_out_time | name | person_id | membership_id
------------- | ------------- | -------------- | ---- | --------- | -------------
20180109 | 1600 | 1730 | Joe Germuska | 28819 | 48Z7A
20180109 | 1530 | 1700 | Jeremy Bowers | 67318 | 48Z55

> **Knowns**
>
> - `crime_scene_report.date=20180115`
> - `crime_scene_report.city='SQL City'`
> - Witness 1: `person.id = 14887`
> - Witness 2: `person.id = 16371`
> - Observations:
>   1. Gym membership number starting with '48Z'
>   2. Gold status membership bag
>   3. Attended gym Jan 9th 2018
>   4. Car registration plate included 'H42W'
> 
> **Persons of interest:**
> - Joe Germuska (`person.id=28819`)
>   - Matching obs-1
>   - Matching obs-2
>   - Matching obs-3
> - Jeremy Bowers (`person.id=67318`)
>   - Matching obs-1
>   - Matching obs-2
>   - Matching obs-3
>
> **Unknowns**
> 1. Car registrations matching 'H42W'
> 2. Interview of persons of interest

Search by car registrations:

```sql
SELECT 
    p.*,
    dl.*
FROM 
    drivers_license dl 
    LEFT JOIN person p ON dl.id = p.license_id
WHERE 
    plate_number LIKE '%H42W%'
```

id | name | license_id | address_number | address_street_name | ssn | id | age | height | eye_color | hair_color | gender | plate_number | car_make | car_model
-- | ---- | ---------- | -------------- | ------------------- | --- | -- | --- | ------ | --------- | ---------- | ------ | ------------ | -------- | ---------
78193 | Maxine Whitely | 183779 | 110 | Fisk Rd | 137882671 | 183779 | 21 | 65 | blue | blonde | female | H42W0X | Toyota | Prius
67318 | Jeremy Bowers | 423327 | 530 | Washington Pl, Apt 3A | 871539279 | 423327 | 30 | 70 | brown | brown | male | 0H42W2 | Chevrolet | Spark LS
51739 | Tushar Chandra | 664760 | 312 | Phi St | 137882671 | 664760 | 21 | 71 | black | black | male | 4H42WR | Nissan | Altima

> **Knowns**
>
> - `crime_scene_report.date=20180115`
> - `crime_scene_report.city='SQL City'`
> - Witness 1: `person.id = 14887`
> - Witness 2: `person.id = 16371`
> - Observations:
>   1. Gym membership number starting with '48Z'
>   2. Gold status membership bag
>   3. Attended gym Jan 9th 2018
>   4. Car registration plate included 'H42W'
> 
> **Persons of interest:**
> - Joe Germuska (`person.id=28819`)
>   - Matching obs-1
>   - Matching obs-2
>   - Matching obs-3
> - Jeremy Bowers (`person.id=67318`)
>   - Matching obs-1
>   - Matching obs-2
>   - Matching obs-3
>   - Matching obs-4
>
> **Unknowns**
> 2. Interview of persons of interest

Search interviews for prime suspect Jeremy Bowers:

```sql
SELECT 
    p.name,
    i.transcript
FROM 
    person p
    LEFT JOIN interview i ON p.id = i.person_id
WHERE p.id = 67318
```

name | transcript
---- | ----------
Jeremy Bowers | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.

This confirms `Jeremy Bowers` as murderer, but implicates another party.

> **Knowns**
>
> - Observations:
>   1. Woman
>   2. Large income
>   3. Height 5'5" (65") - 5'7" (67")
>   4. Red hair
>   5. Tesla Model S
>   6. Attended SQL symphony concert 3 times in December 2017

```sql
SELECT
    p.id,
    p.name,
    p.ssn,
    i.annual_income
FROM
    drivers_license dl
    LEFT JOIN person p ON dl.id = p.license_id
    LEFT JOIN income i ON p.ssn = i.ssn
WHERE 
    dl.gender = 'female'                                --obs:1
    AND dl.height >= 65 AND dl.height <= 67             --obs:3
    AND hair_color = 'red'                              --obs:4
    AND car_make = 'Tesla' AND car_model = 'Model S'    --obs:5
    AND p.id IN (                                       --obs:6
        SELECT person_id
        FROM (
            SELECT person_id, COUNT(*) AS count
            FROM facebook_event_checkin
            WHERE 
                event_name = 'SQL Symphony Concert'
                AND date >= 20171201 AND date <= 20171231
            GROUP BY person_id
        )
        WHERE count = 3
    )
```

id | name | ssn | annual_income
-- | ---- | --- | -------------
99716 | Miranda Priestly | 987756388 | 310000

Submit the prime suspect into the `solution` table:

```sql
INSERT INTO solution VALUES (1, 'Miranda Priestly');
SELECT value FROM solution;
```

| value
| :----
| Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!
