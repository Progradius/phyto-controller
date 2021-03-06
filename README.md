[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]



<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/progradius/PhytoController">
  <img src="images/logo.png" alt="Logo" width="300">
  </a>
  <h3 align="center">PhytoController</h3>
  <p align="center">
  ESP32 based application, enabling full control of your greenhouse
  <br />
  <a href="https://github.com/Progradius/PhytoController/wiki"><strong>Explore the docs.</strong></a>
  <br />
  <br />
  <a href="https://www.youtube.com/watch?v=rAdMIOjdiVc&feature=youtu.be">View Demo</a>
  .
  <a href="https://github.com/Progradius/PhytoController/wiki/Project-Features">View Features</a>
  .
  <a href="https://github.com/Progradius/PhytoController/labels/bug">PhytoController Bug</a>
  .
  <a href="https://github.com/progradius/PhytoController/issues">Request Feature</a>
  </p>
  </p>



<!-- TABLE OF CONTENTS -->
## Table of Contents

* [About the Project](#about-the-project)
  * [Built With](#built-with)
* [Getting Started](#getting-started)
  * [Prerequisites](#prerequisites)
  * [Installation](#installation)
* [Usage](#usage)
* [Roadmap](#roadmap)
* [Contributing](#contributing)
* [License](#license)
* [Contact](#contact)
* [Acknowledgements](#acknowledgements)


<!-- ABOUT THE PROJECT -->
## About The Project

 [![PhytoController Grafana UI][product-screenshot]](/images/screenshot.png)


### Built With

* [ESP32](https://www.espressif.com/en/products/socs/esp32/overview)
* [MicroPython](https://docs.micropython.org/en/latest/esp32/tutorial/intro.html)
* [InfluxDB](https://www.influxdata.com/)
* [Grafana](https://grafana.com/)



<!-- GETTING STARTED -->
## Getting Started
This project aims to automate and log a handful of things, to visualize the data on Grafana, and to configure the ESP32 with a web GUI

To get a local copy up and running follow these simple steps.
You will need an ESP32 of your choice, the common prototyping stuff a bunch of sensors and a spare machine running your preferred OS

### Prerequisites
**You need to have experience with handling electronics and electricity, some part of the project use high voltage and current and could be dangerous if handled with insufficient knowledge.**

**If you are not confident, skip the relay part of the project or seek help and documentation.**

To harness the full power of this project, gather the following stuff:
* ESP32
* USB cable + 5v outlet
* A  bunch of Dupont wire
* A host machine to run influxdb and grafana
* A 5v relay board (with 8 modules)
* BME280  (temp/hygro/pressure sensor)
* DS18B20 (temp sensor)
* TSL2591 (lux and ir sensor)
* VEML6075 (UV sensor)
* HCSR04 (ultrasonic distance sensor)

### Installation

 [In Depth Tutorial HERE](https://github.com/Progradius/PhytoController/wiki/In-Depth-Tutorial)
 
### 1 - Host instructions
Install the following packages on host machine
(Raspberry Pi based example, see your OS documentation for more accurate information)

#### Grafana

1 - Add the APT key used to authenticate packages:
```sh
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```
2 - Add the Grafana APT repository:
```sh
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
3 - Install Grafana:
```sh
sudo apt-get update && sudo apt-get install -y grafana
```
4 - Enable  and start Grafana service:
```sh
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```
Grafana is now reachable on port 3000 with the login admin:admin

#### InfluxDB
1 - Add the APT key used to authenticate packages:
```sh
wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/os-release
```
2 - Add the Influxdb APT repository:
```sh
echo "deb https://repos.influxdata.com/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
```
3 - Install Influxdb:
```sh
sudo apt update && sudo apt install -y influxdb
```
4 - Enable and start Influxdb service:
```sh
sudo systemctl enable influxdb.service
sudo systemctl start influxdb.service 
```
5 - Use the CLI by entering influx and create a database and a user:
```sh
create database <databasename>
use <databasename>
create user <username> with password '<password>' with all privileges
grant all privileges on home to grafana
```
You can access influxb's database on port 8086
   
#### Add influxdb as a datasource in Grafana:

With the previously created database and credentials, use the official Documentation to add a datasource to Grafana: [Documentation](https://grafana.com/docs/grafana/latest/features/datasources/add-a-data-source/)

#### Install ampy:
ampy is a python tool enabling you to communicate with the microcontroller using the serial port.

Install ampy following the official documentation for your choosen OS: [Documentation](https://github.com/scientifichackers/ampy)

### 2 - ESP32 instructions

 1 - Flash micropython firmware on your ESP32
 
  * Check micropython's documentation and flash a firmware built with ESP-IDF v3.x
  * [ESP32-Documentation](https://docs.micropython.org/en/latest/esp32/tutorial/intro.html)
  * [Micropython for Generic ESP32 Module](https://micropython.org/download/esp32/)

 2 - Clone the PhytoController repository on the host machine
 ```sh
 git clone https://github.com/progradius/PhytoController.git
 ```
 3 - cd into the newly created directory
 ```sh
cd PhytoController/ 
```
4 - Open param.json and configure it:

[Detailed BreakDown of param.json](https://github.com/Progradius/PhytoController/wiki/Configuration-File)

* Enter your wifi credentials
* Enter your influxdb credentials
* Check the defaults GPIO pins and modify them to fit your board if necessary
* Enable the sensors that you possess by setting their fields to "enabled" (in initial configuration, only the DS18B20 is active.)

5 - Wire the components:

**WARNING! Be careful with sensor voltage while wiring it, some of them exist in 5v or 3.3v versions.**

**Order yours wisely (3.3v version should be preferred when available) and wire them properly.**

**Respect the chosen GPIO set in param.json, relays board are handling high voltages so please be extra careful.**

6 - Connect the ESP32 to the host machine

7 - Use ampy to upload the project's files:
```sh
ampy --port /dev/ttyUSB0 put boot.py
ampy --port /dev/ttyUSB0 put main.py
ampy --port /dev/ttyUSB0 put function.py
ampy --port /dev/ttyUSB0 put param.json
ampy --port /dev/ttyUSB0 put controller
ampy --port /dev/ttyUSB0 put model
ampy --port /dev/ttyUSB0 put lib
```

8 - Reboot

 <!-- USAGE EXAMPLES -->
## Usage

This project is able to:

* Monitor various metrics and showcase them into Grafana
* Handle two daily timer outlets
* Handle two cyclic timers outlets
* Handle a centrifugal motor and set his speed according to temperature
* Provides a websocket based web UI

 _For more examples, please refer to the [Documentation](https://github.com/Progradius/PhytoController/wiki)_



 <!-- ROADMAP -->
## Roadmap

 See the [open issues](https://github.com/progradius/PhytoController/issues) for a list of proposed features (and known issues).



 <!-- CONTRIBUTING -->
## Contributing

 Contributions are what make the open source community such an amazing place to be learn, inspire, and create. 
 
 Any contributions you make are **greatly appreciated**.

 1. Fork the Project
 2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
 3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
 4. Push to the Branch (`git push origin feature/AmazingFeature`)
 5. Open a Pull Request



 <!-- LICENSE -->
## License

 Distributed under the GNU AGPLv3 License. See `LICENSE` for more information.



 <!-- CONTACT -->
## Contact

 Progradius - progradius@protonmail.com

 Project Link: [https://github.com/progradius/PhytoController](https://github.com/progradius/phyto-controller)



 <!-- ACKNOWLEDGEMENTS -->
## Acknowledgements

* [BME280 robert-hh's MicroPython Library](https://github.com/robert-hh/BME280)
* [TSL2591 jfischer's MicroPython Library](https://github.com/jfischer/micropython-tsl2591)
* [VEML6075 neliogodoi's Micropython Library](https://github.com/neliogodoi/MicroPython-VEML6075)
* [HCSR04 rcs1975's MicroPython Library](https://github.com/rsc1975/micropython-hcsr04)
* [Best-README-Template](https://github.com/othneildrew/Best-README-Template)





 <!-- MARKDOWN LINKS & IMAGES -->
 <!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
 [contributors-shield]: https://img.shields.io/github/contributors/progradius/phytocontroller.svg?style=flat-square
 [contributors-url]: https://github.com/progradius/phytocontroller/graphs/contributors
 [forks-shield]: https://img.shields.io/github/forks/progradius/phytocontroller.svg?style=flat-square
 [forks-url]: https://github.com/progradius/phytocontroller/network/members
 [stars-shield]: https://img.shields.io/github/stars/progradius/phytocontroller.svg?style=flat-square
 [stars-url]: https://github.com/progradius/phytocontroller/stargazers
 [issues-shield]: https://img.shields.io/github/issues/progradius/phytocontroller.svg?style=flat-square
 [issues-url]: https://github.com/progradius/phytocontroller/issues
 [license-shield]: https://img.shields.io/github/license/progradius/phytocontroller.svg?style=flat-square
 [license-url]: https://github.com/progradius/phytocontroller/blob/master/LICENSE.txt
 [linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=flat-square&logo=linkedin&colorB=555
 [linkedin-url]: https://linkedin.com/in/
 [product-screenshot]: images/screenshot.png
