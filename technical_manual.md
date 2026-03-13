# 📖 Detailed Technical Standard Operating Procedure (SOP)

## Stage 1: Provisioning & Data Entry
1. **Created a named Docker Volume:** `docker volume create old_server_data`
2. **Deployed the DB Container:**
   `docker run -d --name legacy_db -v old_server_data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=mysecretpassword postgres:14`
3. **Database Schema Construction:**
   - Entered container: `docker exec -it legacy_db psql -U postgres`
   - SQL Execution:
     ```sql
     CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(50));
     INSERT INTO users (name) VALUES ('Alice'), ('Bob');
     ```

## Stage 2: The Cold Backup Operation
*Crucial Step: Stopping the engine to prevent data corruption.*
1. **Stop Container:** `docker stop legacy_db`
2. **Execute Multi-Volume Mount Backup:**
   `docker run --rm -v old_server_data:/volume -v "${PWD}:/backup" alpine tar -czvf /backup/db_backup.tar.gz -C /volume .`
   - **Explanation:** This mounts the "Vault" and the "Windows Folder" simultaneously. The Alpine container zips the data from one to the other and then self-destructs (`--rm`).

## Stage 3: The Disaster Simulation
1. **Delete Container:** `docker rm legacy_db`
2. **Wipe Volume:** `docker volume rm old_server_data`
   - *Result: 100% of the internal Docker infrastructure was destroyed.*

## Stage 4: Restoration & Verification
1. **Create New Volume:** `docker volume create new_server_data`
2. **Extract Archive:** `docker run --rm -v new_server_data:/volume -v "${PWD}:/backup" alpine tar -xzvf /backup/db_backup.tar.gz -C /volume`
3. **Deploy Recovery Server:**
   `docker run -d --name recovery_db -v new_server_data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=mysecretpassword postgres:14`
