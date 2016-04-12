# REWC1
Rationalization of Electricity and Water Consumption
PYTHON SCRIPT

import os
import time
from Adafruit_CharLCD import Adafruit_CharLCD
from subprocess import *
from time import sleep, strftime
import RPi.GPIO as GPIO
GPIO.setmode(GPIO.BOARD)
GPIO.setwarnings(False)
GPIO.setup(3, GPIO.IN)  # push button 1
GPIO.setup(5, GPIO.IN)  # push button 2
GPIO.setup(32, GPIO.IN)  # push button 3
GPIO.setup(8, GPIO.IN)  # push button 4
GPIO.setup(19, GPIO.IN)  # push button 5
GPIO.setup(38,GPIO.OUT) # relay module 1
GPIO.setup(36,GPIO.OUT) # relay module 2
GPIO.setup(22,GPIO.OUT) #remote control channel 1 ON
GPIO.setup(18,GPIO.OUT) #remote control channel 1 OFF
GPIO.setup(24,GPIO.OUT) #remote control channel 2 ON
GPIO.setup(16,GPIO.OUT) #remote control channel 2 OFF
GPIO.setup(40,GPIO.OUT) # servo motor for analogue power comsumpution
GPIO.setup(15,GPIO.OUT) # servo motor for analogue Water level
GPIO.setup(32,GPIO.OUT) # Buzzer

Servo=GPIO.PWM(40,50) 
Servo1=GPIO.PWM(15,50)
servostate=11.8

Servo.start(1)
Servo.ChangeDutyCycle(servostate)

Servo1.start(1)
Servo1.ChangeDutyCycle(servostate)

p=0
servostate=11.8
stat1=0
stat2=0
stat3=0
stat4=0
stat5=0
statall=0

Pfan=60
Pheater=1600
PRefrigerator=50
PAC=1290

KWATTperHOUR=77 # 77 LBP it is average of KWATTperHOUR price in LEBANON

TRIG = 10 # ultrasonic triger                                 
ECHO = 21 # ultrasonic echo
GPIO.setup(TRIG,GPIO.OUT)                
GPIO.setup(ECHO,GPIO.IN)                   
length=50
width =50
height=150

lcd = Adafruit_CharLCD()

cmd = "ip addr show eth0 | grep inet | awk '{print $2}' | cut -d/ -f1"
#output gpio
GPIO.output(36,GPIO.HIGH)
GPIO.output(38,GPIO.HIGH)
GPIO.output(18,GPIO.LOW)
GPIO.output(22,GPIO.LOW)
GPIO.output(24,GPIO.LOW)
GPIO.output(16,GPIO.LOW)
GPIO.output(32,GPIO.LOW)

# monitor of status of input gpio
print GPIO.input(3)
print GPIO.input(5)
print GPIO.input(8)
print GPIO.input(19)

lcd.begin(16, 1)

def water_level():
        
  GPIO.output(TRIG, False)                 #Set TRIG as LOW
  print "Waitng For Sensor To Settle"
  time.sleep(2)                            #Delay of 2 seconds
  level=0
  GPIO.output(TRIG, True)                  #Set TRIG as HIGH
  time.sleep(0.00001)                      #Delay of 0.00001 seconds
  GPIO.output(TRIG, False)                 #Set TRIG as LOW

  while GPIO.input(ECHO)==0:               #Check whether the ECHO is LOW
    pulse_start = time.time()              #Saves the last known time of LOW pulse

  while GPIO.input(ECHO)==1:               #Check whether the ECHO is HIGH
    pulse_end = time.time()                #Saves the last known time of HIGH pulse 

  pulse_duration = pulse_end - pulse_start #Get pulse duration to a variable

  level = pulse_duration * 17150        #Multiply pulse duration by 17150 to get level
  level = round(level, 0)            #Round to two decimal points

  if level > 2 and level < 60:      #Check whether the level is within range
        #height=40-level      
        #volume=lenght * width * height # v in cm cubic
        #liter  = volume / 1000 # 1000 cm cubic = 1 liter \/
        #time.sleep(0.3)
           
    if level < 11: # minn level
          
          Servo1.ChangeDutyCycle(2)
          
    if level >11 and level <18: 

          Servo1.ChangeDutyCycle(5)
          
    if level >18 and level <25: 

          Servo1.ChangeDutyCycle(9)
          
    if level > 35:  # max level 
          
          Servo1.ChangeDutyCycle(11.8)
  else:
    print "Out Of Range"                   #display out of range

def SWITCHON ():

	
	GPIO.output(32,GPIO.HIGH)
	time.sleep(.1)
	GPIO.output(32,GPIO.LOW)
	time.sleep(.1)
	water_level()

	
def SWITCHOFF ():

	
	GPIO.output(32,GPIO.HIGH)
	time.sleep(.1)
	GPIO.output(32,GPIO.LOW)
	time.sleep(.1)
	GPIO.output(32,GPIO.HIGH)
	time.sleep(.1)
	GPIO.output(32,GPIO.LOW)
	time.sleep(1)
	water_level()

def buzzer ():

	
	GPIO.output(32,GPIO.HIGH)
	time.sleep(.1)
	GPIO.output(32,GPIO.LOW)
	time.sleep(.1)
	GPIO.output(32,GPIO.HIGH)
	time.sleep(.1)
	GPIO.output(32,GPIO.LOW)
	time.sleep(.1)
	GPIO.output(32,GPIO.HIGH)
	time.sleep(.1)

	
	GPIO.output(32,GPIO.LOW)
	time.sleep(.2)
	GPIO.output(32,GPIO.HIGH)
	time.sleep(.2)
	GPIO.output(32,GPIO.LOW)
	time.sleep(.2)
	GPIO.output(32,GPIO.HIGH)
	time.sleep(.2)
	GPIO.output(32,GPIO.LOW)
	time.sleep(.2)
	GPIO.output(32,GPIO.HIGH)
	time.sleep(.2)
	GPIO.output(32,GPIO.LOW)
	time.sleep(.2)

	
	GPIO.output(32,GPIO.HIGH)
	time.sleep(.1)
	GPIO.output(32,GPIO.LOW)
	time.sleep(.1)
	GPIO.output(32,GPIO.HIGH)
	time.sleep(.1)
	GPIO.output(32,GPIO.LOW)
	time.sleep(.1)
	GPIO.output(32,GPIO.HIGH)
	time.sleep(.1)
	GPIO.output(32,GPIO.LOW)
	time.sleep(.7)
	water_level()
	

def run_cmd(cmd):
    p = Popen(cmd, shell=True, stdout=PIPE)
    output = p.communicate()[0]
    return output

#water_leve()

# Welcome msg

lcd.clear()
lcd.message("  Almahdi school\n  Baalbeck ")
time.sleep(3)
lcd.clear()
lcd.message("  Welcome \n  REWC ")
time.sleep(2)
lcd.clear()
lcd.message("  Powered by \n  Raspberry PI ")

while True:
   
     if GPIO.input(3)==False and stat1==0:
        SWITCHON ()
        GPIO.output(38,GPIO.LOW)
        stat1=1
        p = p + Pfan
        servostate = servostate - 2.9
        Servo.ChangeDutyCycle(servostate) 
        time.sleep(0.2)
        print ("Fan  turned ON=")
        print ("consumption=")
        print p
        #lcd.clear()
        lcd.clear()
        lcd.message(" REWC  \n  Fan turned ON ")
        time.sleep(1)
        lcd.clear()
        
        temp = "{:0.1f} * WATT".format(p)
        lcd.message ("consumption:\n")
	lcd.message(temp)
	time.sleep(2)
	lcd.clear()
		
	temp2= "{:0.1f} * LBP".format(p*144)
	lcd.message ("COST per MONTH :\n")
	lcd.message(temp2)
	time.sleep(2)
	lcd.clear()
		
	temp3= "{:0.1f} * LBP".format(p*52560)
	lcd.message ("COST per YEAR :\n")
 	lcd.message(temp3)
        print " GPIO.input(3) = "
        print GPIO.input(3)
        
        if p>2800:
            buzzer()
            buzzer()
            
     if GPIO.input(3)==False and stat1==1:
        SWITCHOFF ()
        GPIO.output(38,GPIO.HIGH)
        p = p - Pfan
        servostate = servostate + 2.9
        Servo.ChangeDutyCycle(servostate) 
        stat1=0
        time.sleep(2)
        print ("Fan  turned OFF=")
        print ("consumption=")
        print p
		
        #lcd.clear()
        lcd.clear()
        lcd.message(" REWC  \n  Fan turned OFF ")
        time.sleep(1)
        lcd.clear()
        
        temp = "{:0.1f} * WATT".format(p)
        lcd.message ("consumption:\n")
	lcd.message(temp)
	time.sleep(2)
	lcd.clear()
		
	temp2= "{:0.1f} * LBP".format(p*144)
	lcd.message ("COST per MONTH :\n")
	lcd.message(temp2)
	time.sleep(2)
	lcd.clear()
		
	temp3= "{:0.1f} * LBP".format(p*52560)
	lcd.message ("COST per YEAR :\n")
	lcd.message(temp3)
        
        
        if p>2800:
            buzzer()
            buzzer()


     if GPIO.input(5)==False and stat2==0:
        SWITCHON ()
        GPIO.output(36,GPIO.LOW)
        stat2=1
        p = p + Pheater
        servostate = servostate - 2.9
        Servo.ChangeDutyCycle(servostate) 
        time.sleep(2)
        print ("consumption=")
        print p
        #lcd.clear()
        lcd.clear()
        lcd.message(" REWC  \n  Heater turned ON ")
        time.sleep(1)
        lcd.clear()
        
        temp = "{:0.1f} * WATT".format(p)
        lcd.message ("consumption:\n")
	lcd.message(temp)
	time.sleep(2)
	lcd.clear()
		
	temp2= "{:0.1f} * LBP".format(p*144)
	lcd.message ("COST per MONTH :\n")
	lcd.message(temp2)
	time.sleep(2)
	lcd.clear()
		
	temp3= "{:0.1f} * LBP".format(p*52560)
	lcd.message ("COST per YEAR :\n")
	lcd.message(temp3)
        
        
        if p>2800:
            buzzer()
            buzzer()

            
     if GPIO.input(5)==False and stat2==1:
        SWITCHOFF ()
        GPIO.output(36,GPIO.HIGH)
        p = p - Pheater
        servostate = servostate + 2.9
        Servo.ChangeDutyCycle(servostate) 
        stat2=0
        time.sleep(2)
        print ("consumption=")
        print p
        
		#lcd.clear()
        lcd.clear()
        lcd.message(" REWC  \n  Heater turned OFF ")
        time.sleep(1)
        lcd.clear()
        
        temp = "{:0.1f} * WATT".format(p)
        lcd.message ("consumption:\n")
	lcd.message(temp)
	time.sleep(2)
	lcd.clear()
		
	temp2= "{:0.1f} * LBP".format(p*144)
	lcd.message ("COST per MONTH :\n")
	lcd.message(temp2)
	time.sleep(2)
	lcd.clear()
		
	temp3= "{:0.1f} * LBP".format(p*52560)
	lcd.message ("COST per YEAR :\n")
	lcd.message(temp3)
        
        
        if p>2800:
            buzzer()
            buzzer()
        
		


     if GPIO.input(19)==False and stat3==0:   # general on channel 1
        SWITCHON ()
        GPIO.output(22,GPIO.HIGH)#high on 1
        time.sleep(.2)
        GPIO.output(22,GPIO.LOW)#high off 1
        stat3=1
        p = p + PRefrigerator
        servostate = servostate - 2.9
        Servo.ChangeDutyCycle(servostate) 
        time.sleep(2)
        print ("consumption=")
        print p
        
		#lcd.clear()
        lcd.clear()
        lcd.message(" REWC  \n  Refrigerator  ON ")
        time.sleep(1)
        lcd.clear()
        
        temp = "{:0.1f} * WATT".format(p)
        lcd.message ("consumption:\n")
	lcd.message(temp)
	time.sleep(2)
	lcd.clear()
		
	temp2= "{:0.1f} * LBP".format(p*144)
	lcd.message ("COST per MONTH :\n")
	lcd.message(temp2)
	time.sleep(2)
	lcd.clear()
		
	temp3= "{:0.1f} * LBP".format(p*52560)
	lcd.message ("COST per YEAR :\n")
	lcd.message(temp3)
        
        
        if p>2800:
            buzzer()
            buzzer()

        
     if GPIO.input(19)==False and stat3==1:   # general off channel 1
        SWITCHOFF ()
        GPIO.output(18,GPIO.HIGH)#high on 1
        time.sleep(.2)
        GPIO.output(18,GPIO.LOW)#high off 1
        p = p - PRefrigerator
        servostate = servostate + 2.9
        Servo.ChangeDutyCycle(servostate) 
        stat3=0
        time.sleep(2)
        print ("consumption=")
        print p
        
		#lcd.clear()
        lcd.clear()
        lcd.message(" REWC  \n  Refrigerator  OFF ")
        time.sleep(1)
        lcd.clear()
        
        temp = "{:0.1f} * WATT".format(p)
        lcd.message ("consumption:\n")
	lcd.message(temp)
	time.sleep(2)
	lcd.clear()
		
	temp2= "{:0.1f} * LBP".format(p*144)
	lcd.message ("COST per MONTH :\n")
	lcd.message(temp2)
	time.sleep(2)
	lcd.clear()
		
	temp3= "{:0.1f} * LBP".format(p*52560)
	lcd.message ("COST per YEAR :\n")
	lcd.message(temp3)
        
        
        if p>2800:
            buzzer()
            buzzer()


     if GPIO.input(8)==False and stat4==0: # general ON channel 2
        SWITCHON ()
        GPIO.output(24,GPIO.HIGH) 
        time.sleep(.2)
        GPIO.output(24,GPIO.LOW)
        stat4=1
        p = p + PAC
        servostate = servostate - 2.9
        Servo.ChangeDutyCycle(servostate) 
        time.sleep(2)
        print ("consumption=")
        print p
        
		#lcd.clear()
        lcd.clear()
        lcd.message(" REWC  \n  AC turned ON ")
        time.sleep(1)
        lcd.clear()
        
        temp = "{:0.1f} * WATT".format(p)
        lcd.message ("consumption:\n")
	lcd.message(temp)
	time.sleep(2)
	lcd.clear()
		
	temp2= "{:0.1f} * LBP".format(p*144)
	lcd.message ("COST per MONTH :\n")
	lcd.message(temp2)
	time.sleep(2)
	lcd.clear()
		
	temp3= "{:0.1f} * LBP".format(p*52560)
	lcd.message ("COST per YEAR :\n")
	lcd.message(temp3)
        
        
        if p>2800:
            buzzer()
            buzzer()

        
     if GPIO.input(8)==False and stat4==1:   # general off channel 2
        SWITCHOFF ()
        GPIO.output(16,GPIO.HIGH)
        time.sleep(.2)
        GPIO.output(16,GPIO.LOW)
        p = p - PAC
        servostate = servostate + 2.9
        Servo.ChangeDutyCycle(servostate) 
        stat4=0
        time.sleep(2)
        print ("consumption=")
        print p
        
		#lcd.clear()
        lcd.clear()
        lcd.message(" REWC  \n  AC turned OFF ")
        time.sleep(1)
        lcd.clear()
        
        temp = "{:0.1f} * WATT".format(p)
        lcd.message ("consumption:\n")
	lcd.message(temp)
	time.sleep(2)
	lcd.clear()
		
	temp2= "{:0.1f} * LBP".format(p*144)
	lcd.message ("COST per MONTH :\n")
	lcd.message(temp2)
	time.sleep(2)
	lcd.clear()
		
	temp3= "{:0.1f} * LBP".format(p*52560)
	lcd.message ("COST per YEAR :\n")
	lcd.message(temp3)
        
        
        if p>2800:
            buzzer()
            buzzer()




     
