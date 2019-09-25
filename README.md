# MIT-Project
#This is done for the UCSC as a final project in MIT Degree.
# Please install Python 3.0, PyQt5 and OpenCV inorder to run the project. Import all the mentioned libraries too. 

import sys
import numpy as np
import cv2
import os.path
import pandas as pd #date & time

from PyQt5 import QtCore, QtWidgets
from PyQt5.QtWidgets import QMainWindow, QWidget, QLabel, QLineEdit, QSlider,  QPushButton, QGridLayout, QVBoxLayout, QApplication
from PyQt5.QtCore import QSize, Qt    
import urllib.request
from PyQt5.QtGui import QPixmap, QPainter, QColor, QPen, QIcon


import smtplib 
from email.mime.multipart import MIMEMultipart 
from email.mime.text import MIMEText 
from email.mime.base import MIMEBase 
from email import encoders 

ip = None
ThresholdVal = 500
url = None
lowMargineT_level = 100
maxMargineT_level = 50000
setT_level = 500
skipcount = 100
schek = 0

def saveImage(ip):
        print('Connecting...' + ip)
        fgbg = cv2.bgsegm.createBackgroundSubtractorMOG();
        url = 'http://'+ ip +':8080/shot.jpg' 
        #url = 'http://test1:testing@192.168.8.100:8080/shot.jpg' 

        
        global ThresholdVal
        global schek

        while True:
            imgRes= urllib.request.urlopen(url)
            imgNp= np.array( bytearray (imgRes.read()),dtype=np.uint8)
            img=cv2.imdecode(imgNp, -1)
        

            gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
            fgmask = fgbg.apply(gray)
            cv2.imshow('diff_frame', fgmask);
            cv2.imshow('Original',img)

            cv2.namedWindow('Original',cv2.WINDOW_NORMAL)
            cv2.resizeWindow('Original', 640,360)

            cv2.namedWindow('diff_frame',cv2.WINDOW_NORMAL)
            cv2.resizeWindow('diff_frame', 640,360)


            n_white_pix = np.sum(fgmask == 255)
            
            
            if (schek != 0 and schek < 200) :
                schek = schek + 1
                print('Waiting!!' , schek)
                        
            if ((n_white_pix > ThresholdVal) and (schek == 0 or schek == 200)):
                storeImage(img)
                 
            cv2.waitKey(10)

        if cv2.waitKey(30) & 0xFF == ord('q'):
            system.quit()
        else:
            print('ERROR!')
            system.quit()


class MainWindow(QMainWindow):
    def __init__(self):
        QMainWindow.__init__(self)
        # super().__init__()

        ip = None
        global ThresholdVal
        global lowMargineT_level 
        global maxMargineT_level
        global setT_level
        url = None

        # Set window background color
        self.setAutoFillBackground(True)
        p = self.palette()
        p.setColor(self.backgroundRole(), QColor(0, 153, 153))
        self.setPalette(p)

        self.setMinimumSize(QSize(420, 440))   #320 ,140 
        self.setMaximumSize(QSize(420, 440))
        self.setWindowTitle("Intelligent Security System") 

        sld = QSlider(Qt.Horizontal, self)
        sld.setGeometry(28, 315, 357, 30) 

        self.label_img = QLabel(self)
        pixmap = QPixmap('icone_1.png')
        self.label_img.setPixmap(pixmap)
        self.label_img.move(150, 10)
        self.label_img.resize(120, 120)

        self.nameLabe2 = QLabel(self)
        self.nameLabe2.setText('ADJUST THE THRESHOLD LEVEL')
        self.nameLabe2.move(7,295)
        self.nameLabe2.resize(190, 10)

        self.nameLabe3 = QLabel(self)
        self.nameLabe3.setText('ENTER YOUR CAMERA IP HERE')
        self.nameLabe3.move(132,115)
        self.nameLabe3.resize(190, 120)
        
        self.line = QLineEdit(self)
        self.line.move(110, 190)
        self.line.resize(200, 32)
    
        self.line.setText('Enter your IP')
        self.line.setText('192.168.1.1')

        self.lowMargine = QLabel(self)
        self.lowMargine.setText(str(lowMargineT_level))
        self.lowMargine.move(3,260)
        self.lowMargine.resize(30, 130)

        self.lowMargine = QLabel(self)
        self.lowMargine.setText(str(maxMargineT_level))
        self.lowMargine.move(389,265)
        self.lowMargine.resize(190, 120)

        self.Tval = QLabel(self)
        self.Tval.setText('THRESHOLD LEVEL : ' + str(setT_level))
        self.Tval.move(140,350)
        self.Tval.resize(190, 10)


        self.display = QLabel(self)
        self.display.setText(str(maxMargineT_level))

        self.display.move(389,265)
        self.display.resize(190, 120)
        
        pybutton = QPushButton('OK', self)
        pybutton.clicked.connect(self.clickMethod)
        pybutton.resize(200,32)
        pybutton.move(110, 230) 

        pybutton2 = QPushButton('EXIT', self)
        pybutton2.clicked.connect(self.close)   
        pybutton2.resize(60,32)
        pybutton2.move(355, 405) 


        #scroll_1
        sld.setRange(lowMargineT_level,maxMargineT_level)
        sld.setValue(setT_level)
        sld.setTickPosition(QSlider.TicksBelow)
        sld.setTickInterval(100)
        sld.valueChanged[int].connect(self.valuechange)


    def valuechange(self,value):
        print('New ThresholdVal Level : ', value)
        global ThresholdVal
        ThresholdVal = value
        self.Tval.setText(str(ThresholdVal))
        self.Tval.setText('THRESHOLD LEVEL :' + str(ThresholdVal))
        #saveImage(ip)
        if ip is not None:
            print(ip)

    def clickMethod(self):
        ip = self.line.text()
        if ip != 'Enter your IP' :
            print('Loging...') 
            print(ip)
            saveImage(ip)
        else :
            print('Set IP First')   


def RNamefn():
    rNamefn = pd.datetime.now()
    rename = str(rNamefn.day) + str(rNamefn.month) + str(rNamefn.year) + '_' + str(rNamefn.hour) + str(rNamefn.minute) + str(rNamefn.second)
    print(rename)
    return(rename)

def storeImage(img) :
    global schek
    schek == skipCal()
    print("point",schek)
    RName = str(RNamefn()) +'.jpg'
    cv2.imwrite(RName, img)
    sendMail(RName, img)
    print(ThresholdVal)
   

def skipCal():
    global schek
    schek = 1
    print('Waiting!!! =' , schek)
    return(schek)    
            


def sendMail(RName, img):
    fromaddr = 'secupdatenofifymit@gmail.com'
    toaddr = 'amalchaminda81@gmail.com'
    print(RName, 'Sending a mail')

    # instance of MIMEMultipart 
    msg = MIMEMultipart() 
      
    # storing the senders email address   
    msg['From'] = fromaddr 
      
    # storing the receivers email address  
    msg['To'] = toaddr 
      
    # storing the subject  
    msg['Subject'] = 'Security Update'
      
    # string to store the body of the mail 
    body = 'Security cam found object'
      
    # attach the body with the msg instance 
    msg.attach(MIMEText(body, 'plain')) 
      
    # open the file to be sent  
    filename = RName
    attachment = open(filename, "rb") 
      
    # instance of MIMEBase and named as p 
    p = MIMEBase('application', 'octet-stream')
      
    # To change the payload into encoded form 
    p.set_payload((attachment).read()) 
      
    # encode into base64 
    encoders.encode_base64(p) 
       
    p.add_header('Content-Disposition', "attachment; filename= %s" % filename) 
      
    # attach the instance 'p' to instance 'msg' 
    msg.attach(p) 
      
    # creates SMTP session 
    s = smtplib.SMTP('smtp.gmail.com', 587) 
      
    # start TLS for security 
    s.starttls() 
      
    # Authentication 
    s.login(fromaddr, 'project2019') 
      
    # Converts the Multipart msg into a string 
    text = msg.as_string() 
      
    # sending the mail 
    s.sendmail(fromaddr, toaddr, text) 
      
    # terminating the session 
    s.quit()    


if __name__ == "__main__":
    
    
    app = QtWidgets.QApplication([])
    mainWin = MainWindow()
    mainWin.show()
    sys.exit( app.exec_() )

    cap.release()
    cv2.destroyAllWindows()



   
