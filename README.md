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
