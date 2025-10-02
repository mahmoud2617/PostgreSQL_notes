## What's the PostgreSQL?

PostgreSQL, commonly pronounced _post-gres_, is an open source database that has a strong reputation for its reliability, flexibility and support of open technical standards.

Unlike other RDMBS (Relational Database Management Systems), [PostgreSQL](https://www.postgresql.org/) supports both non-relational and relational data types. This makes it one of the most compliant, stable, and mature relational databases available today.

---

## Installing & configuring PostgreSQL:

**_`The following installation is for fedora users.`_**

1. **Update package metadata:**

~~~bash
sudo dnf update -y
~~~

2. **Search for available PostgreSQL versions:**

~~~bash
dnf search postgresql
~~~

3. **You’ll see packages like:**

~~~bash
postgresql.x86_64 
postgresql-server.x86_64 
postgresql-contrib.x86_64
~~~

4. **Install server + contrib:**

~~~bash
sudo dnf install postgresql-server postgresql-contrib -y
~~~

5. **Initialize database cluster:**

~~~bash
sudo postgresql-setup --initdb
~~~

6. **Enable and start PostgreSQL:**

~~~bash
sudo systemctl enable postgresql 
sudo systemctl start postgresql
~~~

7. **Check service status:**

~~~bash
systemctl status postgresql
~~~

8. **Switch to `postgres` user and open shell:**

~~~bash
sudo -i -u postgres psql
~~~

---

## psql CLI basics:

-> **When you run:**

~~~bash
psql -U postgres -d mydb
~~~

you’re entering the **psql shell**, where you can type SQL and special commands.

**_Getting in:_**
- `psql` → start the shell.
- `-U postgres` → connect as user `postgres`.
- `-d mydb` → connect to database `mydb`.
- If you just run `psql` after `sudo -i -u postgres`, it connects to the `postgres` database by default.

-> **Prompt looks like:**

~~~bash
mydb=#
~~~

- `mydb` → the database you’re connected to.
- `=#` → `#` means you’re a superuser. If you were a normal user, you’d see `=>`.


#### Two types of commands:

Inside `psql`, you can type:

1. **SQL statements** (must end with `;`)  
    Example:

~~~bash
CREATE DATABASE testdb;
~~~

~~~bash
SELECT version();
~~~

2. **psql meta-commands** (start with backslash `\`, no semicolon needed).
    Example:

~~~bash
\q       -- quit 
\l       -- list databases 
\c mydb  -- connect to database mydb 
\dt      -- list tables 
\du      -- list users/roles
~~~


#### Handy psql tricks:

- `\?` → help about all `psql` commands.
- `\h` → help about SQL syntax (`\h CREATE TABLE`).
- `\dn` → list schemas.
- `\d table_name` → describe a table (columns, types, constraints).
- Arrow keys → browse history.
- `\q` → exit.

---

## Creating databases & users:

#### 1. Connect to PostgreSQL:

-> **First, switch into the postgres system user and open the CLI:**

~~~bash
sudo -i -u postgres psql
~~~

-> **You’ll see a prompt like:**

~~~bash
postgres=#
~~~

That means: connected to the `postgres` database as the `postgres` superuser.


#### 2. Create a new database:

-> **In SQL:**

~~~bash
CREATE DATABASE mydb;
~~~

- `CREATE DATABASE` → the SQL command.    
- `mydb` → your new database’s name.

-> **Check it:**

~~~bash
\l
~~~

- `\l` = list all databases.


#### 3. Create a new user (role):

-> **In PostgreSQL, _users = roles that can log in_.**

~~~bash
CREATE USER mahmoud WITH PASSWORD 'secretpass';
~~~

- `CREATE USER` → makes a new login role.
- `mahmoud` → the username.
- `WITH PASSWORD` → assigns a password.

-> **Check users:**

~~~bash
\du
~~~


#### 4. Grant privileges:

-> **By default, your new user can’t do anything in `mydb`. Let’s fix that:**

~~~bash
GRANT ALL PRIVILEGES ON DATABASE mydb TO mahmoud;
~~~

Now `mahmoud` can connect, create tables, insert data, etc.


#### 5. Connect as the new user:

-> **Exit to the shell:**

~~~bash
\q
~~~

-> **Then connect as the new user:**

~~~bash
psql -U mahmoud -d mydb -h localhost
~~~

- `-U mahmoud` → login as user mahmoud.
- `-d mydb` → connect to database mydb.
- `-h localhost` → host (needed if not running as postgres system user).

You’ll be asked for the password (`secretpass`).


#### 6. Security tip:

-> **Instead of `CREATE USER`, you can use:**

~~~bash
CREATE ROLE mahmoud WITH LOGIN PASSWORD 'secretpass';
~~~

Because technically **USER = ROLE + LOGIN**.


#### Mini cheat sheet:

- `\l` → list databases
- `\du` → list users
- `\c dbname` → connect to a database
- `\d tablename` → describe a table

---

## Connecting to a database:

#### 1. Inside `psql` (already connected as postgres):

-> **Suppose you’re logged in as the `postgres` superuser and you want to switch to another database:**

~~~bash
\c mydb
~~~

- `\c` → short for **connect**.
- `mydb` → the target database name.

-> **Prompt changes to show the new database:**

~~~bash
mydb=#
~~~

-> **If you also want to switch the user at the same time:**

~~~bash
\c mydb mahmoud
~~~

That tries to connect to `mydb` as user `mahmoud`.


#### 2. From the system shell (not inside psql yet):

-> **You can connect directly using the `psql` command:**

~~~bash
psql -U mahmoud -d mydb -h localhost -p 5432
~~~

**_Breakdown:_**
- `-U mahmoud` → username (who you log in as).    
- `-d mydb` → database name.
- `-h localhost` → host (where the DB server is running, here it’s the same machine).
- `-p 5432` → port (default PostgreSQL port).

If the user has a password, psql will prompt you.


#### 3. Defaults (when you omit flags):

-> **PostgreSQL is opinionated with defaults:**

- If you just type `psql` → it tries to connect as your **Linux username** to a database with the **same name**.  
    Example: user `mahmoud` → tries to connect to database `mahmoud`.

- If you type `psql -U postgres` → it connects to the `postgres` database by default.


#### 4. Environment variables (shortcut way):

-> **You can set env vars so you don’t need to type `-U`, `-d`, `-h`, `-p` every time.**

~~~bash
export PGUSER=mahmoud 
export PGPASSWORD=secretpass 
export PGDATABASE=mydb 
export PGHOST=localhost 
export PGPORT=5432
~~~

Now just run:

~~~bash
psql
~~~

…and it connects automatically using those values.


#### 5. Verifying the connection:

-> **Once inside `psql`, check where you are:**

~~~bash
SELECT current_database();
SELECT current_user;
~~~

That shows which DB and which user you’re currently using.


#### So connecting boils down to:

- **Inside `psql`** → `\c dbname [username]`
- **Outside** → `psql -U username -d dbname -h host -p port`

---
