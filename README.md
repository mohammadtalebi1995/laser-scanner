# mohammad
# Master of Communication Networks and R&amp;D Specialist at Novinilya company

# In this Version we will try to find the relative distance of laser line
# Coded by : Mohammad Talebi[lazer_tst.zip](https://github.com/mohammadtalebi1995/laser-scanner/files/7675160/lazer_tst.zip)

import datetime
# import sys
# import time
# import timeit
from pypylon import pylon
from numpy import * # import all of the numpy comp.
import math
# import matplotlib
import matplotlib.pyplot as plt # for ploting the scanned line
import cv2

startpoint = datetime.datetime.now()

deviceInfo = pylon.DeviceInfo()
deviceInfo.SetDeviceClass("BaslerBcon")
camera = pylon.InstantCamera(pylon.TlFactory.GetInstance().CreateFirstDevice())
camera.StartGrabbing(pylon.GrabStrategy_LatestImageOnly)
converter = pylon.ImageFormatConverter()

# Convert the image to Gray Scale Using the pyplon basic facilities
converter.OutputPixelFormat = pylon.PixelType_RGB8packed

grabResult = camera.RetrieveResult(5000, pylon.TimeoutHandling_ThrowException)

image = converter.Convert(grabResult)
img = image.GetArray()


# list of varaibles
'''
    img => source image
    dim => resize ratio (we will resize the picture to optimize the process and time)
'''
# Fetching the source image --- src ---
#img = cv2.imread("7.png") # you can change the likable frame here

# resize it for watchability
w = int(img.shape[1])
h = int(img.shape[0])
dim = (w//5, h//5)
img = cv2.resize(img, dim, interpolation = cv2.INTER_AREA)
show_img = img # we save a healthy img for ploting in end
print("read time: ", datetime.datetime.now() - startpoint)
# Filter the source image => only lazered
img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# img = cv2.GaussianBlur(img, (11, 11), 0)
# thr, img = cv2.threshold(img, 17, 255, cv2.THRESH_BINARY)

# img = cv2.erode(img, None, iterations=1)
img = cv2.dilate(img, None, iterations=4)

for x in range(0,h//5): # in this loop we will filter the light of lazer
    for y in range(0,w//5):
        if img[x][y] < 100 : # lazer ther. = 235 it mean all of pixels under 235 will be convert to 0 (absolute black)
            img[x][y] = 0


cv2.imshow("thresh" , img)
cv2.waitKey(0)
print("Filtering finished !") # after flitering finished this command will be print in cmd
print("resize time: ", datetime.datetime.now() - startpoint)
# Slicing the image to 74 lined picture

x_axiz = [] # X index
cp_list =[] # Z index

for i in range(0,math.ceil(w/30)): # in this loop we will find the position of each cloud point by Res.6 |ex. 74 for Galaxy J7 core shotted pic
    sliced = img[:,(i*6):((i+1)*6)]
    x_axiz.append(i)
    cv2.imshow("Image (ECP) : Extract Cloud Point" , show_img)
    key = cv2.waitKey(5) #line scan delay time
    if key == 27:
        break # if you press Esc the program will be closed
    # Draw Line on Image
    cv2.line(show_img,((i+1)*6,0),((i+1)*6,588),(0,215,0),1)

    # calculate the location of white particle
    xy_val = sliced.nonzero()
    y_val = median(xy_val[0])
    pixel_len = 275 - y_val #275 for is the y-axis zero cordinate for Galaxy J7 Core shotted pic
    if math.isnan(y_val) is True:
        pixel_len = 0
    leng = pixel_len*0.06

    # print and filtering the pixel
    print('CP #{} info | pix.len : {} | height : {}'.format(i,pixel_len,leng))
    cp_list.append(leng)
print("read time2: ", datetime.datetime.now() - startpoint)
# print list of cloud point
# print(cp_list)

# final data print
cv2.imshow("Image (ECP) : Extract Cloud Point" , show_img)
# cv2.waitKey(0)

# Ploting the data using the pyplot lib
fig, ax = plt.subplots()

ax.plot(x_axiz, cp_list)
ax.set(xlabel='cloud point (# no unit)', ylabel='height',title='Frame')
ax.grid()
fig.savefig("plot.png")
print(datetime.datetime.now() - startpoint)
plt.show()
