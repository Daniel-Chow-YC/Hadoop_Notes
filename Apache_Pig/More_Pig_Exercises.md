# Exercises:
There are 4 CSVs with data from the AdventureWorks database on S3 in data-eng-resources
<br />
Produce Pig Scripts to do the following:
- 1 Output the full names of the Top 5 Salespeople
- 2 Find the average salesperson's sales (year to date) per territory name. Also provide the count of salespeople per territory.
- 3 Output a file containing a list of all names of employees as initials and surname (e.g. "D. R. Harvey").

## Task 1 (Output the full names of the Top 5 Salespeople)

- Create pig script: `nano top5salespeople.pig`
- - Inside of nano type:
```
raw_salespeople = LOAD 's3://data-eng-resources/big-data/adventureworks/salespeople.csv' USING PigStorage(',') AS (BusinessEntityID: long, TerritoryID: long, SalesQuota: long, Bonus: long, CommissionPct: float, SalesYTD: float, SalesLastYear: float, rowguid: chararray, ModifiedDate: chararray);
raw_people = LOAD 's3://data-eng-resources/big-data/adventureworks/people.csv' USING PigStorage(',') AS (BusinessEntityID: long, PersonType: chararray, NameStyle: chararray, Title: chararray, FirstName: chararray, MiddleName: chararray, LastName: chararray, Suffix: chararray, EmailPromotion:int, AdditionalContactInfo: chararray, Demographic: chararray, rowguid: chararray, ModifiedDate: chararray);
sales_ytd = FOREACH raw_salespeople GENERATE BusinessEntityID, SalesYTD;
names = FOREACH raw_people GENERATE BusinessEntityID, CONCAT(FirstName, '_' ,MiddleName, '_', LastName) AS Fullname;
names_with_sales = JOIN sales_ytd BY BusinessEntityID, names BY BusinessEntityID;
ordered_names_with_sales = ORDER names_with_sales BY SalesYTD DESC;
top_5_sales_people_data = LIMIT ordered_names_with_sales 5;
top_5_sales_people = FOREACH top_5_sales_people_data GENERATE Fullname;
top_5_sales_people_clean = FOREACH top_5_sales_people GENERATE REPLACE(Fullname,'_NULL','') AS Fullname;
STORE top_5_sales_people_clean INTO 'top_5_sales_people';
```
- Run script in hadoop (not in grunt): `pig -x mapreduce top5salespeople.pig`
- File is stored in hdfs: `hdfs dfs -ls top_5_sales_people`
- Move data to local node: `hdfs dfs -cat top_5_sales_people/part-r-* > top_5_sales_people.txt`
- Check Output: `cat top_5_sales_people.txt`
```
Linda_C_Mitchell
Jae_B_Pak
Michael_G_Blythe
Jillian_Carson
Ranjit_R_Varkey Chudukatil
```

## Task 2 - (Find the average salesperson's sales (year to date) per territory name. Also provide the count of salespeople per territory.)
- Create pig script: `nano territories.pig`
- Inside of nano type:
```
raw_salespeople = LOAD 's3://data-eng-resources/big-data/adventureworks/salespeople.csv' USING PigStorage(',') AS (BusinessEntityID: long, TerritoryID: long, SalesQuota: long, Bonus: long, CommissionPct: float, SalesYTD: float, SalesLastYear: float, rowguid: chararray, ModifiedDate: chararray);
raw_territories = LOAD 's3://data-eng-resources/big-data/adventureworks/territories.csv' using PigStorage(',') AS (TerritoryID: long, Name: chararray, Group: chararray, SalesYTD: long, SalesLastYear: long, CostYTD: long, CostLastYear: long, rowguid: chararray, ModifiedDate: chararray);
salespeople = FOREACH raw_salespeople GENERATE BusinessEntityID, TerritoryID, SalesYTD;
territories = FOREACH raw_territories GENERATE TerritoryID, Name;
salespeople_territories_data = JOIN salespeople BY TerritoryID, territories BY TerritoryID;
salespeople_territories = FOREACH salespeople_territories_data GENERATE BusinessEntityID, SalesYTD, Name AS TerritoryName;
territory_groups = GROUP salespeople_territories BY TerritoryName;
territory_groups_info = FOREACH territory_groups GENERATE group AS TerritoryName, COUNT(salespeople_territories.BusinessEntityID) as Count_of_SalesPeople, AVG(salespeople_territories.SalesYTD) AS AvgSales;
STORE territory_groups_info INTO 'territory_info';
```
- Run script: `pig -x mapreduce territory.pig`
- File is stored in hdfs: `hdfs dfs -ls territory_info`
- Move data to local node: `hdfs dfs -cat territory_info/part-r-* > territory_info_output.txt`
- Check Output: `cat territory_info_output.txt`
```
Canada,2,2029130.125
France,1,3121616.25
Central,1,3189418.25
Germany,1,1827066.75
Australia,1,1421810.875
Northeast,1,3763178.25
Northwest,3,1500717.4583333333
Southeast,1,2315185.5
Southwest,2,3354952.0
United Kingdom,1,4116871.25
```

## Task 3 - (Output a file containing a list of all names of employees as initials and surname (e.g. "D. R. Harvey").)
- Create pig script: `nano employeenames.pig`
```
raw_people = LOAD 's3://data-eng-resources/big-data/adventureworks/people.csv' USING PigStorage(',') AS (BusinessEntityID: long, PersonType: chararray, NameStyle: chararray, Title: chararray, FirstName: chararray, MiddleName: chararray, LastName: chararray, Suffix: chararray, EmailPromotion:int, AdditionalContactInfo: chararray, Demographic: chararray, rowguid: chararray, ModifiedDate: chararray);
raw_employees = LOAD 's3://data-eng-resources/big-data/adventureworks/employees.csv' USING PigStorage(',') AS (BusinessEntityID: long, NationalIDNumber: double, LoginID: chararray, OrganizationNode: chararray, OrganizationLevel: int, JobTitle: chararray, BirthDate: chararray, MaritalStatus: chararray, Gender: chararray, HireDate: chararray, SalariedFlag: int, VacationHours: int, SickLeaveHours: int, CurrentFlag: int, rowguid: chararray, ModifiedDate: chararray);
employees = FOREACH raw_employees GENERATE BusinessEntityID;
people = FOREACH raw_people GENERATE BusinessEntityID, SUBSTRING(REPLACE(FirstName,'NULL',''),0,1) AS FirstInitial, SUBSTRING(REPLACE(MiddleName,'NULL',''),0,1) AS MiddleInitial, REPLACE(LastName,'NULL','') AS LastName;
employees_people = JOIN employees BY BusinessEntityID, people BY BusinessEntityID;
employee_names = FOREACH employees_people GENERATE CONCAT(FirstInitial, '. ' ,MiddleInitial, '. ', LastName) AS Name;
employee_names_clean = FOREACH employee_names GENERATE REPLACE(Name,' . ',' ') AS Name;
STORE employee_names_clean INTO 'employee_names';
```
- Run script: `pig -x mapreduce employeenames.pig`
- File is stored in hdfs: `hdfs dfs -ls employee_names`
- Move data to local node: `hdfs dfs -cat employee_names/part-r-* > employee_names_output.txt`
- Check Output: `cat employee_names_output.txt`
```
K. J. Sánchez
T. L. Duffy
R. Tamburello
R. Walters
G. A. Erickson
J. H. Goldberg
D. A. Miller
D. L. Margheim
G. N. Matthew
M. Raheem
O. V. Cracium
T. B. D'Hers
J. M. Galvin
M. I. Sullivan
S. B. Salavaria
D. M. Bradley
K. F. Brown
J. L. Wood
M. A. Dempsey
W. M. Benshoof
T. J. Eminhizer
S. E. Harnpadoungsataya
M. E. Gibson
J. A. Williams
J. R. Hamilton
P. J. Krebs
J. A. Brown
G. R. Gilbert
M. K. McArthur
B. L. Simon
M. W. Shoop
R. A. Laszlo
A. O. Stahl
S. O. Mohan
B. G. Heidepriem
J. R. Lugo
C. O. Okelberry
K. B. Abercrombie
E. R. Dudenhoefer
J. M. Dobney
B. Baker
J. D. Kramer
N. A. Anderson
S. D. Rapier
T. R. Michaels
E. O. Kogan
A. R. Hill
R. A. Ellerbrock
B. K. Johnson
S. M. Higa
J. L. Ford
D. M. Hartwig
D. R. Glimp
B. N. Kearney
T. R. Maxwell
D. H. Smith
F. T. Miller
K. C. Keil
B. N. Hohman
P. C. Male
D. H. Tibbott
J. T. Campbell
M. W. Dusza
M. J. Zwilling
R. T. Reeves
K. R. Khanna
J. G. Adams
C. B. Fitzgerald
S. F. Masters
D. J. Ortiz
M. S. Ray
S. T. Selikoff
C. M. Poland
B. M. Rettig
M. F. Osada
C. M. Philips
M. A. Netz
R. H. D'sa
E. L. Brown
S. P. Kaliyath
M. U. Frintu
J. T. Creasey
P. M. Cook
F. R. Martinez
B. R. Goldstein
R. L. Cornelsen
C. K. Petculescu
B. A. Stadick
P. C. Wedge
D. C. Tiedt
K. B. Zimmerman
T. M. Vande Velde
K. T. Loh
R. Hunter
J. H. Scardelis
E. I. Keyser
M. H. Samant
S. A. Tejani
N. Yu
L. B. Song
H. N. Pournasseh
Z. W. Mu
E. N. Ersan
M. R. Baker
K. M. Homer
J. T. Kane
C. E. Hill
J. K. Liu
A. O. Ciccu
J. T. Cao
S. R. Fatima
J. P. Evans
L. K. Moschell
M. J. Krapauskas
A. W. Barbariol
M. W. Patten
C. W. Niswonger
D. L. Hall
M. T. Entin
K. H. Lertpiriyasuwat
P. G. Ackerman
S. W. Eaton
V. N. Kuppa
K. T. Ralls
M. T. Berndt
J. T. Bischoff
D. P. Hamilton
P. B. Komosinski
G. W. Yukish
R. T. Caron
B. F. Cetinok
N. B. Holliday
M. L. Rothkugel
E. Gubbels
I. W. Salmre
S. A. Valdez
A. T. Sousa
S. H. Smith
H. T. Ting
P. E. Samarawickrama
M. G. Su
O. C. Turner
K. Sunkammurali
P. R. Singh
C. S. Randall
J. Wang
S. Reátegui Alayo
J. M. Watters
A. M. Ruth
M. T. Vanderhyde
R. E. Shabalin
Y. L. Li
H. P. Feng
R. K. Sam
F. K. Fakhouri
L. M. Sacksteder
L. A. Randall
S. N. Dyck
T. W. Earls
J. V. Hay
K. J. Koenigsbauer
L. C. Steele
A. M. Nayberg
A. M. Cencini
C. T. Preston
J. S. Richins
D. N. Johnson
G. R. Young
S. A. Metters
G. Z. Li
D. A. Yalovsky
M. J. Ingle
E. R. Zabokritski
B. R. Martin
R. T. Koch
D. O. Lawrence
R. M. King
J. N. Frum
J. S. Miksovsky
K. L. McAskill-White
M. T. Hines
N. S. Mirchandani
B. S. Decker
J. Y. Chen
S. A. Hesse
S. S. Kim
Y. S. McKay
D. B. Hite
J. M. Esteves
R. J. Rounthwaite
L. C. Penuchot
B. M. Diaz
A. E. McGuel
F. T. Northup
K. H. Liu
S. G. Mohamed
R. M. Patel
L. O. Nay
P. R. Nartker
F. T. Lee
B. T. Lloyd
T. G. Nusbaum
K. L. Myer
G. B. Mares
L. A. Kane
S. V. Munson
G. F. Alderson
S. R. Gode
K. E. Flood
B. M. Newman
H. E. Abolrous
P. J. Wu
S. T. Charncherngkha
A. T. Berglund
M. L. Harrington
S. P. Alexander
Z. T. Arifin
T. N. Kharatishvili
S. N. Chai
K. R. Berge
C. K. Norred
A. Wright
S. L. Uddin
W. S. Vong
A. J. Brewer
B. P. LaMee
G. E. Altman
C. E. Kleinerman
L. K. Penor
S. J. Macrae
J. L. Berry
P. H. Coleman
M. E. Hedlund
L. F. Norman
P. M. Barreto de Mattos
G. N. Culbertson
H. O. Chen
V. X. Luthra
M. C. Martin
W. T. Johnson
D. J. Liu
D. E. Poe
C. L. Spoon
B. A. Walton
B. C. Moreland
D. K. Tomic
J. L. Sheperdigian
M. K. Seamans
W. B. Kahn
S. H. Word
M. Q. Sandberg
A. B. Rao
L. P. Meisner
F. J. Ogisu
G. L. Hee
F. S. Pellow
E. S. Kurjan
E. M. Hagens
B. T. Miller
A. L. Hill
R. N. Hillmann
D. M. Barber
J. E. Trenary
S. A. Conroy
A. R. Sharma
P. I. Connelly
K. A. Berg
R. V. Meyyappan
D. K. Bacon
F. P. Ajenstat
D. B. Wilson
J. B. Bueno
B. S. Welcker
S. Y. Jiang
M. G. Blythe
L. C. Mitchell
J. Carson
G. R. Vargas
T. M. Reiter
P. O. Ansman-Wolfe
S. K. Ito
J. E. Saraiva
D. R. Campbell
T. A. Mensa-Annan
S. E. Abbas
L. N. Tsoflias
A. E. Alberts
R. B. Valdez
J. B. Pak
R. R. Varkey Chudukatil
```
