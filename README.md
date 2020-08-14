### To start
```
python3 app.py 
```

### IO calibration

- Use the RAW.py and paste into google sheets and download as a csv
- Use any text editor to open the csv as csv format as below. (`visual-code` will work)
- Use https://www.convertcsv.com/csv-to-json.htm and option `CSV To Json Column Array`
- Add the json file in `/data/rubix-wires/io-calibration.json`

```csv
UI1, UI2, UI1_MA
0.1, 0.1, 1
0.2, 0.3, 10
0.4, 0.4, null
```


### Installation

- `bash setup.bash`

#### In details

https://learn.adafruit.com/setting-up-io-python-library-on-beaglebone-black/installation-on-ubuntu

- `sudo apt-get update`
- `sudo apt-get install build-essential python-dev python-setuptools python-pip python-smbus python3-pip virtualenv -y`
- `pip install -U pip setuptools wheel`
- `cd bb-py-rest`
- `rm -r venv` // remove virtual env, if exist
- `virtualenv -p python3 venv` // creating virtual env with python3
- `source /home/debian/bbb-py-rest/venv/bin/activate` // activating virtual env
- `pip3 install -r requirements.txt` // installing the packages for this project
- `deactivate` // deactivate current environment

### Systemd Service

- `sudo cp bbio.service /etc/systemd/system/`
- `sudo systemctl daemon-reload`
- `sudo systemctl enable bbio.service`
- `sudo systemctl start bbio.service`

###### Other Systemd Commands

- `sudo systemctl stop bbio.service`
- `sudo systemctl status bbio.service`
- `sudo journalctl -f -u bbio.service`

### For testing

Comment out all the `Adafruit_BBIO` libs 
& uncomment the random values as below ``val = random.uniform(0, 1)  # for testing``

```python
@app.route('/api/' + api_ver + '/read/' + ui + '/<io_num>', methods=['GET'])
def read_ai(io_num=None):
    gpio = analog_in(io_num)
    if gpio == -1:
        return jsonify({'1_state': "unknownType", '2_ioNum': io_num, '3_gpio': gpio, '4_val': 'null',
                        "5_msg": analogInTypes}), http_error
    else:
        # val = ADC.read(gpio)
        val = random.uniform(0, 1)  # for testing
        return jsonify({'1_state': "readOk", '2_ioNum': io_num, '3_gpio': gpio, '4_val': val,
                        '5_msg': 'read value ok'}), http_success
```

##### Block port 5000

In ip tables -A is to add and -D is to delete an entry

```
// only allow localhost access to port 5000
sudo iptables -A INPUT -p tcp -s localhost --dport 5000 -j ACCEPT
// drop all other hosts
sudo  iptables -A INPUT -p tcp --dport 5000 -j DROP
// then save the tables
!!! Not tested
https://upcloud.com/community/tutorials/configure-iptables-centos/

// to remove a rule
sudo iptables -D INPUT -p tcp --dport 5000 -j DROP

```


### API for the GPIO

### LoRa Connect Reset

This api will let you reset the power on the lora connect

```
// write the lora connect off
localhost:5000/api/1.1/write/do/lc/0/16



```
IO_TYPES
ui, uo, di, do

// read 
/read/IO_TYPE/IO_Number
// read all IOs as per type. like /read/all/ui
/read/all/IO_TYPE

```

#### Read a UI as a DI (jumper needs to be set to 10K)
off/open = around 0.9 vdc
on/closed = around 0.1 vdc


### read all
```
// read all DIs
http://0.0.0.0:5000/api/1.1/read/all/di
// read all AIs
http://0.0.0.0:5000/api/1.1/read/all/ai
```

#### UOs
https://learn.adafruit.com/setting-up-io-python-library-on-beaglebone-black/pwm

```
UOs 0 = 12vdc and 100 = 0vdc (Yes its backwards)
<io_num>/<val>/<pri>
uo/uo1/22/16
the priority (pri) is not supported yet but it's there for future use if needed
http://0.0.0.0:5000/api/1.1/write/uo/uo1/100/16
// this returns the values that was stored in the DB (So not reading the actual pin value)
```

#### DOs
https://learn.adafruit.com/setting-up-io-python-library-on-beaglebone-black/gpio

```
/<io_num>/<val>/<pri>
the priority (pri) is not supported yet but it's there for future use if needed
DOs true for high false for low
http://0.0.0.0:5000/api/1.1/write/do/do1/true/16
```

#### UIs
https://learn.adafruit.com/setting-up-io-python-library-on-beaglebone-black/adc

```
UIs will return a float between 0 and 1
http://0.0.0.0:5000/api/1.1/read/ui/ui1
x
```

#### DIs
https://learn.adafruit.com/setting-up-io-python-library-on-beaglebone-black/gpio

```
DIs will return a int either 0 and 1 (0 is on 1 is off)
http://0.0.0.0:5000/api/1.1/read/di/di1
```
## Cloning eMMc image to microSD card as a Flasher.
* Boot off beaglebone on eMMc you wish to clone.
* Insert a blank FAT formatted microSD card at least 4GB.
* Login to your beaglebone and run the scripts
```bash
cd /opt/scripts/tools/eMMC
sudo ./beaglebone-black-make-microSD-flasher-from-eMMC.sh
```
* Once finish, reboot and boot off the beaglebone using the microSD card as an image flasher


# Add bonescript api and enable UART pins as services to start after boot
## Copy service files to /lib/systemd/system/

```
sudo cp nubeio-bonescript-api.service /lib/systemd/system/
sudo cp nubeio-enable-uart-pins.service /lib/systemd/system/
sudo cp nubeio-enable-uart-pins.timer /lib/systemd/system/
```

## Create symlink for both services

```
sudo ln -s /lib/systemd/system/nubeio-bonescript-api.service /etc/systemd/system/multi-user.target.wants/nubeio-bonescript-api.service
sudo ln -s /lib/systemd/system/nubeio-enable-uart-pins.timer /etc/systemd/system/multi-user.target.wants/nubeio-enable-uart-pins.timer
```

## Enable the new systemd services
```
sudo systemctl daemon-reload
sudo systemctl enable nubeio-bonescript-api.service
sudo systemctl enable nubeio-enable-uart-pins.timer
```

## Reboot and test
```
sudo reboot now
sudo systemctl status nubeio-bonescript-api.service
sudo systemctl status nubeio-enable-uart-pins.timer
```

# Disable UART pin service

Version 1.4 of the Nube iO Edge Controller does not need the UART pins enabled and doing so conflicts with the pins for R1. 

## Delete service files at /lib/systemd/system/
```
sudo rm /lib/systemd/system/nubeio-enable-uart-pins.service
sudo rm /lib/systemd/system/nubeio-enable-uart-pins.timer
```

## Disable the systemd services
```
sudo systemctl daemon-reload
sudo systemctl disable nubeio-enable-uart-pins.timer
```

## Reboot and test
```
sudo reboot now
sudo systemctl status nubeio-enable-uart-pins.timer
```
