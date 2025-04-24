## Script: Manage_SQL_Agent_Jobs.sql

**Description**:
This script set provides tools for **monitoring**, **managing**, and **modifying** SQL Server Agent jobs, including checking job ownership, changing owners, viewing job execution status, and stopping jobs via T-SQL.

---

### 🔍 Part 1: Check Current Job Owner

```sql
SELECT 
    name AS JobName, 
    SUSER_SNAME(owner_sid) AS JobOwner
FROM msdb.dbo.sysjobs;
```

This returns the owner (login) for each SQL Agent job.

---

### 🔄 Part 2: Change Job Owner

```sql
USE msdb;
GO

EXEC sp_update_job 
    @job_name = N'Your Job Name',
    @owner_login_name = N'NewLoginName';
```

> Replace:
> - `'Your Job Name'` with the job you want to modify
> - `'NewLoginName'` with a valid SQL login or Windows login

---

### 📈 Part 3: Check Job Execution State

```sql
SELECT 
    j.name AS JobName,
    ja.start_execution_date,
    ja.stop_execution_date,
    CASE 
        WHEN ja.stop_execution_date IS NULL THEN 'Possibly Running'
        ELSE 'Completed'
    END AS JobStatus
FROM 
    msdb.dbo.sysjobactivity ja
JOIN 
    msdb.dbo.sysjobs j ON ja.job_id = j.job_id
WHERE 
    j.name = 'Job name'
ORDER BY 
    ja.start_execution_date DESC;
```

This helps determine if a job is **currently running**, **completed**, or **hung** (if no stop time recorded).

---

### 🛑 Part 4: Stop Job Using T-SQL

```sql
USE msdb;
GO

EXEC msdb.dbo.sp_stop_job 
    @job_name = 'Job name';
```

Use this to forcibly stop a **long-running** or **stuck** job.

---

### ✅ Use Case:
- Audit SQL Agent job ownership
- Transfer job ownership after DBA or service account change
- Check job runtime history or identify stuck jobs
- Stop resource-consuming or misbehaving jobs

---

### ⚠️ Notes:
- `sp_update_job` requires the new login to exist and have proper permissions.
- Use `sp_stop_job` with caution — it stops the job **immediately**, which may interrupt transactions.
- Always check `msdb.dbo.sysjobhistory` if you need detailed job step logs.


