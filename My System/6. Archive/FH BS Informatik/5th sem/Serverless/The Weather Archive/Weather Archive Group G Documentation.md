# 1. Introduction and Goals

This document describes the Weather Archive - a web application that serves weather data from a particular city. The end user enters the city and specifies the desired date and hour. The system delivers several pieces of information about the weather during this particular period of time. Firstly, the system delivers a video, which is a compilation of pictures takes by the webcam in this city during the specified hour. Secondly, the system delivers a bar chart with average weather metadata, e.g. temperature, humidity, air pressure, collected during this hour.

The system was built primarily utilizing the serverless architecture with multiple moving parts and components described below.
## 1.1 Requirements Overview

The following requirements in the form of user stories for different parts of the system were specified by the client. First, a rough sketch of the architecture diagram:

![[Rough sketch.png]]

The following user stories for the proposed components of the system were established by the client:
1. As a user, I want to navigate to “The Weather Archive” application in the browser and see a logo and a search bar with a button. (similar to the Google Homepage). 
2. As a user, I would like to type into the search bar and to retrieve autocomplete results (topics e.g. cities) in a list. 
3. As a user, I am only able to choose topics (e.g. cities) from the autocomplete. 
4. As a user, I want to retrieve an error message when I am searching for a topic that is not in the autocomplete list. 
5. As a user, I would like to be taken to a new site when I select a valid topic from the autocomplete and click on the search button. 
6. As a user, I would like to see a heading (topic, e.g. Vienna), a video (a compilation of the time period), a plot of the metadata (temperature, humidity, air pressure - of that time period), a date-picker, and a slider to select the hour of the day on the new page.
7. As a user, I am only able to select dates that are available. 
8. As a user, I can only select hours that are available for the selected date. 
9. As a user, I want to see the logo of “The Weather Archive” on the new page so that I can navigate back to the homepage when I click on it. 
10. As a user, I would like to receive good (human-readable) error messages when something goes wrong. 
11. As a webcam, I would like to be able to communicate via a RESTful Interface to POST data (an image, the topic, and metadata) to the API. 
12. As a webcam, I want to use an API key when communicating with the API to ensure only authenticated users can POST data. 
13. As a webcam, I would like to receive good error codes when something goes wrong. 
14. As API, I would like to be able to store an image in an object storage. 
15. As API, I want to be able to store data (image location, topic, timestamps, metadata) in a database 
16. As API, I would like to be able to retrieve the data from the database 
17. As API, I would like to be able to retrieve data from the S3 Bucket 
18. As API, I want to be able to store data in the Redis Cache 
19. As API, I would like to be able to retrieve data from the Cache 
20. As API, I will not access the DB if the requested topic and time period is already in the Redis cache 
21. As API, I will return the cached results if the requested topic and time period is already in the Redis cache 
22. As PictureService, I want to get notified when a new entry was made to the database (e.g. image location) 
23. As PictureService, I would like to be able to retrieve data from the S3 Bucket 
24. As PictureService, I would like to be able to store data in the S3 Bucket 
25. As PictureService, I want to be able to resize an image (compress) 
26. As VideoService, I would like to be scheduled for a specific time period 
27. As VideoService, I would like to be able to read the database 
28. As VideoService, I would like to be able to create a video based on the images of that topic (that were sent from the webcam). 
29. As VideoService, I will only create a video per time period (so a picture that was already used in a past video will not be used for the new schedule). 
30. As VideoService, I want to be able to store a video in a S3 Bucket"
## 1.2 Quality Goals
Out of all the requirements, the following three priority goals are defined and should be adhered to and referenced when implementing any functionality of the system:
1. **Reliability & Availability**
    - The application must reliably store and serve images, videos, and weather data without data loss or frequent downtime.
    - The architecture uses AWS serverless approaches and distributed storage (S3, RDS, Redis) to minimize single points of failure.
2. **Scalability & Performance**    
    - The system should handle sudden increases in the number of uploaded images (webcams posting every 5 minutes across many cities) and user requests (peak traffic for watching videos).
    - Redis caching for repeated queries, S3 for static assets, and a serverless, event-driven design for horizontal scalability.
3. **Maintainability & Ease of Extensions**
    - The architecture must be straightforward to update with new features (e.g., more cities, extra metadata).
    - Using Lambdas for discrete tasks (upload, compression, video generation) encapsulates logic in small, maintainable services.

## 1.3 Stakeholders

Each stakeholder influences certain parts of the architecture:

| FStakeholder                                      | Role / Concern                                                                                                                |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Client / Product Owner                            | Provides business requirements. Expects robust and user-friendly weather archive. Focused on budget, timelines, key features. |
| End Users (City Visitors, Weather Enthusiasts)    | Interact via React frontend. Want quick responses, accurate data, easy date/time picking, watch videos.                       |
| Webcam Owners (technicians or system integrators) | Manage physical webcams. Need an API to push images & metadata. Concerned about API reliability, error codes.                 |
| Development Team                                  | Implement Lambdas, DB schemas, frontend React, DevOps. Need maintainable architecture with minimal friction.                  |
# 3. Context and Scope
## 3.1 Business Context

From a **business** perspective, the system operates as follows:
- **Webcam** devices (or their owners) upload an image + weather metadata (temperature, humidity, air_pressure) every 5 minutes via a REST endpoint (`POST /upload`).
- The **Weather Archive** then compresses and stores images for historical retrieval, and creates a video compilation of received images for each hour.
- **Users** visit the web interface (React app) to see a city’s historical data in the form of a short video plus average weather stats for that hour.

Communication Partners on domain-level:

| **Partner**       | **Inputs**                                                                                  | **Outputs**                                                                                  |
| ----------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Webcams           | Send `POST /upload` with an image (JPEG) + metadata (topic, temperature, humidity, etc.)    | Receive JSON responses confirming success/failure, error codes                               |
| End Users         | Use a React-based site to select city, date, hour                                           | Receive embedded video (from S3) & bar chart data (via JSON endpoint  `getWeather`)          |
| S3 (object store) | Expects PUT/GET operations for storing/retrieving original/compressed images & final videos | Provides object URLs that link to the compressed images and generated video mp4 files        |
| RDS (Postgres DB) | Receives SQL queries to store (INSERT) or retrieve (SELECT) images and videos metadata      | Returns record sets (rows) with metadata such as `compressed_link`, `video_url`, `timestamp` |
| Redis (cache)     | Expects read/write requests (e.g. `GET weather:Vienna:2025-01-04:03`)                       | Provides cached JSON results or empty if not cached                                          |
## 3.2 Technical Context
This sections gives a brief overview of the technical interfaces present in the system.

### webcamService Lambda
This is supposed to be an external part of the system that is currently mocked to showcase functionality. Every 5 minutes, using AWS EventBridge, the `webcamService Lambda` is triggered for each of the 4 existing topics/cities: Vienna, Berlin, Paris, London. Meaning, 4 invocations every 5 minutes.
The `webcamService Lambda` sends a POST /upload API request, supplying the intended topic, metadata (temperature, humidity, air_pressure) and the image.

### uploadImage Lambda
Stands behind the API Gateway for POST /upload. Its purpose is to receive the data from the `webcamService Lambda` and store the metadata in RDS database and the image under non-compressed-images/ in S3 bucket (more details is section 5).

### compressImage Lambda
Gets triggered when a new object appears in the S3 bucket under non-compressed-images/. compressImage Lambda downloads the original image and compresses it using the *sharp* JS library. Then, it uploads the compressed image to the same bucket under /compressed-images and updates the DB column *compressed_link* for this image.

### generateVideo Lambda
Gets triggered 5 minutes after the beginning of each hour (e.g. at 14:05, 15:05...). Its purpose is to fetch compressed images that were uploaded during the last hour, create a video compilation, and upload the video to S3 and create a new record in the *videos* table.

### checkAvailableVideos Lambda
Stands behind the API Gateway for GET /checkAvailableVideos and expects a *topic* as a query parameter. This lambda delivers a list of dates corresponding hours for which a video is created and available. This way, the user can only pick a timestamp for which a video exists.

### getWeather Lambda
Stands behind the API Gateway for GET /getWeather and expects *topic*, *date*, and *hour* as query parameters. This Lambda delivers a video url where the corresponding video for these parameters is stored, and the metadata about all the images that were included in this video.

### Frontend (React on EC2)
Calls `checkAvailableVideos` Lambda when the user picks a topic, and `getWeather` Lambda when the user wants to retrieve the video and the metadata.

### AWS ElastiCache on Redis OSS
Is accessed internally by `getWeather` Lambda. In case a combination of topic + date + hour exists in the cache, it immediately returns it and skips the DB query. In cache the combination is not found in cache, it fetches it from the DB and stores in the cache as well. Cache expires after one hour.

# 4. Solution Strategy

Key solutions: 
- AWS services: 
	- **S3** for object storage - `weather-archive-images-if22b253` bucket that stores internal non-compressed, compressed images, and videos; `webcam-image-storage-if22b253` bucket that stores external pictures of the cities that webcams (`webcamService Lambda`) fetch and send to the system.
	- **Lambda** for compute - the entire system is built on Lambdas and event-driven workflows, excluding the EC2 instance for frontend.
	- **RDS** for managed PostgreSQL - needed and accessed by multiple Lambdas to store metadata, links, timestamps etc.
	- **ElastiCache Redis OSS** - serverless caching.
- `compressImage Lambda`: the usage of the *Sharp* JS library for on-the-fly image compression
- ffmpeg layer in `generateVideo Lambda` for MP4 creation
- Caddy Webserver + EC2 for the React frontend with a self-signed certificate

| **Goal**                            | **Approach**                                                                                                                                                             | **Related Sections**                 |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------ |
| Reliability                         | Use S3 for indefinite image storage, RDS, Lambdas for discrete tasks, always running frontend EC2.                                                                       | 5. Building Block View, 9. Decisions |
| Scalability                         | Serverless-based design (Lambdas scale horizontally), store static artifacts in S3, Redis caching for repeated queries.                                                  | 9. Decisions                         |
| Maintainability & Ease of Extension | Each Lambda has a single responsibility, React frontend is separate instane.                                                                                             | 5. Building Block View               |
| User Friendliness & Troubleshooting | Lambda API responses contain explicit, detailed, human-readable messages and error codes. Frontend handles edge cases with understandable error messages.                | 9. Decisions                         |
| Automated Image & Video Processing  | S3 event triggers `compressImage` automatically, scheduled event triggers `generateVideo`, scheduled webcam triggers - the system work autonomously and supports itself. | 5. Building Block View               |

# 5. Building Block View

## 5.1 Overall White Box (Level 1)

To understand an event-driven system of this kind on a deeper level, it is necessary to take a closer look at the separate workflows present here. In total there are three: Webcam Upload Workflow, Video Generation Workflow, and End User Workflow.

### Webcam Upload Workflow

![[Pasted image 20250107223607.png]]

As depicted on the diagram, `webcamService Lambda` initiates the Webcam Upload Workflow. Every 5 minutes, AWS EventBridge triggers the `webcamService Lambda` which sends a POST request with the image and metadata to the API Gateway. From there, uploadImage Lambda stores the image metadata in RDS database and the non-compressed image in S3 bucket. This triggers `compressImage Lambda` that fetches that image, compresses it, updates the S3 bucket with a new version of this image now under /compressed folder, and updates the RDS row to include a link to the new location of the compressed image.

### Video Generation Workflow

![[Screenshot 2025-01-07 at 22.46.19.png]]

AWS EventBridge triggers `generateVideo Lambda` on the 5th minute of each hour. More specifically, it triggers the execution of 4 `generateVideo Lambdas` - one for each topic i.e. city. The video is stored in S3 bucket, and its additional information is stored in RDS. 

### End User Workflow

![[Screenshot 2025-01-07 at 22.52.51.png]]

The end user accesses the system via frontend interface. First, the user picks a topic, which invokes the `checkAvailableVideos Lambda`. This Lambda delivers the dates and respective hours for which the video generated by `generateVideo Lambda` is available.
The user can picks only the available dates and hours. Once the date & hour combination is picked, `getWeather Lambda` is being called. This Lambda delivers the video link and weather metadata for pictures received on the picked date during the picked hour.
## 5.2 Lambdas (Level 2)

Certain Lambdas from those workflows require careful attention as they perform multiple steps and calculations. 
### generateVideo Lambda from Video Generation Workflow

High-Level Purpose: Periodically creates a video (MP4) from the compressed webcam images of a particular city i.e. topic for a given hour.

**General Flow**

1. **Determine Hour to Process**:
    
    - Reads a `topic` from the event
    - Looks at the current time, subtracts one hour.
    - Prepares local file paths and S3 keys based on date/hour.
2. **Fetch Relevant Images**:
    
    - Retrieves from the `images` table only those entries whose `compressed_link` is set (i.e., already compressed).
    - If none found, the Lambda logs a “no images” message and exits.
3. **Concatenate & Generate Video**:
    - Downloads each compressed image to temporary storage.
    - Uses a “file list” technique to build a short time-lapse video (2 seconds per image).
    - Runs a command-line tool (ffmpeg) to stitch images → MP4 file.
    
4. **Store Final Video**:    
    - Uploads the resulting MP4 to S3 under a `videos/` prefix.
    - Inserts a record into the `videos` table with the resulting URL, so the system knows a video now exists for that hour.

Key Points

- Scheduled hourly
- Pivot: If no images, it gracefully returns without generating anything
- Avoids re-processing older images by focusing on a single hour

### checkAvailableVideos Lambda from End User Workflow

High-Level Purpose: Lists all “(date, hour)” combinations that currently have a compiled video for a city. The frontend calls this to enable/disable date pickers or hour sliders.

**General Flow**

1. **Input**: A `topic` (city name).
2. **SQL Query**:
    - Select from the `videos` table (which holds each generated video’s timestamp).
    - Group results by date + hour so the frontend can see which are valid.
3. **Return**:
    - An array of objects like `{ date: "2025-01-07", hours: [3,4,5] }`.
    - If no results, returns an empty list.

**Key Points**

- Lightweight logic: mostly just a DB SELECT sorted by date/hour.
- No caching needed, because the `videos` table typically has fewer rows.
- Helps the UI show only valid calendar/time slots.
### getWeather Lambda from End User Workflow

High-Level Purpose: Delivers two key items for a user’s chosen city/hour:
    1. The **video URL** (if it exists)
    2. **Weather data** (metadata) for that hour (temperature, humidity, etc.)

**General Flow**

1. **Check Redis Cache**:
    - If `(topic + date + hour)` is already cached, returns immediately.
2. **Query the DB**:
    - Finds a matching record in `videos` (for the hour) to get `video_url`.
    - Gathers any relevant rows in `images` for that hour (to compile average or raw metadata).
3. **Response**:
    - JSON includes `{ video_url, imagesMetadata }`.
    - Writes to Redis for future requests.

**Key Points**

- Ensures repeated queries for the same city/hour are fast (thanks to caching).
- If no video is found, `video_url` might be null, but the user still sees whatever weather data is available.
- The **API** is intended for the React frontend’s date/hour selection flow.

# 9. Architecture Decisions

Below are key architecture decisions (ADRs) in short:
### 9.1 AD 1: Use AWS Serverless for Data Flows

- **Context**: Frequent but discrete image uploads, compression tasks, scheduled video generation, scheduled webcam service - ideal context for an event-driven architecture with clearly defined workflows.
- **Decision**: Event-driven Lambdas with S3 triggers and EventBridge schedules, the whole architecture entirely within AWS ecosystem.

### 9.2 AD 2: DB Record is Inserted Before S3 Upload for uploadImage Lambda

- **Context**: In the beginning, uploadImage Lambda first uploaded an image to S3, and only then stores the link to it in the db. Race condition occurred, because compressImage is invoked on S3 upload and requires the `non_compressed_link` images table column to contain a link. This way, if we were to first upload an image before updating the row, compressImage tried to update a DB column (`compressed_link`), but needed the link from `non_compressed_link` column, that doesn’t exist yet.
- **Decision**: Insert the row in RDS first, then upload the file to S3 (triggering compression).

### 9.3 AD 3: Hourly Video Instead of Full-Day

- **Context**: Full-day videos might be large and slow to generate; Hourly videos are a better granularity. `generateVideo` Lambda is triggered every hour for each topic, generating 4 videos per hour, and providing users with readily available videos.
- **Decision**: `generateVideo` Lambda queries for a single hour.
- **Consequences**:
    - Each hour’s video is ~12 images (5-minute intervals) * 12 = 12.
    - Quicker ffmpeg runs, simpler user selection.

By:
- Yehor Zudikhin
- Luca Carpentieri