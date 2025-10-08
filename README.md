## Overview
This project is part of Data Engineering with AWS nanodegree at Udacity, it demonstrates data modeling in Apache Cassandra through the implementation of a music streaming application database. The focus is on query-driven design, where database tables are structured to optimize specific query patterns rather than following traditional normalization principles.

## Dataset
The project utilizes music event data stored in CSV format (`event_datafile_new.csv`), containing user listening history with the following attributes:
- Artist name and song title
- Song duration
- User information (first name, last name, user ID)
- Session tracking data (session ID, item sequence number)

The data represents typical user interactions in a music streaming service.

## The Three Questions I'm Answering

### 1. What song was playing at a specific moment in a session?
I needed to find the artist, song title, and length for when sessionId was 338 and it was the 4th item played.

**Table Design**:
```sql
CREATE TABLE music_query1 (
    sessionId int,
    itemInSession int,
    artist text,
    song text,
    length float,
    PRIMARY KEY (sessionId, itemInSession)
)
```

**Key Design Decisions**:
- **Partition Key**: `sessionId` - groups all items from the same session together
- **Clustering Column**: `itemInSession` - sorts items within each session by sequence number
- Enables efficient lookup of specific items within a session

### Query 2: User Session History
**Question**: Get artist name, song title (sorted by itemInSession), and user name for userId = 10 and sessionId = 182

**Table Design**:
```sql
CREATE TABLE music_query2 (
    userId int,
    sessionId int,
    itemInSession int,
    artist text,
    song text,
    ufirst text,
    ulast text,
    PRIMARY KEY ((userId, sessionId), itemInSession)
)
```

**Key Design Decisions**:
- **Composite Partition Key**: `(userId, sessionId)` - groups all items for a specific user's session
- **Clustering Column**: `itemInSession` - automatically sorts songs by their play order
- Optimized for retrieving a complete user session in chronological order

### Query 3: Song Listeners
**Question**: Get all user names (first and last) who listened to the song 'All Hands Against His Own'

**Table Design**:
```sql
CREATE TABLE music_query3 (
    song text,
    userId int,
    ufirst text,
    ulast text,
    PRIMARY KEY (song, userId)
)
```

**Key Design Decisions**:
- **Partition Key**: `song` - groups all listeners of the same song together
- **Clustering Column**: `userId` - ensures each user's listen is stored as a unique row
- Prevents data overwrites when multiple users listen to the same song

## Technologies Used
- **Apache Cassandra 3.11+** - NoSQL distributed database
- **Python 3.x** - Programming language
- **cassandra-driver** - Python driver for Cassandra
- **Pandas** - Data manipulation (if used for CSV processing)
- **Jupyter Notebook** - Interactive development environment

## Key Concepts Demonstrated

### 1. Query-Driven Design
In Cassandra, tables are designed around specific query patterns rather than normalizing data. Each query gets its own optimized table.

### 2. Primary Key Structure
- **Partition Key**: Determines data distribution across nodes
- **Clustering Columns**: Determines sort order within a partition
- Format: `PRIMARY KEY ((partition_key), clustering_column1, clustering_column2)`

### 3. Data Denormalization
The same data appears in multiple tables to support different query patterns efficiently - a fundamental NoSQL principle.

### 4. No JOINs
Unlike SQL databases, Cassandra doesn't support JOINs. All data needed for a query must exist in the same table.

## Project Structure
```
.
├── README.md
├── cassandra_data_modeling.ipynb
├── event_datafile_new.csv
└── requirements.txt
```

## Setup Instructions

### Prerequisites
- Python 3.7+
- Apache Cassandra 3.11+ installed and running locally
- pip (Python package manager)

### Installation

1. **Clone the repository**:
```bash
git clone https://github.com/AbdullahMH21/cassandra-data-modeling.git
cd cassandra-data-modeling
```

2. **Install required Python packages**:
```bash
pip install cassandra-driver
pip install pandas
pip install jupyter
```

3. **Start Cassandra** (if not already running):
```bash
# On Linux/Mac
cassandra -f

# On Windows (if installed as service)
net start cassandra
```

4. **Create a keyspace** (in cqlsh or via Python):
```sql
CREATE KEYSPACE IF NOT EXISTS music_keyspace 
WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor': 1};
```

5. **Open the Jupyter Notebook**:
```bash
jupyter notebook cassandra_data_modeling.ipynb
```

## Usage

Run the notebook cells in order:
1. **Setup**: Connect to Cassandra cluster and create keyspace
2. **Create Tables**: Execute CREATE TABLE statements for all three queries
3. **Load Data**: Insert data from CSV into each table
4. **Query Data**: Run SELECT statements to verify the data models
5. **Cleanup**: Drop tables when done (optional)

## Sample Queries

```python
# Query 1: Get session song details
query = """SELECT artist, song, length 
           FROM music_query1 
           WHERE sessionId = 338 AND itemInSession = 4"""

# Query 2: Get user session history
query = """SELECT artist, song, ufirst, ulast 
           FROM music_query2 
           WHERE userId = 10 AND sessionId = 182
           ORDER BY itemInSession"""

# Query 3: Get all listeners of a song
query = """SELECT ufirst, ulast 
           FROM music_query3 
           WHERE song = 'All Hands Against His Own'"""
```

## Learning Outcomes
- Understanding Cassandra's distributed architecture
- Designing partition keys for even data distribution
- Using clustering columns for efficient sorting
- Implementing denormalized data models
- Avoiding common Cassandra anti-patterns
- Query optimization for NoSQL databases

## Author
Abdullah Alhethely - [GitHub Profile](https://github.com/AbdullahMH21)