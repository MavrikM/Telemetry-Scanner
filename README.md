# Telemetry Scanner

The Telemetry Scanner (TS) is a program designed to gather, display, and output live drone flight data. It contains two major components:

1. **cloud_api_mqtt.py** - establishes a Message Queuing Telemetry Transport (MQTT) client, connects the client to a specified IP address, and scans for updated telemetry data from the drone's flight controller.

2. **cloud_api_http.py** - serves as a simple login page for drone pilots, allowing easy integration with the drone control interface

## Installation Instructions

1. Clone the repo
    - `git clone https://github.com/tboenish/DJI_Cloud_API_test.git` 
    - (TO DO: update link from Thomas's Github to new DIP repo)

2. Install dependencies
    - `pip install -r requirements.txt`
    - **NOTE:** the current requirements file assumes you already have Uvicorn installed. If you do not, you will need to install it. If you receive an error while installing dependencies, you can install them manually with `pip install FastAPI paho-mqtt >= 2 uvicorn`
    - (TO DO: update requirements.txt to include Uvicorn)

3. Download and install Docker Desktop and make your account
    - In your browser, navigate to [Docker](https://www.docker.com/get-started/)
    - Download the appropriate version of Docker Desktop
    - Run the Docker Desktop Installer
    - When the installation is finished, you will be prompted to restart your system. Do so.
    - Run Docker Desktop

4. Spin up the EMQX server
    - `docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083  emqx:5.0.20`
    - Verify the container by running `docker ps` in your terminal
    - Open [localhost](http://localhost:18083/)
    - You will arrive on a login page.
        - Username: admin
        - Password: public
    
5. Connect DJI Smart Controller to the same local network as your PC
    - **NOTE:** This step is only necessary when *actually* running the program with a drone. If this is your initial setup, proceed to step 6.

6. Update files
    - Update the "host_addr" variable in both cloud_api files to your own IP address
    - **NOTE:** This can be found by inputting `ipconfig` in your terminal and navigating to the first IPv4 address
    - 'username' and 'password' can remain set to 'admin' and 'public' respectively

7. Running the HTTP file
    - In your terminal, input `python cloud_api_http.py`
    - Your terminal should populate with several 'INFO' statements, one of which will say something like "Uvicorn running on https://xxx.xx."
    - Opening the link should open your browser to a blank page
    - In your browser's address bar, add "/login" at the end of the address and hit enter
    - A basic HTML page titled "DJI Cloud API test" should load
    - This is the page that the drone pilot will see and be able to log into to share data with us

8. Running the MQTT file
    - Open a second terminal, input `python ./cloud_api_mqtt.py`
    - The terminal should populate and say "Connected with result code Success"
    - Reopen [localhost](http://localhost:18083/)
    - You should now see an emqx dashboard login page
    - Login
        - Username: admin
        - Password: public
    - You should now be able to receive data from the drone

## Testing Without a Drone

This section will guide you through the process of inputting dummy data. This is useful for
    - Verifying information is making it from your Docker container to the MQTT server
    - Running tests when we do not have an actual drone available

1. Ensure your HTTP and MQTT files are running
2. Navigate to the MQTT dashboard (opened in Step 8 Part c)
3. In the left sidebar, expand the 'diagnose' toggle menu and click the 'WebSocket Client' button
4. In the 'Connection' section, click the 'Connect' button
5. In the 'Subscription' section, in the 'Topic' field, input "thing/#"
6. Click the 'Subscribe' button
7. In the 'Publish' section
    - Update the 'Topic' field to read "thing/1"
    - Update the 'Payload' field to read "{"msg": "hello world"}" (that is, the first character in the field should be { and the final character should be })
    - Click 'Publish'
8. Return to the terminal associated with your MQTT file. You should have received 'hello world'

## Connecting To a Real Drone

1. Ensure your HTTP and MQTT files are running
2. Ensure the controller is connected to the same Wi-Fi network as your computer
    - The IP your computer is using must be available to the controller
3. On the Controller, Open DJI Pilot App
4. Go to `Cloud Service` -> `Other platforms`
5. Write url `http://<YOUR-IP-ADDRESS-HERE>:5000/login` and connect
6. Press Login
7. If everything is configured correctly, your terminal running the MQTT file should begin displaying live data

## Relevant Data Points
TO DO: Document all data points needed/possibly helpful for the project

## Issues
Errors and bugs should be added in the repositories 'Git Issues' page.  
Not sure how to write a Git issue? [Here](https://github.com/codeforamerica/howto/blob/master/Good-GitHub-Issues.md)'s a great resource!  
TO DO: Include link to repo's 'Git Issues' page 

## Author(s) and Contact
- The original author of this code and readme is Thomas Boenish (https://github.com/tboenish/)
- Subsequent revision(s) of the readme written by Mavrik McMeekan (https://github.com/MavrikM)
