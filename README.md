# theta-wifi-api
 Python implementation of a client to communicate with Ricoh Theta V using the camera as AP

## Status
Starting implementation of alternative to USB communication, pipeline between python client and Ricoh Theta V 360 camera device.

## Context

Connecting camera device with device running python script on inspection robot platform.Implementing communication stream between python client and Ricoh Theta V camera device.
## Decision

Since USB is not convinient for this project because of the used enclosure of the camera device, it has been decided to investigate the WiFi API provided by the developers of the device. There are two ways the  device could be used:

- Start video recording when workflow starts and stop recording when done, then transfer the video from the device. This is a nice solution since it does not require alot of bandwith, since the only required  communication is to send commands to start and stop recording.
- Streaming live preview from the camera device and record the preview on the disk of the device receiving the stream. This is a nice solution since the amount of video material that can be recorded is limited to the computer device storage. Also that way the 360 frames can be processed online, on the robotic platform if required.

## Consequences
In order to use the device as explained above, wi fi communication is required between the client device receiving the image stream or controlling the camera device. The 360 camera device can be used as access point, which means the computer device requires wi fi adapter. Using the device wirelessly also means it cannot be charged while using. The device would be powered by its internal battery which is specified to be able to record video for about 80 minutes. In the case of recording video on the camera device, and not streaming it to the computer device, the video would be written to the internal storage of the camera device which is about 19GB in total, according to the specifications the device can hold up to 40 minutes of recording video at 4K resolution.

# Plan of approach and Ricoh Theta Web API #

First, introduction to the Theta API is required. The used device Ricoh Theta V uses the RICOH THETA API v2.1 and conforms to the Open Spherical Camera(OSC) API version 2 by Google. In addition to OSC the Theta API is extended to make use of the full capabilties of the camera device. <br>
In order to use the theta API, Wireless LAN is used as communication protocol. In this case the theta device acts as HTTP server and the API can be used by sending REST commands. When connected to the device, the server can be accessed at the following address : 192.168.1.1:80. <br>
After connecting the computer device to the wifi access point created by the camera, if a GET request is done at http://192.168.1.1/osc/info the following address - or simply navigate to it on browser from the device - information regarding the device will be returned as response. <br>
For the purpose of the project what will be done in order to investigate if and how this can be used implementation of the few most important commands will be implemented, that requires the following functionality:

The full API can be found at this link https://api.ricoh/docs/theta-web-api-v2.1/

- Obtain information and status of the device - That is done by GET request at endpoint /osc/info of the camera device; If empty POST request is done at /osc/state endpoint of the web api, that would return state of the camera - information like the battery level of the device, latest file, remaining shooting video time and many others
- Command to get and change settings of device - capture mode, camera sensor settings
- Command to take picture - Create a still image 
- Command to start continuous shooting - depending on the camera mode (video or image shooting) the device would enter in continuous shooting mode.
- Command to retrieve files from the device - the latest recording once it is done
- Command for live preview from the camera device

# Useful resources:
https://community.theta360.guide/t/theta-v-live-streaming-using-pi-3/2482/7 - one of the comments contains python examples of client wrapper for the camera API, which is threaded and used in sewer inspection robot platform project.
https://api.ricoh/docs/theta-web-api-v2.1/ - api documentation

# Update
The first commit  to the repo includes 2 scripts. The main script has functions, which access ricoh theta api, once the device executing the script is connected to the access point created by the camera device. There are funcitons for:
- Device information - general information about the camera device (firmware version, api version etc.)
- Get device status - returns the state (battery info and state, remaining recording or photo space)
- start/stop capture methods - Depending on the mode of the device, the camera starts shooting in continous mode until stop capture is called, or device is used or acces in any way. If in video mode records a video, if in image mode - sequence of images with delay between.
- take picture - takes a single picture and returns in the response the download/view URL to the file
- list files - function to list the available files on the camera device
- get file - this function can be used to download any file from the device and write it to disk ( pass the full URL to the file on the camera device; pass the path+filename.extension to write the file)
- get live preview - device starts streaming frames and frames are shown to user using opencv; frames can be written to disk and used with 2nd file - movie.py to create a video from frames - for the purpose uncomment line 100
- get/set options - methods to configure or retrieve camera options

In general the code currently is just to test the functionality of the device and API. The code needs further restructoring and providing more options to the user since now most of the things are hardcoded.

# Update 30-12-2020
Starting building a proper client class wrapper to expose the required methods in a decent way.
Client class needs:
- base url - the ip when connected to ricoh device is the same, the authentication credentials are different - done
- get request method - done
- post request method - done
- Methods from the API:
  - get device info - done
  - get device state - done
  - start/stop capture - done
  - list files - 
  - get file - done
  - Extra could be: 
    - live_preview - 
  - Ricoh Theta V driver class is in the RicohTheta.py file. It can be imported,instantiated and used as a class object

# Observations
- As expected there is lag in the live preview mode, but given the speed at which the inspection platform moves, my initial thoughts are that the latency between the camera and the nano won't be a problem
- When used in live preview mode, the output resolution is limited and FPS is 8. This is not nice, since in that case the device is not used up to its full capacity - 4K video recording.
- Device has GPS and it was able to output correct coordinates for the taken image. This can be interesting to check how it would perform in the sewer
- the set_device_imageMode breaks if fileFormat is specified, invalid param values for some reason... File format param breaks for video too, apparently it is not passed properly


# To Do
- Test on linux and Nano - (done) tested on Nano -> cloned repo -> connected to camera using wifi adapter -> run main with preview.
- Cleanup code (done), can be cleaned up further later
- Add methods for setting the camera specific params (ISO, HDR etc)

