# SneakyTrafficCamera
This repo holds a traffic camera implementation that uses a single web cam to estimate the speed of a car passing by on a street in the camera's field of view. The speed limit of the street is configurable and a local database is kept of the license plate numbers of the cars that exceed the speed limit.


# Deps
OpenCV (Separate installation, TODO: Only for HW IO, remove dep and make own mJpeg framework)
PyTorch (separate installation)
YOLOv5 (submodule)
EasyOCR (submodule)


# System design
1. Camera stream is received from camera. 30Hz 720p MJPG is currently supported.
2. Image is sent to YOLOv5 model for inference. If "car" is returned, the image is temporarily stored in a "passing vehicle" object that also contains the timestamp when the car enters the field of view and the timestamp when it exits the field of view.
3. Once no more "car" is infered, i.e. has left the field of view, send the "Passing Vehicle" object for speed estimation. The length of road in the camera FoV is divided by the duration between the enter and exit timestamps to get the average speed (see #Calibration).
4. If the average speed exceeds the road's speed limit, store one of the images of the car to the local disk, the name will be YYMMDD-HHMMSS-SPEED.


# Caveats
Multiple cars are currently not supported. If more than one car enter the FoV at any given time, all currently living "Passing Vehicles" will be discarded. Hence this software is currently not suited for highly trafficed roads. (ToDo update design to accomodate for multiple vehicles).


# Calibration
- **The length of road visible to the web camera**. More precisily, this has to be the distance in which a car is labeled as such by the model. Best practive that the author has found is to get childrens' street crayons and make markings in the road where you think a car will be sufficintly visible that it will be labelled correctly, and iterate these markings until you are within about 5 % and then measure the distance between the most concurrent markings. If you do not have access to an elevated position, you can try to use wooden posts to the achieve the same result. (ToDo: on roads with two or more lanes, distance will be much greater in the outer lanes.)
- **The relative time from the left hand side of the camera to when the license plate is easily visible**. For the post processing tools to work, the license plate must be visible. Since we ideally only want to save a single image per traffic speed violator, we need to know where in the image sequence to pick that image. I.e. if a car is visible for 20 seconds, and cars that enter on the left hand side have their license plate visible at 16 seconds in, then the value you should input is 16 / 20 = 0.8.


# Post processing
- **Get license plate numbers and speed**. For each image in the database, uses EasyOCR to get the license plate of the vehicle. You then have the option to sort by most numbers of speed limit violations per vehicle or all time speed record.
- **Generate angry mail to speed limit offenders**. If you live in a juristiction where vehicle ownership is public data, it can be a fun trick to generate angry mails to your top scoring offenders. You will have to look up the name and address of the offenders yourself in local databases (though any scraping tool PR will be accepted). You then simply have to input the name for the offender in question (per license number) and an angry pdf will be generated addressed to the offender and which lists the vehicle(s) used and the total set of speed limit violations. You may then print said pdfs, put them in envelopes and drop them off at the nearest post-box. Maybe this will make your community a bit safer in a traffic point of view?
