# **YOUTUBE DATA HARVESTING AND WAREHOUSING USING SQL, MongoDB and Streamlit**

## Abstract

The "YouTube Data Harvesting and Warehousing" project is designed to facilitate the systematic collection, storage, and exploration of data from YouTube channels. Leveraging the YouTube Data API, the project extracts comprehensive information, including channel details, playlists, videos, and comments. The harvested data is then stored in both MongoDB and PostgreSQL databases, ensuring a dual-storage approach for flexibility and efficiency.

The project employs Python scripting, utilizing libraries such as pymongo, psycopg2, pandas, streamlit, and the Google API Python Client. It integrates seamlessly with MongoDB and PostgreSQL, providing a versatile solution for data warehousing.

The user interface is built using Streamlit, offering an intuitive web application for data exploration. Users can input YouTube Channel IDs, trigger the data collection process, and visualize the stored information dynamically. The application supports SQL queries to retrieve insightful information, empowering users to derive valuable insights from the amassed data.

Key features include data harvesting from YouTube, dual storage in NoSQL and relational databases, a user-friendly Streamlit interface, and the ability to perform SQL queries for in-depth analysis. The project is accompanied by comprehensive documentation, guiding users through setup, usage, and customization.

The "YouTube Data Harvesting and Warehousing" project aims to enhance the accessibility and analysis of YouTube data, making it a valuable tool for researchers, analysts, and enthusiasts interested in understanding channel dynamics, video popularity, and user engagement.



---

## Tools and Libraries Used


__GOOGLE API CLIENT:__ The googleapiclient library in Python facilitates the communication with different Google APIs. Its primary purpose in this project is to interact with YouTube's Data API v3, allowing the retrieval of essential information like channel details, video specifics, and comments. By utilizing googleapiclient, developers can easily access and manipulate YouTube's extensive data resources through code.

![Extracting Google API](https://github.com/Jeevith23/YOUTUBE-DATA-HARVESTING-TASK/assets/139576422/5c3f9dee-a28e-48fc-9f94-a61428a12388)

__YOUTUBE DATA SCRAPPING AND ITS ETHICAL PERSPECTIVE:__ When engaging in the scraping of YouTube content, it is crucial to approach it ethically and responsibly. Respecting YouTube's terms and conditions, obtaining appropriate authorization, and adhering to data protection regulations are fundamental considerations. The collected data must be handled responsibly, ensuring privacy, confidentiality, and preventing any form of misuse or misrepresentation. Furthermore, it is important to take into account the potential impact on the platform and its community, striving for a fair and sustainable scraping process. By following these ethical guidelines, we can uphold integrity while extracting valuable insights from YouTube data.

![Youtube API reference](https://github.com/Jeevith23/YOUTUBE-DATA-HARVESTING-TASK/assets/139576422/4e1b80fd-063f-47eb-8bea-0b8621d6bf6b)
__MONGODB:__ MongoDB is built on a scale-out architecture that has become popular with developers of all kinds for developing scalable applications with evolving data schemas. As a document database, MongoDB makes it easy for developers to store structured or unstructured data. It uses a JSON-like format to store documents.

__POSTGRESQL:__ PostgreSQL is an open-source, advanced, and highly scalable database management system (DBMS) known for its reliability and extensive features. It provides a platform for storing and managing structured data, offering support for various data types and advanced SQL capabilities.


__PYTHON:__ Python is a powerful programming language renowned for being easy to learn and understand. Python is the primary language employed in this project for the development of the complete application, including data retrieval, processing, analysis, and visualisation.

__STREAMLIT:__ 
 Streamlit library was used to create a user-friendly UI that enables users to interact with the programme and carry out data retrieval and analysis operations.

 ---

## REQUIRED LIBRARIES:

- googleapiclient.discovery

- streamlit

- psycopg2

- pymongo

- pandas

---
## *Work Pipeline*
- Extract the Google API key from **Google Developer Console** YouTube data Extraction
- Use the **References** in the YouTube API to know about the data.
- YouTube Data to Extract
    - Content Details
    - Channel id
    - Snippets
    - Statistics
- Use the **MongoDB cloud** to insert the acquired data , which is in **JSON** format.
- Connect it to the Database.
- **Python** code for the Analysis
- Connect to the **PostgreSQL** Database
- Streamlit application to view the output

---
## Brief Of Python Code

**Importing Libraries:**
- *pymongo*: Python driver for MongoDB, used for interacting with MongoDB.
- *psycopg2*: PostgreSQL adapter for Python, used for connecting to PostgreSQL.
- *pandas*: A data manipulation library.
- *streamlit*: Streamlit framework for building web applications.
- *googleapiclient.discovery*: Google API client library for interacting with the YouTube Data API.

``` python3
import pymongo
import psycopg2
import pandas as pd
import streamlit as st
from googleapiclient.discovery import build

```
---

## YouTube API Connection:

``` python3
def Api_connect():
    Api_Id = "AIzaSyBMUD52Jov6a1LBkSCvZvyay4Ih0-7GPPk"
    api_service_name = "youtube"
    api_version = "v3"
    youtube = build(api_service_name, api_version, developerKey=Api_Id)
    return youtube

youtube = Api_connect()

```


- `Api_connect()`: Function to establish a connection to the YouTube Data API using the API key.
- The API key (Api_Id) is generated from the Google Cloud Console.
- The build function is used to create a connection to the YouTube API.

---

## Function to Get Channel Information:
``` python3

# get playlist ids
def get_playlist_info(channel_id):
    All_data = []
    next_page_token = None
    next_page = True

    while next_page:
        request = youtube.playlists().list(
            part="snippet,contentDetails",
            channelId=channel_id,
            maxResults=50,
            pageToken=next_page_token
        )
        response = request.execute()

        for item in response['items']:
            data = {
                'PlaylistId': item['id'],
                'Title': item['snippet']['title'],
                'ChannelId': item['snippet']['channelId'],
                'ChannelName': item['snippet']['channelTitle'],
                'PublishedAt': item['snippet']['publishedAt'],
                'VideoCount': item['contentDetails']['itemCount']
            }
            All_data.append(data)

        next_page_token = response.get('nextPageToken')
        if next_page_token is None:
            next_page = False

    return All_data

youtube = Api_connect()
```

- `get_channel_info(channel_id)`: Function to retrieve information about a YouTube channel.
- Uses the `channels().list` API endpoint to fetch details such as channel name, ID, subscription count, views, total videos, description, and playlist ID.
- Returns a dictionary containing the extracted information.

## Function to Get Channel,Video, Comment, Playlist IDs
```python3
# get video information
def get_video_info(video_ids):
    video_data = []

    for video_id in video_ids:
        request = youtube.videos().list(
            part="snippet,contentDetails,statistics",
            id=video_id
        )
        response = request.execute()

        for item in response["items"]:
            data = dict(
                Channel_Name=item['snippet']['channelTitle'],
                Channel_Id=item['snippet']['channelId'],
                Video_Id=item['id'],
                Title=item['snippet']['title'],
                Tags=item['snippet'].get('tags'),
                Thumbnail=item['snippet']['thumbnails']['default']['url'],
                Description=item['snippet']['description'],
                Published_Date=item['snippet']['publishedAt'],
                Duration=item['contentDetails']['duration'],
                Views=item['statistics']['viewCount'],
                Likes=item['statistics'].get('likeCount'),
                Comments=item['statistics'].get('commentCount'),
                Favorite_Count=item['statistics']['favoriteCount'],
                Definition=item['contentDetails']['definition'],
                Caption_Status=item['contentDetails']['caption']
            )
            video_data.append(data)
    return video_data

```

## Function to Connect to the MongoDB and SQL

```python3
# MongoDB Connection
client = pymongo.MongoClient("mongodb+srv://vignesh:vigneshd@vignesh.vrsd7ro.mongodb.net/?retryWrites=true&w=majority")
db = client["Youtube_data"]

# Upload to MongoDB
def channel_details(channel_id):
    ch_details = get_channel_info(channel_id)
    pl_details = get_playlist_info(channel_id)
    vi_ids = get_channel_videos(channel_id)
    vi_details = get_video_info(vi_ids)
    com_details = get_comment_info(vi_ids)

    coll1 = db["channel_details"]
    coll1.insert_one({
        "channel_information": ch_details,
        "playlist_information": pl_details,
        "video_information": vi_details,
        "comment_information": com_details
    })

    return "Upload completed successfully"

# Table creation for channels, playlists, videos, comments
def channels_table():
    mydb = psycopg2.connect(
        host="localhost",
        user="postgres",
        password="vigneshd",
        database="youtube_data",
        port="5432"
    )
    cursor = mydb.cursor()

    drop_query = "DROP TABLE IF EXISTS channels"
    cursor.execute(drop_query)
    mydb.commit()

    try:
        create_query = '''CREATE TABLE IF NOT EXISTS channels(
            Channel_Name VARCHAR(100),
            Channel_Id VARCHAR(80) PRIMARY KEY,
            Subscription_Count BIGINT,
            Views BIGINT,
            Total_Videos INT,
            Channel_Description TEXT,
            Playlist_Id VARCHAR(50)
        )'''
        cursor.execute(create_query)
        mydb.commit()
    except:
        st.write("Channels Table already created")

    ch_list = []
    db = client["Youtube_data"]
    coll1 = db["channel_details"]
    for ch_data in coll1.find({}, {"_id": 0, "channel_information": 1}):
        ch_list.append(ch_data["channel_information"])
    df = pd.DataFrame(ch_list)

    for index, row in df.iterrows():
        insert_query = '''INSERT INTO channels(
            Channel_Name,
            Channel_Id,
            Subscription_Count,
            Views,
            Total_Videos,
            Channel_Description,
            Playlist_Id
        ) VALUES(%s,%s,%s,%s,%s,%s,%s)'''

        values = (
            row['Channel_Name'],
            row['Channel_Id'],
            row['Subscription_Count'],
            row['Views'],
            row['Total_Videos'],
            row['Channel_Description'],
            row['Playlist_Id']
        )
        try:
            cursor.execute(insert_query, values)
            mydb.commit()
        except:
            st.write("Channels values are already inserted")

# Similar functions for playlists_table, videos_table, and comments_table...

# Function to create all tables
def tables():
    channels_table()
    playlists_table()
    videos_table()
    comments_table()
    return "Tables created successfully"

# Streamlit Sidebar and UI components...

# SQL connection
mydb = psycopg2.connect(
    host="localhost",
    user="postgres",
    password="vigneshd",
    database="youtube_data",
    port="5432"
)
cursor = mydb.cursor()

# SQL queries based on user questions...

# Streamlit Application code...

# Run the Streamlit app
if __name__ == "__main__":
    st.run_app() 
```

- **MongoDB Connection:**

    - Establishes a connection to a MongoDB database using the MongoClient.
Uses the Youtube_data database in the MongoDB Atlas cluster.
Upload to MongoDB:
![Storing in data lake](https://github.com/Jeevith23/YOUTUBE-DATA-HARVESTING-TASK/assets/139576422/c80c3aec-4634-4789-a32f-a3687147e4f3)


- `channel_details(channel_id)`: Function to upload channel details, playlist information, video information, and comment information to MongoDB.
Table Creation for Channels, Playlists, Videos, and Comments:

- `channels_table(), playlists_table(), videos_table(), and comments_table()`: Functions to create tables in a PostgreSQL database for storing channels, playlists, videos, and comments.
SQL Connection:

**Establishes a connection to a PostgreSQL database. SQL Queries:**

Based on user-selected questions, SQL queries are executed to retrieve and display relevant information.
Streamlit Application:

The Streamlit app is designed to showcase tables, allow user input for data collection, migration to SQL, and display SQL query results.


![Migrating to SQL warehouse](https://github.com/Jeevith23/YOUTUBE-DATA-HARVESTING-TASK/assets/139576422/0baf0835-db9a-40f8-96fb-deaa4ef2e682)

## Features
- Harvests Data: Fetches channel information, playlists, videos, and comments from YouTube.
- Dual Storage: Stores data in both MongoDB and PostgreSQL databases.
- Streamlit Interface: Uses Streamlit for a user-friendly web interface.
- SQL Queries: Performs SQL queries to retrieve insightful information from the collected data.

## USER INTERFACE
Overall Looks:
![UI](https://github.com/Jeevith23/YOUTUBE-DATA-HARVESTING-TASK/assets/139576422/3a180615-6832-454a-ae37-61a39e4363a1)
![CLOSELOOK UI](https://github.com/Jeevith23/YOUTUBE-DATA-HARVESTING-TASK/assets/139576422/c98479b8-aa06-4c10-923e-59fcbdf66fed)


## Usage
- Enter `Channel IDs`:

    Enter the YouTube Channel IDs in the Streamlit app.
- Collect and Store Data:

    Click the `Collect and Store Data` button to fetch and store data.
- Migrate to SQL:

    Click the `Migrate to SQL` button to transfer data to the SQL database.
- Explore Data with Streamlit:

    Use the Streamlit interface to explore SQL queries and visualize data.

## SQL Queries
The application supports various SQL queries to analyze and extract valuable insights from the stored data. Examples include:
![SQL Queries](https://github.com/Jeevith23/YOUTUBE-DATA-HARVESTING-TASK/assets/139576422/bd3007a4-ec0b-4f5a-b0a4-387d5d9b4351)

## Points to remember
- Create a Virtual Environment in Python IDE for better use.
- Ensure the MongoDB is connected
- Make sure the VPN is `off` this would results in various error.
- Connect the SQL Database with proper password 
- And before Running the application delete the duplicate tables.
- Make sure MongoDB Dont have any duplicate database.
- If the code doesnt run automatically
    - Then open Command Prompt and "streamlit run <filename.py>"

---
## Additional Information
For more information on Streamlit, MongoDB, and PostgreSQL, refer to the respective documentation:

- Streamlit Documentation
- MongoDB Documentation
- PostgreSQL Documentation
--- 
## License
This project is licensed under the MIT License.

    Followed the coding standards: https://www.python.org/dev/peps/pep-0008/













