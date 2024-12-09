### Source: https://devblogs.microsoft.com/azure-sql/introducing-regular-expression-regex-support-in-azure-sql-db/

### Explanation
* REGEXP_LIKE: This function returns TRUE if a string matches a regular expression pattern, or FALSE otherwise.
* REGEXP_COUNT: This function returns the number of times a regular expression pattern matches in a string.
* REGEXP_INSTR: This function returns the starting or ending position, based on the specified option, of the given occurrence of a regular expression pattern in a string.
* REGEXP_REPLACE: This function returns a modified string replaced by a ‘replacement string’, where occurrence of the regular expression pattern found.
* REGEXP_SUBSTR: This function returns a substring that matches a regular expression pattern from a string.


### Employees Table
#### Create Employees table with some records.
| Column Name    | Data Type    |
|----------------|--------------|
| Name           | VARCHAR(150) |
| Email          | VARCHAR(320) |
| Phone_Number   | VARCHAR(20)  |

```sql
CREATE TABLE Emp_DETAILS (
    Name VARCHAR(150),
    Email VARCHAR(320),
    Phone_Number VARCHAR(20)
);
```
```sql
-- Insert some sample data
INSERT INTO Emp_DETAILS (Name, Email, Phone_Number) VALUES
    ('John Doe', 'john@contoso.com', '123-456-7890');
INSERT INTO Emp_DETAILS (Name, Email, Phone_Number) VALUES
    ('Alice Smith', 'alice@fabrikam.com', '234-567-8901');
INSERT INTO Emp_DETAILS (Name, Email, Phone_Number) VALUES
    ('Bob Johnson', 'bob@fabrikam.net','345-678-9012');
INSERT INTO Emp_DETAILS (Name, Email, Phone_Number) VALUES
    ('Eve Jones', 'eve@contoso.com', '456-789-0123');
INSERT INTO Emp_DETAILS (Name, Email, Phone_Number) VALUES
    ('Charlie Brown', 'charlie@contoso.co.in', '567-890-1234');
```

SELECT * FROM EMP_DETAILS;

#### find all the employees whose email addresses end with .com
```sql
SELECT Name, Email FROM EMP_DETAILS WHERE REGEXP_LIKE(Email, '\.com$');
```

#### for each employee, count the number of vowels in their name
```sql
SELECT Name, REGEXP_COUNT(Name, '[AEIOU]',1,'i') AS Vowel_Count FROM EMP_DETAILS;
```
#### for each employee, show their name, email, and the position of the @ sign in their email
```sql
SELECT Name, Email, REGEXP_INSTR(email, '@') AS Position_of FROM EMP_DETAILS;
```
#### format the phone numbers in the Employees table to the format (XXX) XXX-XXXX.
```sql
SELECT Phone_Number, REGEXP_REPLACE(Phone_Number, '(\d{3})-(\d{3})-(\d{4})', '(\1) \2-\3',1) AS Phone_Format FROM EMP_DETAILS;
```
#### for each employee, show the domain of their email address
```sql
SELECT Name, Email, REGEXP_SUBSTR(email, '@(.+)$', 1, 1,'c',1) AS Domain FROM EMP_DETAILS;
```
#### This query retrieves all rows from the EMP_DETAILS table where the EMAIL column contains the substring 'co', regardless of case (due to the 'i' flag for case-insensitivity).
```sql
SELECT * FROM EMP_DETAILS WHERE REGEXP_LIKE(EMAIL,'.co','i');
```
