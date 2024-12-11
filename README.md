# Data Normalization and Entity-Relationship Diagramming

## Normalization

The original table only adheres to the first normal form, and describes the composite primary key of ```assignment_id``` and ```student_id```. Here is the original table, pasted from the [instructions.md](instructions.md) file:

| assignment_id | student_id | due_date | professor | assignment_topic                | classroom | grade | relevant_reading    | professor_email   |
| :------------ | :--------- | :------- | :-------- | :------------------------------ | :-------- | :---- | :------------------ | :---------------- |
| 1             | 1          | 23.02.21 | Melvin    | Data normalization              | WWH 101   | 80    | Deumlich Chapter 3  | l.melvin@foo.edu  |
| 2             | 7          | 18.11.21 | Logston   | Single table queries            | 60FA 314  | 25    | Dümmlers Chapter 11 | e.logston@foo.edu |
| 1             | 4          | 23.02.21 | Melvin    | Data normalization              | WWH 101   | 75    | Deumlich Chapter 3  | l.melvin@foo.edu  |
| 5             | 2          | 05.05.21 | Logston   | Python and pandas               | 60FA 314  | 92    | Dümmlers Chapter 14 | e.logston@foo.edu |
| 4             | 2          | 04.07.21 | Nevarez   | Spreadsheet aggregate functions | WWH 201   | 65    | Zehnder Page 87     | i.nevarez@foo.edu |
| ...           | ...        | ...      | ...       | ...                             | ...       | ...   | ...                 | ...               |

<br>

### 1) Normalizing to 2NF
To work towards making the table compliant with fourth normal form, I began by normalizing it to the second normal form. Firstly, the fields ```due_date```, ```assignment_topic```, and ```relevant_reading``` are facts about the assignment, meaning they only partially describe the entity the table is about. Thus, I split the data into the following tables, ```assignments``` and ```submissions```, respectively: 

<br>

```assignments:```
| id | due_date | assignment_topic                | relevant_reading    |
|----|----------|---------------------------------|---------------------|
| 1  | 23.02.21 | Data normalization              | Deumlich Chapter 3  |
| 2  | 18.11.21 | Single table queries            | Dümmlers Chapter 11 |
| 3  | 18.04.21 | Data munging                    | Sun Tzu Chapter 4   |
| 4  | 04.07.21 | Spreadsheet aggregate functions | Zehnder Page 87     |
| 5  | 05.05.21 | Python and pandas               | Dümmlers Chapter 14 |

<br>

```submissions:```
| student_id | assignment_id | professor | classroom | grade | professor_email   |
|------------|---------------|-----------|-----------|-------|-------------------|
| 1          | 3             | Melvin    | WWH 101   | 70    | l.melvin@foo.edu  |
| 2          | 5             | Logston   | 60FA 314  | 85    | e.logston@foo.edu |
| 3          | 2             | Nevarez   | WWH 201   | 60    | i.nevarez@foo.edu |
| 4          | 1             | Melvin    | 60FA 314  | 95    | l.melvin@foo.edu  |
| 5          | 4             | Logston   | WWH 201   | 90    | e.logston@foo.edu |

The tables are now normalized to the 2NF, as the fields in the ```assignments``` table fully describe the assignments; and the fields in the ```submissions``` table, which still describe the composite primary key of ```student_id``` and ```assignment_id```, are fully related to the combined entity. Some of the values in these tables are made up, as they are not relevant to this exercise. 

<br>

### 2) Normalizing to 3NF
While the ```assignments``` table adheres to the third normal form, there is a clear issue with the ```submissions``` table as the field ```professor_email``` is a fact about ```professor```, which is another non-key field. Thus, we must split the table into ```students``` and ```professors```.

<br>

```submissions:```
| student_id | assignment_id | professor_id | classroom | grade |
|------------|---------------|--------------|-----------|-------|
| 1          | 3             | 1            | WWH 101   | 70    |
| 2          | 5             | 2            | 60FA 314  | 85    |
| 3          | 2             | 3            | WWH 201   | 60    |
| 4          | 1             | 1            | 60FA 314  | 95    |
| 5          | 4             | 2            | WWH 201   | 90    |

<br>

```professors:```
| id | professor | professor_email   |
|----|-----------|-------------------|
| 1  | Melvin    | l.melvin@foo.edu  |
| 2  | Logston   | e.logston@foo.edu |
| 3  | Nevarez   | i.nevarez@foo.edu |


<br>

### 3) Normalizing to 4NF (final answers)
Firstly, I normalized the ```assignments``` record to the fourth normal form. In its current form, it contains more than one independent multi-valued fact about the assignment. Since we know that a specific assignment can have different due dates as well as relevant readings depending on the professor and class section, the current schema can be problematic for maintaining the database. I split the record into three tables: ```assignments```, ```due dates```, and ```relevant readings```.

<br>

```assignments:```
| id | assignment_topic                |
|----|---------------------------------|
| 1  | Data normalization              |
| 2  | Single table queries            |
| 3  | Data Munging                    |
| 4  | Spreadsheet aggregate functions |
| 5  | Python and pandas               |

<br>

```due dates```:
| assignment_id | due_date |
|---------------|----------|
| 1             | 23.02.21 |
| 4             | 04.07.21 |
| 3             | 18.04.21 |
| 3             | 17.04.21 |
| 1             | 26.02.21 |

<br>

```relevant readings```:
| assignment_id | relevant_reading   |
|---------------|--------------------|
| 1             | Deumlich Chapter 3 |
| 4             | Zehnder Page 87    |
| 3             | Sun Tzu Chapter 4  |
| 3             | Aurelius Page 80   |
| 1             | Zehnder Page 18    |

<br>

For the ```submissions``` record, there could potentially be ambiguity if there are null values in the ```grades``` field, or if for some reason a student submits an assignment more than once, with the current schema. Thus, I split the table into ```submissions``` and ```grades```. I decided to keep the composite primary key of ```student_id``` and ```assignment_id``` for the record as none of the fields are unique to a specific student or assignment, but rather the combined entity of both.

<br>

```submissions```:
| student_id | assignment_id | professor_id | classroom |
|------------|---------------|--------------|-----------|
| 1          | 3             | 1            | WWH 101   |
| 2          | 5             | 2            | 60FA 314  |
| 3          | 2             | 3            | WWH 201   |
| 4          | 1             | 1            | 60FA 314  |
| 5          | 4             | 2            | WWH201    |

<br>

```grades```:
| student_id | assignment_id | grades |
|------------|---------------|--------|
| 1          | 4             | 65     |
| 4          | 5             | 90     |
| 2          | 2             | 75     |
| 5          | 1             | 80     |
| 3          | 4             | 85     |

<br>

For the ```professors``` record - assuming that each professor has one unique university email - the current format satsifies the fourth normal form.

<br>

```professors:```
| id | professor | professor_email   |
|----|-----------|-------------------|
| 1  | Melvin    | l.melvin@foo.edu  |
| 2  | Logston   | e.logston@foo.edu |
| 3  | Nevarez   | i.nevarez@foo.edu |

<br>

## ER Diagram 

Below is my ER diagram for the database. Regarding the cardinalities between the three entities, the relationship between ```assignment``` and ```submission``` is one-to-many, as each assignment can have many submissions, but each submission is only one completed assignment. ```professor``` to ```assignment``` is many-to-many, as each professor assigns multiple assignments and each assignment can be assigned by more than one professor. Lastly, there is a one-to-many relationship between ```professor``` and ```submission```, as one professor grades multiple submissions but one submission can only be graded by one professor.

![ER diagram](/images/database-design-assn.drawio.svg)

