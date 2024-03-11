# Advanced Data Types TripAdvisor

### 1. Enumerated Data Types

Enumerated types are a way to create a custom type that can hold one of a few predefined values.

**Step-by-Step Guide:**

- **Create an Enum Type:** First, define your enum type with its possible values.

```sql
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');
```

- **Use the Enum Type in a Table:**

```sql
CREATE TABLE person (
    name text,
    current_mood mood
);
```

- **Insert Data:**

```sql
INSERT INTO person (name, current_mood) VALUES ('John', 'happy');
```

- **Query Data:**

```sql
SELECT * FROM person WHERE current_mood = 'happy';
```

### 2. UUID

UUIDs are 128-bit labels used for uniquely identifying information in a database.

**Step-by-Step Guide:**

- **Create a Table with a UUID Column:**

```sql
CREATE TABLE user_accounts (
    user_id uuid PRIMARY KEY,
    username text
);
```

- **Insert Data:**

```sql
INSERT INTO user_accounts (user_id, username) VALUES (uuid_generate_v4(), 'johndoe');
```
(Note: Ensure the `uuid-ossp` extension is enabled in your database to use `uuid_generate_v4()`.)

### 3. JSON and JSONB

PostgreSQL supports JSON data types, allowing you to store and query JSON format data.

**Step-by-Step Guide:**

- **Create a Table with JSON/JSONB Column:**

```sql
CREATE TABLE products (
    product_id serial PRIMARY KEY,
    product_info json
);
```

- **Insert JSON Data:**

```sql
INSERT INTO products (product_info) VALUES ('{"name": "Laptop", "specs": {"memory": "16GB", "storage": "512GB SSD"}}');
```

### 4. Arrays

Arrays allow you to store multiple values in a single column.

**Step-by-Step Guide:**

- **Create a Table with an Array Column:**

```sql
CREATE TABLE contacts (
    name text,
    phone_numbers text[]
);
```

- **Insert Data:**

```sql
INSERT INTO contacts (name, phone_numbers) VALUES ('John Doe', ARRAY['+123456789', '+987654321']);
```

### 5. Binary Data Types

Binary data types are used to store data such as images, files, etc., in binary format.

**Step-by-Step Guide:**

- **Create a Table with a BYTEA Column:**

```sql
CREATE TABLE files (
    file_name text,
    file_data bytea
);
```

- **Insert Binary Data:**

```sql
INSERT INTO files (file_name, file_data) VALUES ('example.txt', pg_read_binary_file('/path/to/your/file/example.txt'));
```

### 6. Date-Time Related Data Types

PostgreSQL offers a variety of date and time data types.

**Step-by-Step Guide:**

- **Create a Table with Date, Time, Timestamp, and Interval Columns:**

```sql
CREATE TABLE events (
    event_name text,
    event_date date,
    event_time time,
    event_timestamp timestamptz,
    event_duration interval
);
```

- **Insert Data:**

```sql
INSERT INTO events (event_name, event_date, event_time, event_timestamp, event_duration) VALUES ('Conference', '2023-10-05', '09:00:00', '2023-10-05 09:00:00+00', '3 hours');
```

- **Query Data:**

```sql
SELECT * FROM events WHERE event_date = '2023-10-05';
```

Each of these examples demonstrates how to use advanced data types in PostgreSQL, from creation to insertion and querying. These data types enable more complex and powerful data structures and queries, enhancing the capabilities of your PostgreSQL databases.


Let's create a comprehensive example that incorporates all the advanced data types discussed, in the context of a hypothetical `trip_advisor` database. This database will include tables for users, trips, reviews, and more, showcasing how enumerated types, UUIDs, JSON/JSONB, arrays, binary data, and date-time related data types can be utilized.

### Step 1: Setup

First, ensure your PostgreSQL database is ready and that you have the necessary extensions, especially for UUIDs.

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

### Step 2: Enumerated Data Types

Let's define an enumerated type for review ratings.

```sql
CREATE TYPE rating AS ENUM ('poor', 'fair', 'good', 'excellent');
```

### Step 3: UUID

We'll use UUIDs for user IDs for better scalability and uniqueness.

```sql
CREATE TABLE users (
    user_id uuid PRIMARY KEY DEFAULT uuid_generate_v4(),
    username text NOT NULL,
    preferences jsonb
);
```

### Step 4: JSON/JSONB

Storing user preferences in a JSONB column allows for flexible data about user interests and settings.

```sql
INSERT INTO users (username, preferences) VALUES ('john_doe', '{"interests": ["hiking", "photography"], "newsletter": true}');
```

### Step 5: Arrays

We can use arrays to store multiple phone numbers for trip organizers.

```sql
CREATE TABLE organizers (
    organizer_id serial PRIMARY KEY,
    name text NOT NULL,
    phone_numbers text[] NOT NULL
);
```

### Step 6: Binary Data Types

Assuming we want to store a binary representation of trip brochures.

```sql
CREATE TABLE brochures (
    brochure_id serial PRIMARY KEY,
    trip_id int NOT NULL,
    brochure_data bytea
);
```

### Step 7: Date-Time Related Data Types

We'll manage trip dates, booking windows, and review timestamps.

```sql
CREATE TABLE trips (
    trip_id serial PRIMARY KEY,
    destination text NOT NULL,
    trip_date date NOT NULL,
    booking_window tsrange NOT NULL
);

CREATE TABLE reviews (
    review_id serial PRIMARY KEY,
    trip_id int NOT NULL,
    user_id uuid NOT NULL,
    review_text text,
    rating rating,
    review_timestamp timestamptz DEFAULT now()
);
```

### Step 8: Comprehensive Example

Now, let's put it all together with some example data.

```sql
-- Add an organizer
INSERT INTO organizers (name, phone_numbers) VALUES ('Mountain Adventures', ARRAY['+1234567890', '+0987654321']);

-- Add a trip
INSERT INTO trips (destination, trip_date, booking_window) VALUES ('Mount Everest', '2023-12-01', '[2023-01-01 00:00, 2023-11-01 00:00]');

-- Add a brochure
INSERT INTO brochures (trip_id, brochure_data) VALUES (1, pg_read_binary_file('/path/to/brochure.pdf'));

-- Add a review
INSERT INTO reviews (trip_id, user_id, review_text, rating) VALUES (1, (SELECT user_id FROM users WHERE username = 'john_doe'), 'Incredible experience, highly recommended!', 'excellent');
```

### Step 9: Querying

To showcase querying these data types:

```sql
-- Find all excellent reviews
SELECT * FROM reviews WHERE rating = 'excellent';

-- Find trips with booking available
SELECT * FROM trips WHERE now() <@ booking_window;

-- Get user preferences
SELECT username, preferences ->> 'interests' AS interests FROM users WHERE username = 'john_doe';
```

This comprehensive example demonstrates how to use advanced data types in PostgreSQL to model a complex `trip_advisor` database, covering users, trips, organizers, brochures, and reviews. Each data type serves a specific purpose, from ensuring unique identifiers with UUIDs, storing complex and flexible data with JSONB, to managing dates and times for trips and reviews.


Building upon the `trip_advisor` database example, let's explore additional queries that leverage the advanced data types we've discussed. These queries will demonstrate how to interact with and extract meaningful information from the database, covering a range of functionalities.

### 1. Querying Enumerated Types

Find the number of reviews by rating:

```sql
SELECT rating, COUNT(*) AS review_count
FROM reviews
GROUP BY rating
ORDER BY review_count DESC;
```

### 2. Querying UUIDs

Retrieve a user's information based on their UUID:

```sql
SELECT username, preferences
FROM users
WHERE user_id = 'specific-uuid-here'; -- Replace 'specific-uuid-here' with an actual UUID
```

### 3. Querying JSON/JSONB

Find users interested in a specific activity, e.g., "hiking":

```sql
SELECT username
FROM users
WHERE preferences -> 'interests' ? 'hiking';
```

### 4. Querying Arrays

Find organizers with a specific phone number:

```sql
SELECT name
FROM organizers
WHERE phone_numbers @> ARRAY['+1234567890']; -- Use the desired phone number
```

### 5. Querying Binary Data

Assuming you want to find if a brochure exists for a specific trip (Note: This is more about the existence check rather than querying binary data directly, as binary data is not typically queried for specific content in SQL):

```sql
SELECT EXISTS (
    SELECT 1
    FROM brochures
    WHERE trip_id = 1 -- Use the desired trip_id
);
```

### 6. Querying Date-Time Related Data Types

Find trips happening within a specific date range:

```sql
SELECT destination, trip_date
FROM trips
WHERE trip_date BETWEEN '2023-01-01' AND '2023-12-31';
```

Find reviews made in the last month:

```sql
SELECT review_text, review_timestamp
FROM reviews
WHERE review_timestamp > NOW() - INTERVAL '1 month';
```

### 7. Additional Queries

#### Find Trips with No Reviews Yet

```sql
SELECT t.destination
FROM trips t
LEFT JOIN reviews r ON t.trip_id = r.trip_id
WHERE r.review_id IS NULL;
```

#### Update User Preferences

Add a new interest to a user's preferences:

```sql
UPDATE users
SET preferences = jsonb_set(preferences, '{interests}', preferences->'interests' || '"surfing"', true)
WHERE username = 'john_doe';
```

#### List Trips and Their Average Ratings

```sql
SELECT t.destination, AVG(CASE r.rating
    WHEN 'poor' THEN 1
    WHEN 'fair' THEN 2
    WHEN 'good' THEN 3
    WHEN 'excellent' THEN 4
    END) AS avg_rating
FROM trips t
JOIN reviews r ON t.trip_id = r.trip_id
GROUP BY t.destination
ORDER BY avg_rating DESC;
```

These queries showcase the versatility and power of PostgreSQL's advanced data types in a real-world application scenario. They allow for complex and efficient data storage, retrieval, and analysis, which are crucial for dynamic and data-driven applications like a trip advisor platform.
