## Oracle PL/SQL(Procedural Language / Standard Query Language) CheatSheet 

### Contents
- Blocks
- Variables
- Constant
- Select Into
- %Type
- Conditions
- Loops
- Triggers
- Cursors
- Records
- Functions
- Stored Procedure
- Package
- Exceptions
- Collections
- Arrays

### Blocks
```sql
DECLARE
 --Declaration statements

BEGIN
 --Executable statements

Exceptions
 --Exception handling statements

END;
```

### Variables
###### Data Types
- Scalar
> Number, Date, Boolean, Character
- Large Object
> Large Text, Picture - BFILE, BLOB, CLOB, NCLOB
- Composite
> Collections, Records
- Reference

```sql
--NUMBER(precision,scale)
v_number NUMBER(5,2) := 5.01;
v_character VARCHAR2(20) := 'test';
```

### Constant
```sql
DECLARE
	v_pi CONSTANT NUMBER(7,6) := 3.141592;
BEGIN
	DBMS_OUTPUT.PUT_LINE(v_pi);
END;
```
### Select Into
```sql
DECLARE
	v_last_name VARCHAR2(20);
BEGIN
	SELECT last_name INTO v_last_name FROM persons WHERE person_id = 1;
	DBMS_OUTPUT.PUT_LINE('Last name: ' || v_last_name);
END;
```

### %Type
```sql
DECLARE
	v_last_name persons.last_name%TYPE;
BEGIN
	SELECT last_name INTO v_last_name FROM persons WHERE person_id = 1;
	DBMS_OUTPUT.PUT_LINE('Last Name: ' || v_last_name);
END;
```

### Conditions
```sql
DECLARE
	v_num NUMBER := &enter_a_number;
BEGIN
	IF mod(v_num,2) = 0 THEN
	    dbms_output.put_line(v_num || ' is even');
	ELSIF mod(v_num,2) = 1 THEN
	    dbms_output.put_line(v_num || ' is odd');
	ELSE
	    dbms_output.put_line('None');
	END IF;
END;
```
### Loops
```sql
--Simple Loop
DECLARE
	v_num number(5) := 0;
BEGIN
	loop
	    v_num := v_num + 1;
	    dbms_output.put_line('Number: ' || v_num);
	    
	    exit when v_num = 3;
	    /*
	    if v_num = 3 then
	        exit;
	    end if;
	    */
	end loop;
END;

--While Loop
DECLARE
	v_num number := 0;
BEGIN
	while v_num <= 100 loop

	    exit when v_num > 40;
	    
	    if v_num = 20 then
	        v_num := v_num + 1;
	        continue;
	    end if;
	    
	    if mod(v_num,10) = 0 then
	        dbms_output.put_line(v_num || ' can be divided by 10.');
	    end if;
	    
	    v_num := v_num + 1;
	    
	end loop;
END;

--For Loop
DECLARE
	v_num number := 0;
BEGIN
	for x in 10 .. 13 loop
	    dbms_output.put_line(x);
	end loop;

	for x in reverse 13 .. 15 loop
	    if mod(x,2) = 0 then
	        dbms_output.put_line('even: ' || x);
	    else
	        dbms_output.put_line('odd: ' || x);
	    end if;
	end loop;
END;
```

### Triggers
```sql
-- DML Triggers
CREATE OR REPLACE TRIGGER tr_persons
BEFORE INSERT OR DELETE OR UPDATE ON persons
FOR EACH ROW
ENABLE
DECLARE
	v_user varchar2(20);
BEGIN
	SELECT user INTO v_user FROM dual;
	IF INSERTING THEN
		DBMS_OUTPUT.PUT_LINE('One line inserted by ' || v_user);
	ELSIF DELETING THEN
		DBMS_OUTPUT.PUT_LINE('One line Deleted by ' || v_user);
	ELSIF UPDATING THEN
		DBMS_OUTPUT.PUT_LINE('One line Updated by ' || v_user);
	END IF;
END;

-- DDL Triggers
CREATE OR REPLACE TRIGGER db_audit_tr 
AFTER DDL ON DATABASE
BEGIN
    INSERT INTO schema_audit VALUES (
		sysdate, 
		sys_context('USERENV','CURRENT_USER'), 
		ora_dict_obj_type, 
		ora_dict_obj_name, 
		ora_sysevent);
END;

-- Instead of Triggers
CREATE VIEW vw_twotable AS
SELECT full_name, subject_name FROM persons, subjects;

CREATE OR REPLACE TRIGGER tr_Insert
INSTEAD OF INSERT ON vw_twotable
FOR EACH ROW
BEGIN
  INSERT INTO persons (full_name) VALUES (:new.full_name);
  INSERT INTO subjects (subject_name) VALUES (:new.subject_name);
END;

insert into vw_twotable values ('Caner','subject');
```
### Cursors
```sql
declare
    v_first_name varchar2(20);
    v_last_name varchar2(20);
    Cursor test_cursor is select first_name,last_name from persons;
begin
    open test_cursor;
    loop
        fetch test_cursor into v_first_name,v_last_name;
        exit when test_cursor%NOTFOUND;
        dbms_output.put_line('Name: ' || v_first_name || ', Lastname: ' || v_last_name);
    end loop;
    close test_cursor;
end;

----

declare
    v_first_name varchar2(20);
    v_last_name varchar2(20);
    Cursor test_cursor (first_name_parameter varchar2) is select first_name,last_name from persons where first_name = first_name_parameter;
begin
    open test_cursor('caner');
    loop
        fetch test_cursor into v_first_name,v_last_name;
        exit when test_cursor%NOTFOUND;
        dbms_output.put_line('Name: ' || v_first_name || ', Lastname: ' || v_last_name);
    end loop;
    close test_cursor;
end;

----

declare
    v_first_name varchar2(20);
    v_last_name varchar2(20);
    Cursor test_cursor (first_name_parameter varchar2 := 'caner') is select first_name,last_name from persons where first_name = first_name_parameter;
begin
    open test_cursor;
    loop
        fetch test_cursor into v_first_name,v_last_name;
        exit when test_cursor%NOTFOUND;
        dbms_output.put_line('Name: ' || v_first_name || ', Lastname: ' || v_last_name);
    end loop;
    close test_cursor;
end;

--for
declare
    Cursor test_cursor is select first_name,last_name from persons;
begin
    for obj in test_cursor
    loop 
        dbms_output.put_line('Name: ' || obj.first_name || ', Lastname: ' || obj.last_name);
    end loop;
end;

--for parameter
declare
    Cursor test_cursor (first_name_parameter varchar2 := 'can') is select first_name,last_name from persons where first_name = first_name_parameter;
begin
    for obj in test_cursor('caner')
    loop 
        dbms_output.put_line('Name: ' || obj.first_name || ', Lastname: ' || obj.last_name);
    end loop;
end;

```

### Records
```sql
--table based
declare 
    v_person persons%ROWTYPE;
begin
    select * into v_person from persons where PERSON_ID = 2;
    dbms_output.put_line('Name: ' || v_person.first_name || ', Lastname: ' || v_person.last_name);
end;

--

declare 
    v_person persons%ROWTYPE;
begin
    select first_name,last_name into v_person.first_name,v_person.last_name from persons where PERSON_ID = 2;
    dbms_output.put_line('Name: ' || v_person.first_name || ', Lastname: ' || v_person.last_name);
end;

--cursor based record
declare
    Cursor test_cursor is select first_name,last_name from persons where person_id = 2;
    v_person test_cursor%rowtype;
begin
    open test_cursor;
    fetch test_cursor into v_person;
    dbms_output.put_line('Name: ' || v_person.first_name || ', Lastname: ' || v_person.last_name);
    close test_cursor;
end;

--

declare
    Cursor test_cursor is select first_name,last_name from persons;
    v_person test_cursor%rowtype;
begin
    open test_cursor;
    loop
        fetch test_cursor into v_person;
        exit when test_cursor%NOTFOUND;
        dbms_output.put_line('Name: ' || v_person.first_name || ', Lastname: ' || v_person.last_name);
    end loop;
    close test_cursor;
end;

--user based

declare
    type rv_person is record(
        f_name varchar2(20),
        l_name persons.last_name%type
    );
    v_person rv_person;
begin
    select first_name,last_name into v_person.f_name,v_person.l_name from persons where person_id = 2;
    dbms_output.put_line('Name: ' || v_person.f_name || ', Lastname: ' || v_person.l_name);
end;
```

### Functions
```sql
CREATE OR REPLACE FUNCTION circle_area (radius NUMBER) 
RETURN NUMBER IS
--Declare a constant and a variable
pi      CONSTANT NUMBER(7,3) := 3.141;
area    NUMBER(7,3);
BEGIN
  --Area of Circle pi*r*r;
  area := pi * (radius * radius);
  RETURN area; 
END;

BEGIN
	dbms_output.put_line('Alan: ' || circle_area(10));
END;
```

### Stored Procedure
```sql
create or replace procedure pr_test is
    v_name varchar(20) := 'Caner';
    v_city varchar(20) := 'Istanbul';
begin
    dbms_output.put_line(v_name || ',' || v_city);
end pr_test;
--
execute pr_test;
--
begin
    pr_test;
end;

----

create or replace procedure pr_test_param(v_name varchar2 default 'caz') 
is
    v_city varchar(20) := 'Istanbul';
begin
    dbms_output.put_line(v_name || ',' || v_city);
end pr_test_param;
--
execute pr_test_param(v_name => 'cam');

----

create or replace procedure pr_test_param(v_name varchar2) 
is
    v_city varchar(20) := 'Istanbul';
begin
    dbms_output.put_line(v_name || ',' || v_city);
end pr_test_param;
--
execute pr_test_param('Caner');
--
begin
    pr_test_param('Caner');
end;
```

### Package
```sql
CREATE OR REPLACE PACKAGE pkg_person IS
  FUNCTION get_name (v_name VARCHAR2) RETURN VARCHAR2;
  PROCEDURE proc_update_lastname(p_id NUMBER, l_name VARCHAR2);
END pkg_person;

--Package Body
CREATE OR REPLACE PACKAGE BODY pkg_person IS
  --Function Implimentation
  FUNCTION get_name (v_name VARCHAR2) RETURN VARCHAR2 IS
    BEGIN
      RETURN v_name;
    END get_name;
   
  --Procedure Implimentation
   PROCEDURE proc_update_lastname(p_id NUMBER, l_name VARCHAR2) IS
     BEGIN
      UPDATE persons SET last_name = l_name where person_id = p_id;
     END;
   
END pkg_person;

--
begin
    dbms_output.put_line(pkg_person.get_name('Caner'));
end;
execute pkg_person.proc_update_lastname(2,'new lastname');
```

### Exceptions
```sql
accept p_divisor number prompt 'Enter divisor';
declare
    v_divided number := 24;
    v_divisor number := &p_divisor;
    v_result number;
    ex_four exception;
    pragma exception_init(ex_four,-20001); --20000 , 20999
begin
    if v_divisor = 4 then
        raise ex_four;
    end if;
    
    if v_divisor = 5 then
        raise_application_error(-20001,'div five');
    end if;
    
    if v_divisor = 6 then
        raise_application_error(-20002,'div six');
    end if;
    
    v_result := v_divided/v_divisor;
    
    exception 
        when ex_four then  --user defined
            dbms_output.put_line('Div four');
            dbms_output.put_line(SQLERRM);
        when ZERO_DIVIDE then --system defined
            dbms_output.put_line('Div zero');
        when OTHERS then
            dbms_output.put_line('Other exception');
            dbms_output.put_line(SQLERRM);
end;
```
