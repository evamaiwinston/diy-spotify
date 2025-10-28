# This is a DIY Spotify Project for Intro to Computing

In this project I built a web-based music player that resembles Spotify, but contains my favorite songs/covers that aren't availble on Spotify. 

In this project:
- I organized and created data files according to a schema
- Ingested the data using cloud-native techniques
- Stored each song's metadata in a relational database
- Exposed that data in an API endpoint


Much of the infrastructure for this project is created with an Amazon CloudFormation template.

![AWS Diagram](https://s3.amazonaws.com/nem2p-dp1-spotify/diagram.png)

#### S3 Bucket

The CF template created an S3 bucket based on my UVA computing ID, plus `-dp1-spotify`. 
This bucket is configured with a few special options:

- All objects will be visible to the public.
- The bucket will be configured as a website.
- The bucket allows instructional staff to upload objects.

Each song is uploaded with a 3-file bundle associated with it: The MP3 song file, a JSON metadata file, and a JPG album image.


#### Lambda Function

To manually create a Lambda function, I used Chalice, a Python3 library. 
Chalice creates a Lambda function associated with my bucket, and run the code each time a new `.json` object is uploaded.

Chalice configures the bucket-to-function connection, and can be easily deployed and re-deployed.

#### Database Service

The RDS database service was provided by our class's instructor. 
I did not to create the database server myself, but I created a new database and all the necessary tables within it.

#### EC2 Instance

The CF template created an EC2 instance with the following features configured/installed:

- Docker
- Fixed IP address (ElasticIP)
- Port 80 visible to the Internet [`0.0.0.0/0`]

#### FastAPI

I had previously been working on a FastAPI container in class. 
I made two important additions to my API:

1. MySQL connection libraries
2. A new `/songs` endpoint using the **GET** method that lists song metadata

- - -

### Data Flow 

These are the steps that  occur when a new song is added to the music player:

1. The song file bundle (3 files) is uploaded to the S3 bucket.
2. The arrival of a new `.json` file triggers Lambda function to execute.
3. Lambda function retrieves the JSON metadata file and parses it. It then calculates the name of the MP3 and JPG files associated with the song metadata, and generates the full S3 URI to each of those files.
4. It performs a MySQL `INSERT` query to add the new song to my `songs` database. These are the fields it inserts:
   - `title` - The song title (string)
   - `artist` - The musical artist (string)
   - `album` - The album containing the song (string)
   - `genre` - The index of the genre (integer)
   - `year` - The year of the song (integer)
   - `file` - The full S3 URI to the MP3 file
   - `image` - The full S3 URI to the JPG image
5. Data inserted into the `songs` table of my database are immediately available in the API's `/songs` endpoint as a `GET` method. 
6. The song player's frontend (S3 bucket website, with the `index.html` added) can then be refreshed to display the new song after ingestion.

   > Note: I’m not hosting this project publicly (no paid S3/static site URL). For copyright reasons, this project was for academic and temporary use only.
