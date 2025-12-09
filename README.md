# Sqlite_Depositor
A little bash script to dump csv files into a sqlite database. Inspired because the FBI'S NIBRS stopped providing the SQL commands in the README

# Usage

```
loadsqlite <directory_with_csv_files> <Name_of_output_database_file>
```

## For Example:

```
loadsqlite Texas_data/ nibrs_texas.db
```

## Some Sample Queries FROM THE FBI'S README FILE

Found at: https://cde.ucr.cjis.gov/LATEST/webapp/#/pages/downloads

The best way to understand how NIBRS relates is to look at some
example queries. Because the NIBRS database is in fifth-normal form,
it is often necessary to join in multiple tables just to resolve
specific codes for things like race or weapon type. In general, almost
every one of the data tables has a foreign key linking it to a
nibrs_incident which you can use to find all the
offenses/offenders/victims/property related together in an incident
(but you do need to look at other tables to relate victims directly to
offenders or offenses). Let's get started with some examples.

To get all the incidents in West Virginia that happened in 2015, you can run the following query:

``` sql
SELECT ni.*
FROM nibrs_incident ni
JOIN nibrs_month nm ON nm.nibrs_month_id = ni.nibrs_month_id
JOIN agencies c ON c.agency_id = nm.agency_id
WHERE c.state_abbr = 'WV'
AND nm.data_year = 2014;
```

To get all homicide offenses in West Virginia in 2015

``` sql
SELECT o.*
FROM nibrs_offense o
JOIN nibrs_incident ni ON o.incident_id = ni.incident_id
JOIN nibrs_month nm ON nm.nibrs_month_id = ni.nibrs_month_id
JOIN agencies c ON c.agency_id = nm.agency_id
JOIN nibrs_offense_type ot ON ot.offense_type_id = o.offense_type_id
WHERE c.state_abbr = 'WV'
AND nm.data_year = 2014
AND ot.offense_code = '09A';
```

To get information about homicide victims in West Virginia in 2015 you
will need to use the victim_offense table and also join in some of the
lookup tables.

``` sql
SELECT r.race_code, a.age_code, v.age_num, e.ethnicity_code
FROM nibrs_victim v
JOIN nibrs_victim_offense vo ON vo.victim_id = v.victim_id
JOIN nibrs_offense o ON o.offense_id = vo.offense_id
JOIN nibrs_incident ni ON ni.incident_id = v.incident_id
JOIN nibrs_month nm ON nm.nibrs_month_id = ni.nibrs_month_id
JOIN ref_race r ON r.race_id = v.race_id
JOIN nibrs_age a ON a.age_id = v.age_id
JOIN nibrs_ethnicity e ON e.ethnicity_id = v.ethnicity_id
JOIN agencies c ON c.agency_id = nm.agency_id
JOIN nibrs_offense_type ot ON ot.offense_type_id = o.offense_type_id
WHERE c.state_abbr = 'WV'
AND nm.data_year = 2014
AND ot.offense_code = '09A';
```

To see a breakdown of where robberies happened in West Virginia in 2014

``` sql
SELECT location_code, location_name, count(*)
FROM nibrs_offense o
JOIN nibrs_incident ni ON ni.incident_id = o.incident_id
JOIN nibrs_month nm ON nm.nibrs_month_id = ni.nibrs_month_id
JOIN nibrs_offense_type ot ON ot.offense_type_id = o.offense_type_id
JOIN nibrs_location_type l ON l.location_id = o.location_id
JOIN agencies c ON c.agency_id = nm.agency_id
WHERE c.state_abbr = 'WV'
AND nm.data_year = 2014
AND ot.offense_code = '120'
GROUP by location_code, location_name
ORDER by location_code;
