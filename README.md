# Handwritten Number Generator

The goal of this project is to write a program that can generates images of an input number
for data training and augmentation purposes from MNIST-like Data.

## Installation

Install the requirements in your python3 environment
> pip install -r requirements.txt

> "pip install handwritten-number-generator-1.0.0-py3-none-any.whl"  (present in dist/ folder)

## Solution

Control Flow :

- WebService is started
- Application initializes by downloading MNIST data, if not present already in the
data/ folder
- When the "/generate" endpoint is triggered by a request:
  - it generates a task_id for the particular request and
  - it randomly chooses images of the required digits 
from the dataset.
- If the "augment" flag is set to 1 in the request payload, the images are augmented using 
albumentations (a fast image augmentation library)
- these images are then horizontally stacked together, and the is stretched according 
- finally, post processed image is sent back in the response payload and is saved in 
"data/processed/" folder in PNG format, by the name of the task_id and number requested.

## Usage

### Third party Usage

General use
```
from handwritten_number_generator import generator
import matplotlib.pyplot as plt
%matplotlib inline

im = generator.generate_numbers_sequence(digits=[4,2], spacing_range=(5,10), image_width=56) # generates 42, augments images by default

im = generator.generate_numbers_sequence(digits=[4,2], spacing_range=(5,10), image_width=56, augment_flag="false") # generates 42, does not augment digits

plt.gray()
plt.imshow(im)
plt.show()
```

For fetching MNIST data
```
mnist_data = generator.get_mnist_data()
```

Same generated image can be regenerated by using random.seed() beforehand.

### Web Service

Web Service can be run by starting the "startup.sh" shell script found 
in "digits_sequence_generator" folder. Run this first.

> Run command "sh startup.sh" ( from the directory digit-sequence-generator/digits_sequence_generator/ )

The webservice uses gunicorn for deploying the flask based app. 
Gunicorn is helpful in handling multiple application workers, timeouts 
and improves scalablity of the application.

Response Json
> {
    "augment": "false",
    "generatedImage": [...],
    "number": 1234560,
    "spacing_min": 5,
    "spacing_max": 10,
    "image_width":130,
    "taskId": "122246147423"
}


#### Endpoints

"http://127.0.0.1:5000/health" -> Health Check

"http://127.0.0.1:5000/generate" -> Receiving post requests for generating data.

Example payload for posting :
> {
    "augment": "true",
    "number": 42,
    "spacing_min": 5,
    "spacing_max": 10,
    "image_width": 56
}

Keys:
"augment": Augmentation flag (Values: "true", "false")
"number": Number to be generated
"spacing_min": minimum threshold of spacing_range
"spacing_max": maximum threshold of spacing_range
"image_width": width of the final image


### Scripts

Scripts can be found in "scripts" folder. For interacting with the webservice 

##### digits_generator.py
A script that saves examples of generated images based on user input:

> usage: digits_generator.py [-h] [-n NUMBER] [-a {0,1}] [-smin SPACING_MIN]
                           [-smax SPACING_MAX] [-w IMAGE_WIDTH] 
                           
> - -h, --help            show this help message and exit
> - -n NUMBER, --number NUMBER
                    Set the number
> - -a {0,1}, --augment {0,1}
                    Set the augmentation flag, 1 -> Yes, 0 -> No
> - -smin SPACING_MIN, --spacing_min SPACING_MIN
                    Set minimum value of spacing_range in pixels
> - -smax SPACING_MAX, --spacing_max SPACING_MAX
                    Set maximum value of spacing_range in pixels
> - -w IMAGE_WIDTH, --image_width IMAGE_WIDTH
                    Set width of the final image in pixels

Example :

> "python digits_generator.py -n 42 -a 1 -smin 5 -smax 10 -w 56"

##### Output images
Output images are saved in both
- directory of the "digits_generator.py" script 
- "data/processed/" folder

## Testing
pytest is used as testing framework. The tests can be run from project root by running the command "pytest"

## Augmentations 
Albumentations library was chosen as I've observed it to be faster library for augmentations, and is quite stable as well.

Augmentations are made randomly, based on the set probabilitites. The following operations are supported. 
**Note -  Augmentations are made if *augment* flag is NOT "false"**

- Shifting
- Scaling 
- Rotating
- Blur
- Optical Distortions
- Grid Distortions 
- ElasticTransforms [Simard2003]
- Adding GaussNoise
- Median Blur
- Brightness and Contrast
- Emboss effect

## Production Scenario

The application would be deployed via a docker container 

- Flask: Python based server backend;
- Nginx: Reverse proxy;
- Gunicorn: Deploy flask app;
- Supervisor: Monitor and control gunicorn process

The configuration files for deploying in production are present in "docker/". Warning, these files are **NOT TESTED**

