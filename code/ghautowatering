import RPi.GPIO as GPIO, time, os, glob, datetime
import Adafruit_DHT
from gpiozero import *

GPIO.setmode(GPIO.BCM)

# Set all your pins connected to each sensor.
powerled = LED(17)    #Program running status LED
bedone = 20			  #Soil moisture sensor
bedtwo = 21 		  #Soil moisture sensor
valveone = LED(23)	  #Pump valve number 1
valveoneled = LED(27) #Pump 1 indicator led
valvetwo = LED(24)	  #Pump valve number 2 
valvetwoled = LED(22) #Pump 2 indicator led
LDRpin = 19			  #Set the pin for the LDR, Light Dependent Resistor
Humiditysensor = 26	  #Set temp sensor pin
clock = 1800		  #Set intervals for readings in seconds. example 1 min is 60 seconds, 30mins is 1800 
pumptime = 30		  #Set how long you wan't your pumps to run for in seconds
pin = 16			  #Shut down switch

# Sensor should be set to Adafruit_DHT.DHT11,
# Adafruit_DHT.DHT22, or Adafruit_DHT.AM2302.
sensor = Adafruit_DHT.DHT11
humidity, temperature = Adafruit_DHT.read_retry(sensor, Humiditysensor)

powerled.on()			#Power led indiccator

# ---------------- Set up Shutdown Button -----------#
	
# Setup the shutdown Pin with Internal pullups enabled.  
GPIO.setmode(GPIO.BCM)  
GPIO.setup(pin, GPIO.IN, pull_up_down = GPIO.PUD_UP)  

# Our function on what to do when the button is pressed  
def Shutdown(channel):  
    os.system("sudo shutdown -h now")
 
# Add our function to execute when the shutdown button is pressed  
GPIO.add_event_detect(pin, GPIO.FALLING, callback = Shutdown, bouncetime = 2000)  


# ------ Definition Set Up -----------#
def lightSensor ():		#Light Sensor
        reading = 0
        GPIO.setup(LDRpin, GPIO.OUT)
        GPIO.output(LDRpin, GPIO.LOW)
        time.sleep(0.1)

        GPIO.setup(LDRpin, GPIO.IN)
        # This takes about 1 millisecond per loop cycle
        while (GPIO.input(LDRpin) == GPIO.LOW):
                reading += 1
        return reading

def pumpOne():			#Pump One
	GPIO.setup(bedone, GPIO.IN)
	if GPIO.input(bedone):
		valveone.off()
		valveoneled.on()
		reading = False				#moisture Not Present
	else:
		reading = True 				#moisture present
		valveone.on()
		valveoneled.off()
	return reading

def pumpTwo():			#Pump two
	GPIO.setup(bedtwo, GPIO.IN)
	if GPIO.input(bedtwo):
		valvetwo.off()
		valvetwoled.on()
		reading = False
	else:
		reading = True
		valvetwo.on()
		valvetwoled.off()
	return reading
	
def bothPumps():		#Turn both the pumps on
	if pumpOne() == True:
		reading = False
		
	elif pumpTwo() == True:
		reading = False
	
	else:
		reading = True
	return reading

def moisterPresent():	#Determin if both bed have mositer
	if pumpOne() == False:
		reading = False
	
	elif pumpTwo() == False:
		reading = False
		
	else:
		reading = True
	return reading
	
def sensorSave():		#Save the light, temperature and humidity data
	f=open('beddata.txt','a')
	now = datetime.datetime.now()
	timestamp = now.strftime("%d/%m/%Y %H:%M")
	data = str(timestamp)+"- " 'Temp={0:0.1f}*C  Humidity={1:0.1f}%'.format(temperature, humidity) + ' Light Levels= ' + str(lightSensor())
	f.write(str(data) + '\r\n')
	
def pumpSave():			#Save both pump states
	f=open('beddata.txt','a')
	now = datetime.datetime.now()
	timestamp = now.strftime("%d/%m/%Y %H:%M")
	data = str(timestamp)+"- " + 'Is there moisture in bed One, ' + str(pumpOne())+ '. Is there moisture in bed Two,  ' + str(pumpTwo())
	f.write(str(data) + '\r\n')	

#-------------- Main Program ------------ #

while True:
	if moisterPresent() == True:
		print lightSensor()
		print('Temp={0:0.1f}*C  Humidity={1:0.1f}%'.format(temperature, humidity))
		sensorSave()
		time.sleep(clock)

	elif bothPumps() == True:
		print "Both pumps are running"
		pumpSave()
		time.sleep(pumptime)
	
	elif pumpOne() == True:
		print "moisture not present within bed Two"
		pumpSave()
		time.sleep(pumptime)
		
	elif pumpTwo() == True:
		print "moisture not present within bed One"
		pumpSave()
		time.sleep(pumptime)
	
	else:	
		print 'Waiting'
		time.sleep(3)
