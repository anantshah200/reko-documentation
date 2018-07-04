# Kpoint-Rekognition API Documentation

Documentation for REST-ful API's implemented using AWS API's.

## Title 

Show All Videos Where the UserId was found\
Given a User-Id, this API will list all the videos stored in a Amazon S3 bucket in which the user was found, along with the timestamps at which the user appeared in the video. An image with the user's face only needs to be stored in another S3 bucket so that it can be added to an AWS collection from where it is compared with the faces in the videos. 

### URL 

/app/video/search?userId=${userId}

### METHOD

GET

### URL Params

Required:

* userId=[String]\
example: userId=anant-shah-kpoint

### Success Response

The response of the API is a mapping, in which the key is the ID of the video and the value is a timestamp list of where the user was found in that specific video.

Example:

Code:200\
Content:{"gcc-abcde123-fg45-hi67-jklmno890123":{NULL},"gcc-abefq412-ef34-mn97-abcdeg786512":{0,1,2,8,9,10,11}}

### Error Response

Example:

Code:422 Unprocessable Entity\
Content:{"User does not exist;please add an image":{NULL}}

### Notes 

The application has been made such that the user just needs to enter the User-ID of the person he wants to find in the videos to the API, and the application will deliver a list of videos along with the timestamps at which the user was found.

## Title
Coordinates of Persons in a Video \
Given a video-ID, this API will return a map in which the key is the timestamp and the value is a list consisting of the coordinates along with the ID of the person.

### URL

/app/video/coordinates?gconfid=${gconfid}

### METHOD

GET

### URL Params

Required :

* gconfid=[String]\
example: gconfid=gcc-abcde123-fg45-hi67-jklmno890123

### Success Response

The response of the API is a map whose key is the timestamp and value is a list of persons found at that timestamp in the video along with their coordinates.

Example :

Code:200\
Content:{"1.0":{("Left"=0.45,"Top"=0.55,"Width"=0.34,"Height"=0.43,"userId"="anant-shah-kpoint"),("Left"=0.12,"Top"=0.78,"Width"=0.56,"Height"=0.67,"userId"=NULL)},"2.0":{("Left"=0.45,"Top"=0.55,"Width"=0.34,"Height"=0.43,"userId"="anant-shah-kpoint"),("Left"=0.12,"Top"=0.78,"Width"=0.56,"Height"=0.67,"userId"=NULL)}}

### Error Response

Example:

Code:422 Unprocessable Entity\
Cause : Video with the entered ID does not exist

### Notes

Once the user selects a video, the ID of the video is mapped and the application run with that ID. At each integer second, it will produce the location of each person at that time instant in the video.

## Title

Add Face to a Collection\
Given an image stored in a Amazon S3 bucket, the face metadata obtained from that image is stored in an Amazon Collection. A face ID is given to the face meta-data in the collection. The face ID needs to be provided as a parameter to the API.

### URL 

/app/image/add?image=${Image_Path_S3_Bucket}&userId=${userId}

### METHOD

PUT

### URL Params

Required:

* image=[String]\
example: image=Test_Face.jpg
* userID=[String]\
 example: userId=test-face-kpoint

### Success Response

* If the image is successfully added to the collection, then the faceID of the face in the collection is returned.

Example:

Code:200\
Content:{"FaceId : ABCD1234efgh789"}

* If the image has successfully been identified in the S3 bucket, but could not be added to the collection due to the fact that the same face already exists in the collection, a string is returned stating the fact that the face already exists in the collection.

Example :

Code:200\
Content:{"Not adding this image to the collection(Already exists). User Id : " + ${IDOfFaceFound}}

* If the image has successfully been identified in the S3 bucket, but could not be added to the collection because of the fact that a face with the same User ID exists, a string stating the above reason is returned.

Example :

Code:200\
Content:{"User with this ID exists, cannot add this image"}

* If the image has successfully been identified in the S3 bucket, but could not be added to the collection as the image contains multiple faces.

Example :

Code:200\
Content:{"Could not add face to the collection as multipled faces were found in the image."}
### Error Response

Example:

* Code:422 Unprocessable Entity\
Content:{"Please enter another image"}
Cause : An error occured while deleting the multiple faces found in the image.

* Code:422 Unprocessable Entity\
Content:{"No faces found in the image. Please add another image"}
Cause: The image could not be read from the S3 bucket or the image can be read but the Amazon Rekognition service cannot read any face in that image.

### Notes

* The whole application is built on the fact that unique face meta-data are stored in the collection. To ensure this, if the input face matches a face that already exists, the application will reject it. The face matching threshold is kept at 60%.
* Each face meta-data in the collection is assigned a unique User Id. Now what AWS API's allow is that we can add a "externalImageId" to an image. What we are doing is, we are assigning the ID entered by the user (argument to the API) to the image(also an argument to the API). Since we are making sure that the image only contains one face, the ID is assigned to the face. 
* Since we need to make sure that the input image consists just one face, images which contain multiple faces are rejected.
* If the image cannot be read, an error message is returned. 

## Title

Update Face in Collection\
Suppose the user wants to update an existing face in the collection, he can do so with the help of this API.

### URL

/app/video/update?image=${image_S3_Bucket}&userId=${userId}

### METHOD

PUT

### URL Params

Required:

* image=[String]\
example: image=Test_Face.jpg
* userId=[String]\
 example: userId=test-face-kpoint

### Success Response 

* If the image has successfully been updated in the collection, a string is returned by the API.

Example : 

Code:200\
Content:{"Face Updating .... FaceId : ABCD1234abce"}

* If the face could not be updated in the collection because of the fact that the new image consisted of multiple faces.

Example : 

Code:200\
Content:{"Face Updating ....Could not add face to the collection as multipled faces were found in the image."}

* If the face could not be updated in the collection because of the fact that a user with the entered UserId does not exist in the collection.

Example :

Code:200\
Content:{"Could not update this face as it was not found in the collection"}

### Error Response

* If the face could not be updated in the collection becuase the image that the user entered was not valid.

Example :

Code:422\
Content:{"Please enter another image"}

### Notes

* If the user wants to update a photo in the collection, he can do so by calling the above described API.
* What the API first does is that it deleted the existing face meta-data(which is identified by the UserId passed as a parameter to the API) and then adding the new face meta-data indexed with the same UserId.
* Hence, if the image that the user enters is not valid(does not contain a face, contains multiple faces, etc.), then the old user face data will be deleted and hence cannot be retrieved.

## SPECIFICATIONS

AWS Services : 
* AWS SQS Queue : 
1. Label : anant_us-east-1_rekognition_queue
2. URL : https://sqs.us-east-1.amazonaws.com/701980022429/anant_us-east-1_rekognition_queue 
3. Region : US-EAST-1

* AWS SNS Topic : 
1. Label : AmazonRekognition-anant_us-east-1_topic
2. Region : US-EAST-1

* AWS S3 Bucket to store face images : 
1. Label : rekognition-user-faces
2. Region : US-EAST-1

* AWS S3 Bucket where the videos are stored : 
1. Label : cobra-inr.us-east-1.kpoint
2. Region : US-EAST-1

* AWS Collection : 
1. Label : kpoint-rekognition-photos


## DOCKER 

1. The docker image is built on the openjdk image.
2. Port 8080 is exposed to the container and hence that tag will be required while executing the container.
3. The aws credentials file has been added to the image as the credentials file at the location /.aws/credentials is searched in the environment of where the application is run.
4. The image solely contains the jar file and java which are the two things required to run the project which is built on the Spring framework. 
5. As stated earlier, different images could be created with different credentials file, so as to solve the problem of excess simultaneous load on the AWS Rekognition service.

## ENVIRONMENT 

1. The application is built using the project management tool Maven. 
2. The application has been developed with the help of the Spring framework. 
3. The server is implemented using the Tomcat servlet. The default port that Tomcat uses is 8080. 

## SECURITY

1. The images that are enetered into the AWS S3 bucket need some sort of verification. For example, if they enter an image of Linus Torvalds but enter the User ID as Roger Federer, then it will not be right.
2. Some specific convention has to be given to the user Id's such as their kpoint login ID or something like <name>-kpoint. For example : anant-shah-kpoint. This will ensure that the user Id's are unique.
3. When they enter the image, it is mandatory that they should have a kpoint ID(need to add this feature before they can run the API).
4. Problem :
*  At a time, for one IAM User(one set of credentials) in AWS, only 20 operations can simultaneously be performed by the Rekognition Service. And hence if there are a large number of requests(greater than 20) an error might be thrown.
Proposed Solution :
* Map requests to multiple user credentials and hence as a result a lot of load will not come up on one rekognition service.
* Have some sort of a queueing system, where operations which take time(/app/video/search) can be performed later while urgent operations like "/app/image.." and "/app/video/coordinates" can be performed first.
5. The image parameter is the path of the image in the S3 bucket.
6. The update API will delete the old image and add a new one, so only special users(the users themselves) should be given the right to update an image. Otherwise anyone will be able to delete face metadata from a collection. 
