# 🐘 PostgreSQL Cheat Sheet

## **1. Checking PostgreSQL Version & Connection**
```sh
psql --version                         # Check PostgreSQL version
psql -U username -d database_name       # Connect to a database
\conninfo                               # Show current connection info
```

---

## **2. Managing Databases & Users**
```sh
CREATE DATABASE mydb;                  # Create a new database
DROP DATABASE mydb;                     # Delete a database
CREATE USER myuser WITH PASSWORD 'mypassword';  # Create a new user
ALTER USER myuser WITH SUPERUSER;       # Grant superuser privileges
DROP USER myuser;                        # Delete a user
```

---

## **3. Schema & Table Management**
```sh
CREATE SCHEMA myschema;                  # Create a schema
CREATE TABLE mytable (id SERIAL PRIMARY KEY, name TEXT);  # Create a table
ALTER TABLE mytable ADD COLUMN age INT;   # Add a column to an existing table
DROP TABLE mytable;                       # Delete a table
```

---

## **4. Indexing for Performance Optimization**
```sh
CREATE INDEX idx_name ON mytable(name);   # Create a basic index
CREATE UNIQUE INDEX idx_email ON users(email);  # Create a unique index
CREATE INDEX idx_gin ON mytable USING gin(to_tsvector('english', column));  # Full-text search index
DROP INDEX idx_name;                      # Remove an index
```

---

## **5. JSON & JSONB Handling**
```sh
SELECT myjson->'key' FROM mytable;       # Extract a JSON field
SELECT myjson->>'key' FROM mytable;      # Extract a JSON field as text
SELECT jsonb_pretty(myjson) FROM mytable; # Pretty print JSONB
UPDATE mytable SET myjson = jsonb_set(myjson, '{key}', '"new_value"');  # Modify JSON field
```

---

## **6. User-Defined Functions (UDFs) & Stored Procedures**
```plpgsql
CREATE FUNCTION add_numbers(a INT, b INT) RETURNS INT AS $$
BEGIN
  RETURN a + b;
END;
$$ LANGUAGE plpgsql;

SELECT add_numbers(5, 10);  # Call the function

CREATE PROCEDURE log_message(msg TEXT) AS $$
BEGIN
  INSERT INTO logs (message) VALUES (msg);
END;
$$ LANGUAGE plpgsql;
```

---

## **7. Partitioning for Large Datasets**
```sh
CREATE TABLE parent_table (id SERIAL, name TEXT, date TIMESTAMP) PARTITION BY RANGE (date);

CREATE TABLE partition_2024 PARTITION OF parent_table FOR VALUES FROM ('2024-01-01') TO ('2024-12-31');

INSERT INTO parent_table VALUES (1, 'John', '2024-06-01');  # Data is automatically routed
```

---

## **8. Performance Optimization & Vacuuming**
```sh
VACUUM ANALYZE;                         # Clean up and optimize tables
EXPLAIN ANALYZE SELECT * FROM mytable;  # Analyze query execution plan
ALTER TABLE mytable SET (autovacuum_enabled = false);  # Disable autovacuum on a table
```

---

## **9. Replication & High Availability**
```sh
SHOW wal_level;                           # Check WAL settings for replication
SELECT * FROM pg_stat_replication;        # Check active replication status
ALTER SYSTEM SET wal_level = 'replica';   # Enable replication
```

---

## **10. Security & Access Control**
```sh
GRANT SELECT ON mytable TO myuser;        # Grant read access
REVOKE INSERT ON mytable FROM myuser;     # Remove insert permission
ALTER SYSTEM SET password_encryption = 'scram-sha-256';  # Enforce secure password encryption
```

---

### **Happy PostgreSQL Tuning! 🚀**
