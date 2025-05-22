# Telemetry Scanner

The Telemetry Scanner (TS) is a program designed to gather, display, and output live drone flight data. It contains two major components:

1. **cloud_api_mqtt.py** - establishes a Message Queuing Telemetry Transport (MQTT) client, connects the client to a specified IP address, and scans for updated telemetry data from the drone's flight controller

2. **cloud_api_http.py** - serves as a simple login page for drone pilots, allowing easy integration with the drone control interface

## Table of Contents
- [Installation Instructions](#installation-instructions)
- [Testing Without a Drone](#testing-without-a-drone)
- [Connecting To a Real Drone](#connecting-to-a-real-drone)
- [Relevant Data Points](#relevant-data-points)
- [Issues](#issues)
- [Author(s) and Contact](#authors-and-contact)

## Installation Instructions

1. Clone the repo
    - `git clone https://github.com/tboenish/DJI_Cloud_API_test.git` 
    - (TO DO: update link from Thomas's Github to new DIP repo)

2. Install dependencies
    - `pip install -r requirements.txt`
    - **NOTE:** the current requirements file assumes you already have Uvicorn installed. If you do not, you will need to install it. If you receive an error while installing dependencies, you can install them manually with `pip install FastAPI paho-mqtt uvicorn`
    - (TO DO: update requirements.txt to include Uvicorn)

3. Download and install Docker Desktop and make your account
    - In your browser, navigate to [Docker](https://www.docker.com/get-started/)
    - Download the appropriate version of Docker Desktop
    - Run the Docker Desktop Installer
    - **NOTE:** During installation, you may be prompted to choose between Windows and Linux containers. The Telemetry Scanner uses **Linux Containers**, so if you are prompted, do not choose Windows. If you are not prompted, proceed to the next step
    - When the installation is finished, you will be prompted to restart your system. Do so
    - When your system comes back up, Docker should run automatically, with a terms of service page. Click 'Accept'
    - You will then be taken to the account setup page. Ensure you select 'personal'
    - Create your Docker account
    - **NOTE:** You must verify your email address using the email Docker sends or it will not run properly 

4. Spin up the EMQX server
    - Open a terminal and run the command `docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083  emqx:5.0.20`
    - Verify the container by running `docker ps` in your terminal. You should get an output with headings like 'CONTAINER ID', 'IMAGE', and so on. You should also now see your EMQX server in the Docker Desktop app
    - Open [localhost](http://localhost:18083/)
    - You will arrive on a login page
        - Username: admin
        - Password: public
    - You will then be prompted to update your password
    - You should now arrive at the EMQX Dashboard
    
5. Connect DJI Smart Controller to the same local network as your PC
    - **NOTE:** This step is only necessary when *actually* running the program with a drone. If this is your initial setup, or you are running simulations, proceed to step 6

6. Update files
    - Update the "host_addr" variable in both cloud_api files to your own IP address
    - **NOTE:** This can be found by inputting `ipconfig` in your terminal and navigating to the first IPv4 address
    - 'username' and 'password' can remain set to 'admin' and 'public' respectively

7. Running the HTTP file
    - In your terminal, input `python cloud_api_http.py`
    - Your terminal should populate with several 'INFO' statements, one of which will say something like "Uvicorn running on http://your-ip-address:5000"
    - Opening the link should open your browser to a blank page
          - **NOTE:** When you open the link, you may see some 404 errors in your terminal. The root page and favicon are not currently defined. This does not indicate a bug and you may safely proceed to the next step
    - In your browser's address bar, add "/login" at the end of the address and hit enter
    - A basic HTML page titled "DJI Cloud API test" should load
    - This is the page that the drone pilot will see and be able to log into to share data with us

8. Running the MQTT file
    - Open a second terminal, input `python ./cloud_api_mqtt.py`
    - The terminal should populate and say "Connected with result code Success"
    - Reopen [localhost](http://localhost:18083/)
    - You should now be able to receive data from the drone

## Testing Without a Drone

This section will guide you through the process of inputting dummy data. This is useful for  
- Verifying information is making it from your Docker container to the MQTT server  
- Running tests when we do not have an actual drone available

To test without a drone:
1. Ensure your HTTP and MQTT files are running
2. Navigate to the EMQX dashboard (opened in Step 8 Part c)
3. In the left sidebar, expand the 'diagnose' toggle menu (it looks like a magnifying glass) and click the 'WebSocket Client' button
4. In the 'Connection' section, click the 'Connect' button
5. In the 'Subscription' section, in the 'Topic' field, input "thing/#"
6. Click the 'Subscribe' button
7. In the 'Publish' section
    - Update the 'Topic' field to read "thing/product/testdrone/osd"
    - Update the 'Payload' field  
    (copy everything inside the block below):
    ```json
    {
    "data": {
        "latitude": 47.6062,
        "longitude": -122.3321,
        "attitude_head": 90,
        "attitude_pitch": 1.5,
        "attitude_roll": -2.3,
        "height": 15.0
    }
    }
    ```
    - Click 'Publish'
8. Return to the terminal associated with your MQTT file. You should have received the telemetry data
   - **NOTE:** The code currently outputs with emojis, which may render strangely in your terminal. This is normal and not indicative of any bugs

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

| Data Point Name | Description | Type |
| --- | --- | --- |
| mode_code | Aircraft state | enum_int: {"0":"Standby","1":"Takeoff preparation","2":"Takeoff preparation completed","3":"Manual flight","4":"Automatic takeoff","5":"Wayline flight","6":"Panoramic photography","7":"Intelligent tracking","8":"ADS-B avoidance","9":"Auto returning to home","10":"Automatic landing","11":"Forced landing","12":"Three-blade landing","13":"Upgrading","14":"Not connected","15":"APAS","16":"Virtual stick state","17":"Live flight Controls","18":"Airborne RTK fixing mode"} |
| mode_code_reason | The reason the aircraft entered the current state | enum_int: {"0":"No meaning","1":"Insufficient battery power (return, landing)","2":"Insufficient battery voltage (return, landing)","3":"Severely low voltage (return, landing)","4":"Requested by remote controller buttons (takeoff, return, landing)","5":"Requested by App (takeoff, return, landing)","6":"Loss of remote controller signal (return, landing, hover)","7":"Triggered by external devices such as navigation, SDK, etc. (takeoff, return, landing)","8":"Entered the dock GEO Zone (landing)","9":"Although a return was triggered, it was too close to the Home point (landing)","10":"Although a return was triggered, it was too far from the Home point (landing)","11":"Requested when executing waypoint missions (takeoff)","12":"Requested after reaching above the Home point in the return phase (landing)","13":"Continued descent after the aircraft's height dropped to 0.7m from the ground (second-stage descent limit) leading to (landing)","14":"Forced breakthrough of low altitude protection by devices like App, SDK (landing)","15":"Requested due to passing flights in the vicinity (returning, landing)","16":"Requested due to height control failure (return, landing)","17":"Entered after intelligent low battery return (landing)","18":"AP controls the flight mode (manual flight)","19":"Hardware abnormally (return, landing)","20":"End of anti-collision protection (landing)","21":"Return canceled (hover)","22":"Encountered obstacles during the return (landing)"} |
| cameras | Aircraft camera information | JSON object that contains remain_photo_num, remain_record_duration, record_time, camera_mode, photo_state, and recording_state |
| remain_photo_num | Remaining number of photos to take | int |
| remain_record_duration | Remaining recording time | int |
| record_time | Video recording duration | int |
| camera_mode | Camera mode | enum_int: {"0":"Capturing","1":"Recording","2":"Smart Low-Light","3":"Panoramic photography"} |
| photo_state | Photo capturing status | enum_int: {"0":"Idle","1":"Capturing photo"} |
| recording_state | Recording state | enum_int: {"0":"Idle","1":"Recording"} |
| is_near_area_limit | Whether approaching the GEO Zone | enum_int: {"0":"Not reaching the GEO Zone","1":"Approaching the GEO Zone"} |
| is_near_height_limit | Whether approaching the set height limit | enum_int: {"0":"Not reaching the set height limit","1":"Approaching the set height limit"} |
| height_limit | Aircraft height limit | int |
| storage | Storage capacity | JSON object that contains total and used |
| total | Total storage capacity | int: {"unit_name":"Kilobytes / KB"} |
| used | Total used storage capacity | int: {"unit_name":"Kilobytes / KB"} |
| battery | Aircraft battery information | JSON object that contains capacity_percent, remain_flight_time, and return_home_power |
| capacity_percent | Total remaining battery capacity | int: {"max":100,"min":0} |
| remain_flight_time | Remaining flight time | int: {"unit_name":"Seconds / s"} |
| return_home_power | Percentage of power required for return home | int: {"max":100,"min":0} |
| total_flight_distance | Accumulated total mileage of the aircraft | float: {"unit_name":"Meters / m"} |
| total_flight_time | Accumulated total flight time of the aircraft | int: {"unit_name":"Seconds / s"} |
| home_distance | Distance from the Home point | float |
| home_latitude | Home point latitude | float |
| home_longitude | Home point longitude | float |
| attitude_head | Yaw axis angle | int |
| elevation | Relative takeoff point altitude | float |
| height | Absolute height | float |
| latitude | Current latitude | float: {"max":"3.4028235E38","min":"-1.4E-45","step":"0.1"} |
| longitude | Current longitude | float: {"max":"3.4028235E38","min":"-1.4E-45","step":"0.1"} |

## Issues
Errors and bugs should be added in the repositories 'Git Issues' page  
New to Git Issues? [Here's a great guide!](https://github.com/codeforamerica/howto/blob/master/Good-GitHub-Issues.md)  
TO DO: Include link to repo's 'Git Issues' page 

## Author(s) and Contact
- The Telemetry Scanner was built to be used in coordination with the [Community Development Group's Drone Integration Project](https://github.com/Interject-CommDevTeam/DroneIntegration)
- The original author of this code and readme is [Thomas Boenish](https://github.com/tboenish/)
- Subsequent revision of the readme written by [Nate Shaw](https://github.com/NateShaw2), [Aidan Bucerzan](https://github.com/aidanb58), and [Mavrik McMeekan](https://github.com/MavrikM)
