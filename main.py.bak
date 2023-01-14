#!/usr/bin/python
# -*- coding:utf-8 -*-
from bottle import get,post,run,route,request,template,static_file
from PCA9685 import PCA9685
import threading
import os
import socket
import time

pwm = PCA9685(0x40)
pwm.setPWMFreq(50)

#Set the Horizontal vertical servo parameters
HPulse = 1500  #Sets the initial Pulse
HStep = 0      #Sets the initial step length
VPulse = 1375 #Sets the initial Pulse
VStep = 0      #Sets the initial step length

start = int(time.time())

pwm.setServoPulse(1,HPulse)
pwm.setServoPulse(0,VPulse)

@get("/")
def index():
    return template("index.html")
    
@route('/<filename>')
def server_static(filename):
    return static_file(filename, root='./')

@route('/fonts/<filename>')
def server_fonts(filename):
    return static_file(filename, root='./fonts/')
    
@post("/cmd")
def cmd():
    global HStep,VStep,HPulse,VPulse
    code = request.body.read().decode()
    print(("code ",code))
    print('HPulse ',HPulse , 'VPulse ', VPulse )
    # speed = request.POST.get('speed')
    # print(code)
    # if(speed != None):

        # print(speed)
    if code == "reset":
        HPulse = 1500
        VPulse = 1373
        print(code)
        pwm.setServoPulse(0,VPulse)
        pwm.setServoPulse(1,HPulse)
    elif code == "stop":
        HStep = 0
        VStep = 0
        print(code)
    elif code == "up":
        VStep = -5
        print("up")
    elif code == "down":
        VStep = 5
        print(code)
    elif code == "left":
        HStep = 5
        print(code)
    elif code == "right":
        HStep = -5
        print(code)
    return "OK"


def camera():
    Road = 'mjpg-streamer/mjpg-streamer-experimental/'
    os.system('./' + Road + 'mjpg_streamer -i "./' + Road + '/input_uvc.so -f 30 -r 800x600" -o "./' + Road + 'output_http.so -w ./' + Road + 'www"') 
    # ./mjpg_streamer -i "./input_uvc.so" -o "./output_http.so -w ./www"
    
def timerfunc():
    global HPulse,VPulse,HStep,VStep,pwm,start
    
    if(HStep != 0):
        HPulse += HStep
        if(HPulse >= 2500): 
            HPulse = 2500
        if(HPulse <= 500):
            HPulse = 500
        #set channel 2, the Horizontal servo
        pwm.start_PCA9685()
        pwm.setServoPulse(1,HPulse)
        start = int(time.time())        


    if(VStep != 0):
        VPulse += VStep
        if(VPulse >= 2500): 
            VPulse = 2500
        if(VPulse <= 500):
            VPulse = 500
        #set channel 3, the vertical servo
        pwm.start_PCA9685()
        pwm.setServoPulse(0,VPulse)   
        start = int(time.time())
        
    end = int(time.time())
    if((end - start) > 3):
        pwm.exit_PCA9685()
        start = int(time.time())

    
    global t        #Notice: use global variable!
    t = threading.Timer(0.02, timerfunc)
    t.start()
    
try:
    t2 = threading.Thread(target = camera)
    t2.setDaemon(True)
    t2.start()
        
    t = threading.Timer(0.02, timerfunc)
    t.setDaemon(True)
    t.start()


    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    s.connect(('8.8.8.8', 80))
    localhost = s.getsockname()[0]

    run(host=localhost, port="8001")
except:
    pwm.exit_PCA9685()
    print("\nProgram end")
    exit()
