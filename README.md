# Kpoint-Rekognition API Documentation

Documentation for REST-ful API's implemented using AWS API's.

## Title 

Show All Videos
Given a User-Id, this API will list all the videos stored in a Amazon S3 bucket in which the user was found, along with the timestamps at which the user appeared in the video. An image with the user's face only needs to be stored in another S3 bucket so that it can be added to an AWS collection from where it is compared with the faces in the videos. 

### URL 

/app/video/search?userId=${userId}

### METHOD

GET

### URL Params

Required:

* userId=[String]
* example: userId=anant-shah-kpoint

### Success Response

The response of the API is a mapping, in which the key is the ID of the video and the value is a timestamp list of where the user was found in that specific video.

Example:

Code:200
Content:{"gcc-abcde123-fg45-hi67-jklmno890123":{NULL},"gcc-abefq412-ef34-mn97-abcdeg786512":{0,1,2,8,9,10,11}}

### Error Response

Example:

Code:422 Unprocessable Entity
Content:{"User does not exist;please add an image":{NULL}}

### Notes 

The application has been made such that the user just needs to enter the User-ID of the person he wants to find in the videos to the API, and the application will deliver a list of videos along with the timestamps at which the user was found.

## Title
Coordinates of Persons
Given a video-ID, this API will return a map in which the key is the timestamp and the value is a list consisting of the coordinates along with the ID of the person.

### URL

/app/video/coordinates?gconfid=${gconfid}

### METHOD

GET

### URL Params

Required :

* gconfid=[String]
* example: gconfid=gcc-abcde123-fg45-hi67-jklmno890123

### Success Response

The response of the API is a map whose key is the timestamp and value is a list of persons found at that timestamp in the video along with their coordinates.

Example :

Code:200
Content:{"1.0":{("Left"=0.45,"Top"=0.55,"Width"=0.34,"Height"=0.43,"userId"="anant-shah-kpoint"),("Left"=0.12,"Top"=0.78,"Width"=0.56,"Height"=0.67,"userId"=NULL)},"2.0":{("Left"=0.45,"Top"=0.55,"Width"=0.34,"Height"=0.43,"userId"="anant-shah-kpoint"),("Left"=0.12,"Top"=0.78,"Width"=0.56,"Height"=0.67,"userId"=NULL)}}

### Error Response

Example:

Code:422 Unprocessable Entity
Cause : Video with the entered ID does not exist

### Notes

Once the user selects a video, the ID of the video is mapped and the application run with that ID. At each integer second, it will produce the location of each person at that time instant in the video. 
