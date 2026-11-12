# spotify-listening-history-analysis
**Overview** \
This is a SQL analysis of my personal Spotify streaming history going back to 2018. I will be exploring my listening habits and trends.
\
\
**Tools Used:** \
MS Excel \
SQL Server Management Studio 22
\
\
**Data Discovery** \
Spotify is legally required to give you access to all your personal data they've collected. I went to the Spotify 'Privacy' page and then clicked 'Download Your Data'. A week or two later, I received an email from Spotify with my Extended Streaming History data files. I downloaded the ZIP files, and found the data files in JSON format broken up year by year. I converted the streaming_history_audio files from JSON format to CSV using Excel. \
\
In Excel, I followed these steps to import each JSON file to Excel:
1. Get Data > From File > From JSON > Streaming_History_Audio_20XX
2. Select ‘To table’ in power query editor
3. Expand the column header to select fields I want to retain. \
   Columns I will be using: ts, ms_played, master_metadata_track_name, master_metadata_artist_album_name, master_metadata_album_name, spotify_track_uri, reason_start, reason_end, shuffle, skipped, offline, offline_timestamp
4. Close and Load to Table in existing worksheet

**Data Cleaning**
- Align all columns to the left
- Autofit column width so all data is fully visible
- Rename column headers to be easily understood
- Convert 'offline_timestamp' units to Unix timestamp in seconds. 
- Ensure all data files imported are being filtered together

All 9 JSON data files were combined to 1 Excel worksheet in CSV format called 'straming_history', with 12 columns and 37,712 rows after being cleaned. 
\
\
**Load Data to MS SQL Server** 
1. Create a database called 'SpotifyStreamingHistory'
2. Create a table called 'streaming_history_staging' with 12 fields, assigning the correct data type to each field.
  ```sql
  CREATE TABLE streaming_history_staging (
	[timestamp] NVARCHAR(100),
	msPlayed INT,
	trackName NVARCHAR(MAX),
	artistName NVARCHAR(MAX),
	albumName NVARCHAR(MAX),
	trackUri NVARCHAR(500),
	reasonStart NVARCHAR(100),
	reasonEnd NVARCHAR(100),
	shuffle NVARCHAR(20),
	skipped NVARCHAR(20),
	offline NVARCHAR(20),
	offline_timestamp NVARCHAR(50)
  );
  ```
4. Import the 'streaming_history' CSV by right clicking the 'SpotifyStreamingHistory' database > Tasks > Import Flat File > specify location of 'streaming history' file > specify table name 'streaming_history_staging' > Preview data > Import data to new table
5. Run a SELECT query to verify data is loaded correctly
   ```sql
   SELECT * FROM dbo.streaming_history_staging
   ```
\
**Analyze the Data** \









