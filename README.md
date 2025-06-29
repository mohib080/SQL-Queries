![Alt Text](https://srikanthtechnologies.com/oracle/hrtables.gif)

online A1-A2

1.

CREATE OR REPLACE PROCEDURE RANK_JOBS(MIN_HIRED_COUNT IN NUMBER) IS
BEGIN
  -- Remove old data
  DELETE FROM JOB_RANK;

  -- Insert new ranked jobs
  INSERT INTO JOB_RANK (JOB_ID, RANK)
  SELECT JOB_ID, RANK
  FROM (
    SELECT j.JOB_ID,
           RANK() OVER (ORDER BY AVG(e.SALARY) DESC) AS RANK
    FROM JOBS j
    JOIN EMPLOYEES e ON j.JOB_ID = e.JOB_ID
    WHERE j.JOB_ID = UPPER(SUBSTR(j.JOB_TITLE, 1, 2) || '_' ||
                            SUBSTR(j.JOB_TITLE, INSTR(j.JOB_TITLE, ' ') + 1, 3))
    GROUP BY j.JOB_ID
    HAVING COUNT(*) >= MIN_HIRED_COUNT
  );
END;
/

2.

CREATE OR REPLACE TRIGGER VALIDATE_SALARY
BEFORE UPDATE OF SALARY ON TEMP_EMPLOYEES
FOR EACH ROW
DECLARE
  v_min_salary NUMBER;
BEGIN
  -- Case 1: Salary must not be less than min salary of same job
  SELECT MIN(SALARY)
  INTO v_min_salary
  FROM EMPLOYEES
  WHERE JOB_ID = :NEW.JOB_ID;

  IF :NEW.SALARY < v_min_salary THEN
    RAISE_APPLICATION_ERROR(-20001, 'New salary is below minimum for this job.');
  END IF;

  -- (Simplified: Other cases skipped for basic version)
END;
/