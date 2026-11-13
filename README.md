# spotify-listening-history-analysis
### **Overview** 
This is a SQL analysis of my personal Spotify streaming history going back to 2018. I will be exploring my listening habits and trends.


### **Tools Used:** 
MS Excel \
SQL Server Management Studio 22


### **Data Discovery** 
Spotify is legally required to give you access to all your personal data they've collected. I went to the Spotify 'Privacy' page and then clicked 'Download Your Data'. A week or two later, I received an email from Spotify with my Extended Streaming History data files. I downloaded the ZIP files, and found the data files in JSON format broken up year by year. I converted the streaming_history_audio files from JSON format to CSV using Excel. \
\
In Excel, I followed these steps to import each JSON file to Excel:
1. Get Data > From File > From JSON > Streaming_History_Audio_20XX
2. Select ‘To table’ in power query editor
3. Expand the column header to select fields I want to retain. \
   Columns I will be using: ts, ms_played, master_metadata_track_name, master_metadata_artist_album_name, master_metadata_album_name, spotify_track_uri, reason_start, reason_end, shuffle, skipped, offline, offline_timestamp
4. Close and Load to Table in existing worksheet

### **Data Cleaning**
- Align all columns to the left
- Autofit column width so all data is fully visible
- Rename column headers to be easily understood
- Convert 'offline_timestamp' units to Unix timestamp in seconds. 
- Ensure all data files imported are being filtered together

All 9 JSON data files were combined to 1 Excel worksheet in CSV format called 'straming_history', with 12 columns and 37,712 rows after being cleaned. 
\
\
**Load Data to MS SQL Server** 
1. Create a database called `SpotifyStreamingHistory`.
2. Create a table called `streaming_history_staging` with 12 fields, assigning the correct data type to each field.
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
3. Import the `streaming_history` CSV using Tasks > Import Flat File. Specify the file location, table name (`streaming_history_staging`), preview the data, and import it.
4. Run a SELECT query to verify the data loaded correctly.
   ```sql
   SELECT * FROM dbo.streaming_history_staging;
   ```

### **Analyze the Data** 
This query sums up the top songs I've spent the most time listening to, sorting them in descending order. \
\
The 'GROUP BY' clause groups rows with the same values (the track and artist names), so the aggregate function (the sum of msPlayed) can be applied to each group seperately. 
```sql
SELECT TOP 5 trackName, artistName, SUM(msPlayed) /60000 AS TotalMinutes FROM dbo.streaming_history_staging
WHERE trackName IS NOT NULL
GROUP BY trackName, artistName
ORDER BY TotalMinutes DESC;
```
### My Top 5 Most Played Songs of All Time
| Rank | Track | Artist | Total Minutes |
|---:|---|---|---:|
| 1 | Breakfast At Tiffany's | Deep Blue Something | 401 |
| 2 | Smells Like Teen Spirit | Nirvana | 391 |
| 3 | Cigarette Daydreams | Cage The Elephant | 384 |
| 4 | Trouble | Cage The Elephant | 335 |
| 5 | On Top | The Killers | 321 |

Thinking back, this does make sense. I used to LOVE The Killers and Cage The Elephant when I was 16. I useed to overplay the same songs again and again at work. \
\
I do vaguely remember Breakfast At Tiffany's being one of the first songs I ever added to a playlist on Spotify. I believe at that time, I hadn't found the recommended songs algorithm and would just listen back to the same playlist over and over again. \
\
When I was 19, I got really into Nirvana. I remember I would solely listen to Nirvana albums on my walk back home from the restaurant I worked at. Of course, Smells Like Teen Spirit always came back to me. I did have a strong bias to the song Lithium off the Nevermind album. I'm surprised I don't see that one up there. 


### **My Top Songs in 2018** 
Going as far back as my listening history takes me, we arrive in 2018. For context, I was 16-17 years old, had just downloaded Spotify, and was probably acting like an angsty teen listening to music as I stocked groceries after school. \
\
This query isolates the top 3 songs I spent the most time listening to in the year 2018.
```sql
SELECT TOP 3 trackName, artistName, SUM(msPlayed) /60000 AS TotalMinutes FROM dbo.streaming_history_staging
WHERE trackName IS NOT NULL AND timestamp LIKE '2018%'
GROUP BY trackName, artistName
ORDER BY TotalMinutes DESC;
```
| Rank | Track | Artist | Total Minutes |
|---:|---|---|---:|
| 1 | CIA | New Politics | 89 |
| 2 | Spirits | The Strumbellas | 78 |
| 3 | Breakfast At Tiffany's | Deep Blue Something | 75 |

I would describe these songs as belonging to the alternative/indie pop-rock genre. It was like dipping a toe into the alt music space with upbeat and catchy tracks that weren't particularly aggressive. 

### **My Top Songs in 2021**
Let's skip forward a few years to 2021 and see if my music taste has diversified at all. At this time, I was around 19-20 years old and was working at a resturant/bar as a waitress while commuting to my local university. Times had indeed changed for me, so I'm sure my music taste had as well. \
\
I'm using the same query as before, but with the LIKE operator with the % wildcard to include timestamps of songs I listened to only in 2021.
```sql
SELECT TOP 3 trackName, artistName, SUM(msPlayed) /60000 AS TotalMinutes FROM dbo.streaming_history_staging
WHERE trackName IS NOT NULL AND timestamp LIKE '2021%'
GROUP BY trackName, artistName
ORDER BY TotalMinutes DESC;
```
| Rank | Track | Artist | Total Minutes |
|---:|---|---|---:|
| 1 | Shine On You Crazy Diamond (Pts. 1-5) | Pink Floyd | 81 |
| 2 | Heart-Shaped Box | Nirvana | 79 |
| 3 | CORALINE | Måneskin | 75 |

My music taste appears to have shifted to more progressive and heavier rock, while still staying in the alternative/grunge space. 90's music is one of my favorite eras of music, giving us so many hits from the likes of Nirvana, Sonic Youth and Hole, so I'm not surprised a Nirvana track made the list. Pink Floyd's music is timeless, so it's no wonder it was my top song this year. I also listened to a lot of Måneskin this year, right after the band blew up after winning the 2021 Eurovision Song Contest. I think it's also worth noting that my attachment to my favorite songs didn't seem to change much between 2018 and 2021. My top three songs accumulated almost exactly the same amount of listening time in both years. \

### **My Top Songs in 2025**
We finally arrive at 2025, the latest full calendar year of my listening history at the time of this analysis. A few years have passed again, making me 23-24 years old this year. During this time, I completed my final year of university, and started my first full-time role. 
```sql
SELECT TOP 3 trackName, artistName, SUM(msPlayed) /60000 AS TotalMinutes FROM dbo.streaming_history_staging
WHERE trackName IS NOT NULL AND timestamp LIKE '2025%'
GROUP BY trackName, artistName
ORDER BY TotalMinutes DESC;
```
| Rank | Track | Artist | Total Minutes |
|---:|---|---|---:|
| 1 | Change (In the House of Flies) | Deftones | 143 |
| 2 | King For A Day | Pierce The Veil | 133 |
| 3 | Kids | Current Joys | 128 |

Fast forward to 2025, it appears my music taste is leaning heavier into the alternative music scene reflecting my love for darker and more emotional music. But, it also looks like I'm still enjoying my fair share of melodic music with Current Joys. I spent 67% more listening time on these 3 top tracks compared to the previous years we looked at. I seem to have developed an even stronger attachment to certain songs. At first I believed that this may have been due to a increase of listening time overall with of my purchase of Spotify Premium Ad-free music this year.
I constructed this query to test this theory, looking my total listening hours per year.
```sql
SELECT YEAR(timestamp) AS Years, FORMAT(SUM(msPlayed) / 3600000.0, '0.##') AS TotalHours FROM dbo.streaming_history_staging
GROUP BY YEAR(timestamp)
ORDER BY YEAR(timestamp)
```
**My Total Listening Hours Per Year**
| Year | Total Listening Hours |
|---:|---:|
| 2018 | 36.31 |
| 2019 | 441.49 |
| 2020 | 292.48 |
| 2021 | 185.01 |
| 2022 | 154.13 |
| 2023 | 183.48 |
| 2024 | 274.16 |
| 2025 | 103.33 |
| 2026 | 59.64 |

It actually appears that my listening hours decreased in 2025 compared to previous years.


