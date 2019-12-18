---
layout: post
title:  "Optimizing live video edge detection in Open CV (Python)"
meta: "A simple method of varying the arguments to image processing functions in real time with sliders."
date:   2019-09-26 09:39:00 +0100
categories: jekyll update
permalink: 
---

### Optimizing live video edge detection in Open CV (Python)

As a follow up to [this]({{site.baseurl}}/open-cv-image-review/) post on tweaking images to achieve better edge detection results I wrote this script which allows you to vary the inputs into median blur, Gaussian blur, bilateral filter, and the upper and lower limits for canny edge detection. It then applies a laplacian gradient filter to the resulting image, this appears to have the greatest effect on the results.

It is important to note that the filters are applied in the order stated above so the effects of the first will influence others down stream. In future I hope to build an application that can move the filters up or down the chain in real time to better evaluate their effects. This will probably require a more advanced GUI than Open CV has natively. 

### The code
**************

``` python
import cv2

def on_change(self): # Does nothing but function must be passed to cv2.createTrackbar
    pass

cap = cv2.VideoCapture(0) # Define capture device 0 = first capture device etc
cv2.namedWindow('Canny',cv2.WINDOW_AUTOSIZE) # Create a named window to add trackbars and video frames to

# Define maximum and starting values for each trackbar, min values are always 0

d_start, d_end = 6, 100 # Bilateral filter
sc_start, sc_end = 30, 150
ss_start, ss_end = 1, 150


c_lower_l, c_lower_u = 20, 255 # Limits for canny edge detection
c_upper_l, c_upper_u = 25, 255

blur_s, blur_f = 1, 15 # Gaussian blur

m_blur_s, m_blur_f = 0, 99 # Median blur

#Create trackbars
cv2.createTrackbar("D", "Canny", d_start, d_end, on_change) # Bilateral filter
cv2.createTrackbar("Sigma Colour", "Canny", sc_start, sc_end, on_change)
cv2.createTrackbar("Sigma Space", "Canny", ss_start, ss_end, on_change)

cv2.createTrackbar("Lower", "Canny", c_lower_l, c_lower_u, on_change) # Limits for canny edge detection
cv2.createTrackbar("Upper", "Canny", c_upper_l, c_upper_u, on_change)

cv2.createTrackbar("Gaussian Blur", "Canny", blur_s, blur_f, on_change) # Gaussian blur

cv2.createTrackbar("Median Blur", "Canny", m_blur_s, m_blur_f, on_change) # Median blur


cv2.startWindowThread() # Required with namedWindow()

# Define list of odd numbers, required for median blur.
med = list(range(1, 200, 2)) 

while(True):
    d = cv2.getTrackbarPos("D","Canny")
    sc = cv2.getTrackbarPos("Sigma Colour","Canny")
    ss = cv2.getTrackbarPos("Sigma Space","Canny")

    l = cv2.getTrackbarPos("Lower","Canny") #Limits for canny edge detection
    u = cv2.getTrackbarPos("Upper","Canny")

    m = cv2.getTrackbarPos("Median Blur","Canny") # Median blur
    m = med[m] # Select m from list of odd numbers, faster than conditionals?

    b = cv2.getTrackbarPos("Gaussian Blur","Canny") # Gaussian blur
    
    if b == 0: # Ensure that b is not 0, this would cause error in cv2.blur()
        b = 1

    _, frame = cap.read() # Capture frame, ret not used so assigned to _
    fps = cap.get(cv2.CAP_PROP_FPS)
    print(fps)
    
    # Process frame to (hopefully) improve edge detection
    frame = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    median = cv2.medianBlur(frame, m)
    gaussian = cv2.blur(median,(b,b))
    processed = cv2.bilateralFilter(gaussian,d,sc,ss)
    laplacian = cv2.Laplacian(processed,cv2.CV_64F)
    canny = cv2.Canny(processed,l,u, L2gradient = True)

    # Open windows showing each step of the image processing.
    # Comment out lines which are un-nessesery if your computer slows down (like mine)
    cv2.imshow('original', frame)
    cv2.imshow("median", median)
    cv2.imshow("Gaussian", gaussian)
    cv2.imshow('Bilateral', processed)
    cv2.imshow('Laplacian', laplacian)
    cv2.imshow('Canny', canny)
    
    if cv2.waitKey(1) & 0xFF == ord('q'): # Quits on q key press
        break

cap.release()
cv2.destroyAllWindows()
```