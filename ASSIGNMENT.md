# THIS md IS FOR A FUTURE ASSIGNMENT, IGNORE IT FOR NOW

# dbt Assignment: AirStats Capstone Project

Build a dbt project analyzing global airport data on Snowflake.

---

## Getting Started

1. Copy this repository clicking the "Use Template" button
2. Make your repository private
3. In your repository do any of these:
   - **Use as a Codespace**: Click "Code" > "Codespaces" > "Create codespace on main"
   - **Clone locally**: Clone your fork, then run:
     ```bash
     uv sync
     source .venv/bin/activate # (or equivalent on your platform, see the README of the course)
     ```

---

## Part 1: Project Setup

### Step 0: Add Your Student id
Add your student here (edit the next line):
```
Student id: REPLACE THIS WITH YOUR STUDENT ID
```

### Step 1: Initialize the dbt Project

* Create a new dbt project called `airstats`. You only need to do the `dbt init ...` step, your uv/virtualenv is already up and running

### Step 2: Configure the Connection

Copy the `profiles.yml` from the `airbnb` project to the `airstats` folder:

```sh
cp airbnb/profiles.yml airstats/profiles.yml
```

Now edit `airstats/profiles.yml` and make these changes:

**Before (airbnb configuration):**
```yaml
airbnb:
  outputs:
    dev:
      type: snowflake
      account: "..."
      user: dbt
      role: TRANSFORM
      private_key: "..."
      private_key_passphrase: q
      database: AIRBNB
      schema: DEV
      threads: 1
      warehouse: COMPUTE_WH
  target: dev
```

**After (airstats configuration):**
```yaml
airstats:
  outputs:
    dev:
      type: snowflake
      account:  "..."
      user: dbt
      role: TRANSFORM
      private_key: "..."
      private_key_passphrase: q
      database: AIRSTATS
      schema: DEV
      threads: 1
      warehouse: COMPUTE_WH
  target: dev
```

**Summary of changes:**
1. Change the profile name from `airbnb` to `airstats` (first line)
2. Change the database from `AIRBNB` to `AIRSTATS`

### Step 3: Verify the Connection

```sh
cd airstats
dbt debug
```

You should see "All checks passed!" if the connection is configured correctly.

Once done **Add `airstats` to git**, ensure that your `profiles.yml` is added too (this is OK as it's an assignment in a private repository; never add credentials to git in a real-world project)

### Step 4: Clean Up Example Files

Remove the example models that dbt created by default:

```sh
rm -rf models/example
```

Also remove the example model configuration from `dbt_project.yml`. Delete these lines at the end of the file:

```yaml
models:
  airstats:
    example:
      +materialized: view
```

---

## Part 2: Data Exploration (Optional)

Before building models, explore the data in Snowflake, here are a few SQLs to get you started

```sql
USE AIRSTATS.RAW;

-- Check the airports table
SELECT * FROM airports LIMIT 10;

-- Count airports by type
SELECT type, COUNT(*) as count
FROM airports
GROUP BY type
ORDER BY count DESC;

-- Check the runways table
SELECT * FROM runways LIMIT 10;

-- Check the comments table
SELECT * FROM airport_comments LIMIT 10;
```

---

## Part 3: Define Sources

The AIRSTATS database has been set up in Snowflake with the following tables in the `RAW` schema:

| Table | Description | Key Columns |
|-------|-------------|-------------|
| `airports` | Global airport data (~72K rows) | `id`, `ident`, `type`, `name`, `iso_country` |
| `airport_comments` | User comments about airports | `id`, `airport_ref`, `airport_ident`, `date` |
| `runways` | Runway information (~44K rows) | `id`, `airport_ref`, `airport_ident`, `closed` |

### Exercise 1: Create the Sources File

Create a new file `models/sources.yml` that defines these three source tables.

**Requirements:**
1. The source name should be `airstats`
2. The database should be `AIRSTATS`
3. The schema should be `RAW`
4. Define all three tables: `airports`, `airport_comments`, and `runways`

### Verify Your Sources

After creating the sources file, you can verify it works by running:

```sh
dbt compile
```

If there are no errors, your sources are configured correctly.

---

## Part 4: Staging Models (src layer)

The staging layer (src) is responsible for:
- Selecting only the columns we need from source tables
- Renaming columns to follow consistent naming conventions
- Referencing sources using the dbt `source()` function

Create a folder `models/src/` for these models.

### Exercise 2: src_airports

Create `models/src/src_airports.sql`

**Requirements:**
1. Use a CTE to reference the `airports` source
2. Select and rename the following columns:

| Source Column | Target Column |
|---------------|---------------|
| `ident` | `airport_ident` |
| `type` | `airport_type` |
| `name` | `airport_name` |
| `latitude_deg` | `airport_lat` |
| `longitude_deg` | `airport_long` |
| `continent` | `continent` (no rename) |
| `iso_country` | `iso_country` (no rename) |
| `iso_region` | `iso_region` (no rename) |
| `scheduled_service` | `scheduled_service` (no rename) |

### Exercise 3: src_airport_comments

Create `models/src/src_airport_comments.sql`

**Requirements:**
1. Use a CTE to reference the `airport_comments` source
2. Select and rename the following columns:

| Source Column | Target Column |
|---------------|---------------|
| `id` | `comment_id` |
| `airport_ident` | `airport_ident` (no rename) |
| `date` | `comment_date` |
| `member_nickname` | `member_nickname` (no rename) |
| `subject` | `comment_subject` |
| `body` | `comment_body` |

### Exercise 4: src_runways

Create `models/src/src_runways.sql`

**Requirements:**
1. Use a CTE to reference the `runways` source
2. Select and rename the following columns:

| Source Column | Target Column |
|---------------|---------------|
| `id` | `runway_id` |
| `airport_ref` | `airport_ref` (no rename) |
| `airport_ident` | `airport_ident` (no rename) |
| `length_ft` | `runway_length_ft` |
| `width_ft` | `runway_width_ft` |
| `surface` | `runway_surface` |
| `lighted` | `runway_lighted` |
| `closed` | `runway_closed` |

### Verify Your Models

Run dbt to build the staging models:

```sh
dbt run
```

All three models should complete successfully.

## Submission

TODO
