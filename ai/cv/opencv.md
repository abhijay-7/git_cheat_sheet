# OpenCV Complete Operations Cheatsheet

## Installation
```bash
pip install opencv-python
pip install opencv-contrib-python  # For extra modules
```

## Importing
```python
import cv2
import numpy as np
```

---

## 1. Reading and Displaying Images

### Read Image
```python
img = cv2.imread('image.jpg')  # Read in BGR format
img_gray = cv2.imread('image.jpg', 0)  # Read as grayscale
img_unchanged = cv2.imread('image.jpg', cv2.IMREAD_UNCHANGED)  # Include alpha channel
img_color = cv2.imread('image.jpg', cv2.IMREAD_COLOR)  # Force color
```

### Display Image
```python
cv2.imshow('Window Name', img)
cv2.waitKey(0)  # Wait indefinitely, 0 = wait forever, value in ms for timeout
cv2.destroyAllWindows()
```

### Save Image
```python
cv2.imwrite('output.jpg', img)
cv2.imwrite('output.png', img, [cv2.IMWRITE_PNG_COMPRESSION, 9])  # With compression
```

---

## 2. Window Handling

### Create and Manage Windows
```python
# Create named window
cv2.namedWindow('Window Name', cv2.WINDOW_NORMAL)  # Resizable
cv2.namedWindow('Window Name', cv2.WINDOW_AUTOSIZE)  # Auto-size (default)
cv2.namedWindow('Window Name', cv2.WINDOW_FREERATIO)  # Free aspect ratio

# Resize window
cv2.resizeWindow('Window Name', width, height)

# Move window
cv2.moveWindow('Window Name', x, y)

# Destroy windows
cv2.destroyWindow('Window Name')  # Destroy specific window
cv2.destroyAllWindows()  # Destroy all windows

# Get window properties
cv2.getWindowProperty('Window Name', cv2.WND_PROP_VISIBLE)
cv2.getWindowProperty('Window Name', cv2.WND_PROP_FULLSCREEN)

# Set window to fullscreen
cv2.setWindowProperty('Window Name', cv2.WND_PROP_FULLSCREEN, cv2.WINDOW_FULLSCREEN)
cv2.setWindowProperty('Window Name', cv2.WND_PROP_FULLSCREEN, cv2.WINDOW_NORMAL)
```

### Trackbars
```python
def on_trackbar(val):
    pass

# Create trackbar
cv2.createTrackbar('Trackbar Name', 'Window Name', 0, 255, on_trackbar)

# Get trackbar position
value = cv2.getTrackbarPos('Trackbar Name', 'Window Name')

# Set trackbar position
cv2.setTrackbarPos('Trackbar Name', 'Window Name', value)
```

### Mouse Events
```python
def mouse_callback(event, x, y, flags, param):
    if event == cv2.EVENT_LBUTTONDOWN:
        print(f"Left click at ({x}, {y})")
    elif event == cv2.EVENT_RBUTTONDOWN:
        print(f"Right click at ({x}, {y})")
    elif event == cv2.EVENT_MOUSEMOVE:
        print(f"Mouse moved to ({x}, {y})")

cv2.setMouseCallback('Window Name', mouse_callback)

# Mouse events
# EVENT_LBUTTONDOWN, EVENT_RBUTTONDOWN, EVENT_MBUTTONDOWN
# EVENT_LBUTTONUP, EVENT_RBUTTONUP, EVENT_MBUTTONUP
# EVENT_LBUTTONDBLCLK, EVENT_RBUTTONDBLCLK, EVENT_MBUTTONDBLCLK
# EVENT_MOUSEMOVE, EVENT_MOUSEWHEEL, EVENT_MOUSEHWHEEL
```

---

## 3. Video Capture and Handling

### Capture from Camera
```python
cap = cv2.VideoCapture(0)  # 0 for default camera, 1, 2... for other cameras

# Check if camera opened successfully
if not cap.isOpened():
    print("Cannot open camera")
    exit()

while True:
    ret, frame = cap.read()
    if not ret:
        print("Can't receive frame")
        break
    
    cv2.imshow('Frame', frame)
    
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```

### Capture from Video File
```python
cap = cv2.VideoCapture('video.mp4')

# Check if video opened successfully
if not cap.isOpened():
    print("Error opening video file")
```

### Video Properties (Get)
```python
# Frame properties
fps = cap.get(cv2.CAP_PROP_FPS)
frame_count = cap.get(cv2.CAP_PROP_FRAME_COUNT)
width = cap.get(cv2.CAP_PROP_FRAME_WIDTH)
height = cap.get(cv2.CAP_PROP_FRAME_HEIGHT)
current_pos = cap.get(cv2.CAP_PROP_POS_FRAMES)
milliseconds = cap.get(cv2.CAP_PROP_POS_MSEC)
codec = cap.get(cv2.CAP_PROP_FOURCC)
brightness = cap.get(cv2.CAP_PROP_BRIGHTNESS)
contrast = cap.get(cv2.CAP_PROP_CONTRAST)
saturation = cap.get(cv2.CAP_PROP_SATURATION)
hue = cap.get(cv2.CAP_PROP_HUE)
gain = cap.get(cv2.CAP_PROP_GAIN)
exposure = cap.get(cv2.CAP_PROP_EXPOSURE)
```

### Video Properties (Set)
```python
# Set frame position
cap.set(cv2.CAP_PROP_POS_FRAMES, 100)

# Set resolution
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)

# Set FPS (may not work on all cameras)
cap.set(cv2.CAP_PROP_FPS, 30)

# Set brightness, contrast, etc.
cap.set(cv2.CAP_PROP_BRIGHTNESS, 0.5)
cap.set(cv2.CAP_PROP_CONTRAST, 0.5)
cap.set(cv2.CAP_PROP_SATURATION, 0.5)
cap.set(cv2.CAP_PROP_HUE, 0.5)
```

### Video Writer (Save Video)
```python
# Define codec and create VideoWriter object
fourcc = cv2.VideoWriter_fourcc(*'XVID')  # or 'MJPG', 'MP4V', 'X264'
out = cv2.VideoWriter('output.avi', fourcc, 20.0, (640, 480))

while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        # Write frame
        out.write(frame)
        cv2.imshow('Frame', frame)
        
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break
    else:
        break

cap.release()
out.release()
cv2.destroyAllWindows()

# Common codecs
# cv2.VideoWriter_fourcc(*'XVID') - .avi
# cv2.VideoWriter_fourcc(*'MP4V') - .mp4
# cv2.VideoWriter_fourcc(*'X264') - .mp4 (better compression)
# cv2.VideoWriter_fourcc(*'MJPG') - .avi
```

---

## 4. Image Properties and Manipulation

### Image Properties
```python
height, width, channels = img.shape  # For color images
height, width = img.shape  # For grayscale images
total_pixels = img.size
data_type = img.dtype
image_size_bytes = img.nbytes

# Access pixel value
pixel = img[y, x]  # Returns [B, G, R] for color image
blue = img[y, x, 0]
green = img[y, x, 1]
red = img[y, x, 2]

# Modify pixel value
img[y, x] = [255, 255, 255]  # Set to white
```

### Image ROI (Region of Interest)
```python
# Extract ROI
roi = img[y1:y2, x1:x2]

# Place ROI in another image
img[y1:y2, x1:x2] = roi

# Copy ROI
roi_copy = img[y1:y2, x1:x2].copy()
```

### Splitting and Merging Channels
```python
# Split channels
b, g, r = cv2.split(img)

# Merge channels
img = cv2.merge([b, g, r])

# Set channel to zero
img[:, :, 0] = 0  # Remove blue channel
```

---

## 5. Color Space Conversion
```python
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
lab = cv2.cvtColor(img, cv2.COLOR_BGR2LAB)
yuv = cv2.cvtColor(img, cv2.COLOR_BGR2YUV)
hls = cv2.cvtColor(img, cv2.COLOR_BGR2HLS)

# Back to BGR
bgr = cv2.cvtColor(gray, cv2.COLOR_GRAY2BGR)
bgr = cv2.cvtColor(rgb, cv2.COLOR_RGB2BGR)
```

---

## 6. Image Resizing and Scaling
```python
# Resize to specific dimensions
resized = cv2.resize(img, (width, height))

# Resize by scaling factor
resized = cv2.resize(img, None, fx=0.5, fy=0.5)

# Different interpolation methods
resized = cv2.resize(img, (width, height), interpolation=cv2.INTER_LINEAR)
# INTER_NEAREST - fastest
# INTER_LINEAR - bilinear (default)
# INTER_CUBIC - bicubic (slower, better)
# INTER_AREA - resampling using pixel area (good for shrinking)
# INTER_LANCZOS4 - Lanczos (best quality, slowest)
```

---

## 7. Image Rotation and Transformation

### Rotation
```python
# Get rotation matrix
(h, w) = img.shape[:2]
center = (w // 2, h // 2)
M = cv2.getRotationMatrix2D(center, angle=45, scale=1.0)
rotated = cv2.warpAffine(img, M, (w, h))

# Rotate 90, 180, 270 degrees
rotated_90 = cv2.rotate(img, cv2.ROTATE_90_CLOCKWISE)
rotated_180 = cv2.rotate(img, cv2.ROTATE_180)
rotated_270 = cv2.rotate(img, cv2.ROTATE_90_COUNTERCLOCKWISE)
```

### Flipping
```python
flipped_h = cv2.flip(img, 1)   # Horizontal flip
flipped_v = cv2.flip(img, 0)   # Vertical flip
flipped_both = cv2.flip(img, -1)  # Both axes
```

### Affine Transformation
```python
rows, cols = img.shape[:2]
pts1 = np.float32([[50, 50], [200, 50], [50, 200]])
pts2 = np.float32([[10, 100], [200, 50], [100, 250]])
M = cv2.getAffineTransform(pts1, pts2)
transformed = cv2.warpAffine(img, M, (cols, rows))
```

### Perspective Transformation
```python
pts1 = np.float32([[56, 65], [368, 52], [28, 387], [389, 390]])
pts2 = np.float32([[0, 0], [300, 0], [0, 300], [300, 300]])
M = cv2.getPerspectiveTransform(pts1, pts2)
warped = cv2.warpPerspective(img, M, (300, 300))
```

---

## 8. Cropping
```python
# Crop image
cropped = img[y1:y2, x1:x2]  # [row_start:row_end, col_start:col_end]

# Example: crop center 200x200
h, w = img.shape[:2]
center_crop = img[h//2-100:h//2+100, w//2-100:w//2+100]
```

---

## 9. Drawing Operations

### Line
```python
cv2.line(img, (x1, y1), (x2, y2), (255, 0, 0), thickness=2)
cv2.line(img, (x1, y1), (x2, y2), (255, 0, 0), 2, cv2.LINE_AA)  # Anti-aliased
```

### Rectangle
```python
cv2.rectangle(img, (x1, y1), (x2, y2), (0, 255, 0), thickness=2)
cv2.rectangle(img, (x1, y1), (x2, y2), (0, 255, 0), -1)  # Filled
```

### Circle
```python
cv2.circle(img, (center_x, center_y), radius=50, (0, 0, 255), thickness=2)
cv2.circle(img, (center_x, center_y), radius=50, (0, 0, 255), -1)  # Filled
```

### Ellipse
```python
cv2.ellipse(img, (center_x, center_y), (major_axis, minor_axis), 
            angle=0, startAngle=0, endAngle=360, color=(255, 0, 0), thickness=2)
```

### Polygon
```python
pts = np.array([[10, 5], [20, 30], [70, 20], [50, 10]], np.int32)
pts = pts.reshape((-1, 1, 2))
cv2.polylines(img, [pts], isClosed=True, color=(0, 255, 255), thickness=3)

# Filled polygon
cv2.fillPoly(img, [pts], color=(0, 255, 255))
```

### Text
```python
cv2.putText(img, 'Hello OpenCV', (x, y), cv2.FONT_HERSHEY_SIMPLEX, 
            fontScale=1, color=(255, 255, 255), thickness=2, lineType=cv2.LINE_AA)

# Font options:
# FONT_HERSHEY_SIMPLEX, FONT_HERSHEY_PLAIN, FONT_HERSHEY_DUPLEX
# FONT_HERSHEY_COMPLEX, FONT_HERSHEY_TRIPLEX, FONT_HERSHEY_COMPLEX_SMALL
# FONT_HERSHEY_SCRIPT_SIMPLEX, FONT_HERSHEY_SCRIPT_COMPLEX, FONT_ITALIC

# Get text size
(text_width, text_height), baseline = cv2.getTextSize('Hello', 
                                      cv2.FONT_HERSHEY_SIMPLEX, 1, 2)
```

---

## 10. Image Thresholding
```python
# Simple thresholding
ret, thresh = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)

# Threshold types:
# THRESH_BINARY - Binary thresholding
# THRESH_BINARY_INV - Inverse binary
# THRESH_TRUNC - Truncate
# THRESH_TOZERO - To zero
# THRESH_TOZERO_INV - To zero inverted

# Adaptive thresholding
adaptive_mean = cv2.adaptiveThreshold(gray, 255, cv2.ADAPTIVE_THRESH_MEAN_C,
                                       cv2.THRESH_BINARY, blockSize=11, C=2)
adaptive_gaussian = cv2.adaptiveThreshold(gray, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
                                          cv2.THRESH_BINARY, blockSize=11, C=2)

# Otsu's thresholding
ret, otsu = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)

# Otsu with Gaussian blur
blur = cv2.GaussianBlur(gray, (5, 5), 0)
ret, otsu = cv2.threshold(blur, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
```

---

## 11. Filtering and Smoothing

### Blurring
```python
# Average blur
avg_blur = cv2.blur(img, (5, 5))

# Gaussian Blur
gaussian = cv2.GaussianBlur(img, (5, 5), 0)

# Median Blur (good for salt-and-pepper noise)
median = cv2.medianBlur(img, 5)

# Bilateral Filter (preserves edges)
bilateral = cv2.bilateralFilter(img, d=9, sigmaColor=75, sigmaSpace=75)
```

### Sharpening
```python
# Sharpening kernel
kernel = np.array([[-1, -1, -1],
                   [-1,  9, -1],
                   [-1, -1, -1]])
sharpened = cv2.filter2D(img, -1, kernel)
```

### Custom Kernel Filtering
```python
kernel = np.ones((5, 5), np.float32) / 25
filtered = cv2.filter2D(img, -1, kernel)
```

---

## 12. Edge Detection

### Canny Edge Detection
```python
edges = cv2.Canny(img, threshold1=100, threshold2=200)
edges = cv2.Canny(img, threshold1=100, threshold2=200, apertureSize=3, L2gradient=True)
```

### Sobel
```python
sobelx = cv2.Sobel(gray, cv2.CV_64F, 1, 0, ksize=5)
sobely = cv2.Sobel(gray, cv2.CV_64F, 0, 1, ksize=5)
sobel = cv2.magnitude(sobelx, sobely)
```

### Laplacian
```python
laplacian = cv2.Laplacian(gray, cv2.CV_64F)
```

### Scharr
```python
scharrx = cv2.Scharr(gray, cv2.CV_64F, 1, 0)
scharry = cv2.Scharr(gray, cv2.CV_64F, 0, 1)
```

---

## 13. Morphological Operations
```python
kernel = np.ones((5, 5), np.uint8)

# Erosion
erosion = cv2.erode(img, kernel, iterations=1)

# Dilation
dilation = cv2.dilate(img, kernel, iterations=1)

# Opening (erosion followed by dilation) - removes noise
opening = cv2.morphologyEx(img, cv2.MORPH_OPEN, kernel)

# Closing (dilation followed by erosion) - closes small holes
closing = cv2.morphologyEx(img, cv2.MORPH_CLOSE, kernel)

# Morphological Gradient (difference between dilation and erosion)
gradient = cv2.morphologyEx(img, cv2.MORPH_GRADIENT, kernel)

# Top Hat (difference between input and opening)
tophat = cv2.morphologyEx(img, cv2.MORPH_TOPHAT, kernel)

# Black Hat (difference between closing and input)
blackhat = cv2.morphologyEx(img, cv2.MORPH_BLACKHAT, kernel)

# Structuring elements
kernel_rect = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))
kernel_ellipse = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))
kernel_cross = cv2.getStructuringElement(cv2.MORPH_CROSS, (5, 5))
```

---

## 14. Contour Detection and Analysis

### Find Contours
```python
contours, hierarchy = cv2.findContours(thresh, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

# Retrieval modes:
# RETR_EXTERNAL - only external contours
# RETR_LIST - all contours, no hierarchy
# RETR_CCOMP - all contours, 2-level hierarchy
# RETR_TREE - all contours, full hierarchy

# Approximation methods:
# CHAIN_APPROX_NONE - all boundary points
# CHAIN_APPROX_SIMPLE - compresses horizontal, vertical, diagonal segments
```

### Draw Contours
```python
# Draw all contours
cv2.drawContours(img, contours, -1, (0, 255, 0), 2)

# Draw specific contour
cv2.drawContours(img, contours, 0, (0, 255, 0), 2)

# Draw with hierarchy
cv2.drawContours(img, contours, -1, (0, 255, 0), 2, hierarchy=hierarchy)
```

### Contour Properties
```python
cnt = contours[0]

# Area
area = cv2.contourArea(cnt)

# Perimeter
perimeter = cv2.arcLength(cnt, closed=True)

# Bounding rectangle
x, y, w, h = cv2.boundingRect(cnt)
cv2.rectangle(img, (x, y), (x+w, y+h), (0, 255, 0), 2)

# Rotated bounding rectangle
rect = cv2.minAreaRect(cnt)
box = cv2.boxPoints(rect)
box = np.int0(box)
cv2.drawContours(img, [box], 0, (0, 0, 255), 2)

# Minimum enclosing circle
(x, y), radius = cv2.minEnclosingCircle(cnt)
center = (int(x), int(y))
radius = int(radius)
cv2.circle(img, center, radius, (0, 255, 0), 2)

# Fit ellipse
if len(cnt) >= 5:
    ellipse = cv2.fitEllipse(cnt)
    cv2.ellipse(img, ellipse, (0, 255, 0), 2)

# Convex hull
hull = cv2.convexHull(cnt)
cv2.drawContours(img, [hull], 0, (0, 0, 255), 2)

# Convexity
is_convex = cv2.isContourConvex(cnt)

# Aspect ratio
aspect_ratio = float(w) / h

# Extent (ratio of contour area to bounding rectangle area)
rect_area = w * h
extent = float(area) / rect_area

# Solidity (ratio of contour area to convex hull area)
hull_area = cv2.contourArea(hull)
solidity = float(area) / hull_area

# Moments
M = cv2.moments(cnt)
cx = int(M['m10'] / M['m00'])
cy = int(M['m01'] / M['m00'])

# Approximate contour
epsilon = 0.01 * cv2.arcLength(cnt, True)
approx = cv2.approxPolyDP(cnt, epsilon, True)
```

---

## 15. Image Gradients and Derivatives
```python
# Sobel derivatives
grad_x = cv2.Sobel(img, cv2.CV_64F, 1, 0, ksize=3)
grad_y = cv2.Sobel(img, cv2.CV_64F, 0, 1, ksize=3)

# Absolute gradient
abs_grad_x = cv2.convertScaleAbs(grad_x)
abs_grad_y = cv2.convertScaleAbs(grad_y)

# Combined gradient
grad = cv2.addWeighted(abs_grad_x, 0.5, abs_grad_y, 0.5, 0)
```

---

## 16. Histograms

### Calculate Histogram
```python
# Grayscale histogram
hist = cv2.calcHist([gray], [0], None, [256], [0, 256])

# Color histogram
hist_b = cv2.calcHist([img], [0], None, [256], [0, 256])
hist_g = cv2.calcHist([img], [1], None, [256], [0, 256])
hist_r = cv2.calcHist([img], [2], None, [256], [0, 256])

# With mask
mask = np.zeros(img.shape[:2], np.uint8)
mask[100:300, 100:400] = 255
hist_masked = cv2.calcHist([img], [0], mask, [256], [0, 256])
```

### Histogram Equalization
```python
# Grayscale equalization
equalized = cv2.equalizeHist(gray)

# CLAHE (Contrast Limited Adaptive Histogram Equalization)
clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
cl_img = clahe.apply(gray)

# Color image equalization (convert to YUV)
yuv = cv2.cvtColor(img, cv2.COLOR_BGR2YUV)
yuv[:, :, 0] = cv2.equalizeHist(yuv[:, :, 0])
equalized_color = cv2.cvtColor(yuv, cv2.COLOR_YUV2BGR)
```

### Histogram Comparison
```python
hist1 = cv2.calcHist([img1], [0], None, [256], [0, 256])
hist2 = cv2.calcHist([img2], [0], None, [256], [0, 256])

# Normalize histograms
cv2.normalize(hist1, hist1, 0, 1, cv2.NORM_MINMAX)
cv2.normalize(hist2, hist2, 0, 1, cv2.NORM_MINMAX)

# Compare
correlation = cv2.compareHist(hist1, hist2, cv2.HISTCMP_CORREL)
chi_square = cv2.compareHist(hist1, hist2, cv2.HISTCMP_CHISQR)
intersection = cv2.compareHist(hist1, hist2, cv2.HISTCMP_INTERSECT)
bhattacharyya = cv2.compareHist(hist1, hist2, cv2.HISTCMP_BHATTACHARYYA)
```

---

## 17. Bitwise Operations
```python
# AND
result = cv2.bitwise_and(img1, img2)
result = cv2.bitwise_and(img1, img2, mask=mask)

# OR
result = cv2.bitwise_or(img1, img2)

# XOR
result = cv2.bitwise_xor(img1, img2)

# NOT
result = cv2.bitwise_not(img)
```

---

## 18. Masking and ROI
```python
# Create mask
mask = np.zeros(img.shape[:2], dtype=np.uint8)
cv2.circle(mask, (center_x, center_y), radius, 255, -1)

# Apply mask
masked = cv2.bitwise_and(img, img, mask=mask)

# Inverse mask
inverse_mask = cv2.bitwise_not(mask)
masked_inv = cv2.bitwise_and(img, img, mask=inverse_mask)

# Color range masking (HSV)
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
lower_blue = np.array([110, 50, 50])
upper_blue = np.array([130, 255, 255])
mask = cv2.inRange(hsv, lower_blue, upper_blue)
result = cv2.bitwise_and(img, img, mask=mask)
```

---

## 19. Image Arithmetic

### Addition and Subtraction
```python
# Addition
result = cv2.add(img1, img2)
result = img1 + img2  # NumPy addition (may overflow)

# Weighted addition (blending)
result = cv2.addWeighted(img1, alpha=0.7, img2, beta=0.3, gamma=0)

# Subtraction
result = cv2.subtract(img1, img2)
result = img1 - img2  # NumPy subtraction

# Absolute difference
result = cv2.absdiff(img1, img2)
```

### Multiplication and Division
```python
# Multiply
result = cv2.multiply(img1, img2)

# Divide
result = cv2.divide(img1, img2)
```

### Image Scaling
```python
# Scale pixel values
scaled = cv2.convertScaleAbs(img, alpha=1.5, beta=0)  # alpha=gain, beta=bias
```

---

## 20. Template Matching
```python
template = cv2.imread('template.jpg', 0)
result = cv2.matchTemplate(gray, template, cv2.TM_CCOEFF_NORMED)

# Methods:
# TM_CCOEFF, TM_CCOEFF_NORMED
# TM_CCORR, TM_CCORR_NORMED
# TM_SQDIFF, TM_SQDIFF_NORMED (lower is better)

# Find best match
min_val, max_val, min_loc, max_loc = cv2.minMaxLoc(result)

# For TM_SQDIFF and TM_SQDIFF_NORMED, use min_loc
# For others, use max_loc
top_left = max_loc
h, w = template.shape
bottom_right = (top_left[0] + w, top_left[1] + h)
cv2.rectangle(img, top_left, bottom_right, (0, 255, 0), 2)

# Multiple matches (using threshold)
threshold = 0.8
loc = np.where(result >= threshold)
for pt in zip(*loc[::-1]):
    cv2.rectangle(img, pt, (pt[0] + w, pt[1] + h), (0, 255, 0), 2)
```

---

## 21. Feature Detection

### Harris Corner Detection
```python
gray = np.float32(gray)
corners = cv2.cornerHarris(gray, blockSize=2, ksize=3, k=0.04)
corners = cv2.dilate(corners, None)
img[corners > 0.01 * corners.max()] = [0, 0, 255]
```

### Shi-Tomasi Corner Detection
```python
corners = cv2.goodFeaturesToTrack(gray, maxCorners=100, qualityLevel=0.01, 
                                   minDistance=10)
corners = np.int0(corners)
for corner in corners:
    x, y = corner.ravel()
    cv2.circle(img, (x, y), 3, (0, 0, 255), -1)
```

### SIFT (Scale-Invariant Feature Transform)
```python
sift = cv2.SIFT_create()
keypoints, descriptors = sift.detectAndCompute(gray, None)
img_with_keypoints = cv2.drawKeypoints(img, keypoints, None, 
                                        flags=cv2.DRAW_MATCHES_FLAGS_DRAW_RICH_KEYPOINTS)
```

### ORB (Oriented FAST and Rotated BRIEF)
```python
orb = cv2.ORB_create(nfeatures=500)
keypoints, descriptors = orb.detectAndCompute(gray, None)
img_with_keypoints = cv2.drawKeypoints(img, keypoints, None, color=(0, 255, 0))
```

### Feature Matching
```python
# Using BFMatcher (Brute Force)
bf = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)
matches = bf.match(des1, des2)
matches = sorted(matches, key=lambda x: x.distance)

# Draw matches
img_matches = cv2.drawMatches(img1, kp1, img2, kp2, matches[:10], None,
                               flags=cv2.DrawMatchesFlags_NOT_DRAW_SINGLE_POINTS)

# Using FLANN (Fast Library for Approximate Nearest Neighbors)
FLANN_INDEX_KDTREE = 1
index_params = dict(algorithm=FLANN_INDEX_KDTREE, trees=5)
search_params = dict(checks=50)
flann = cv2.FlannBasedMatcher(index_params, search_params)
matches = flann.knnMatch(des1, des2, k=2)

# Apply Lowe's ratio test
good_matches = []
for m, n in matches:
    if m.distance < 0.7 * n.distance:
        good_matches.append(m)
```

---

## 22. Image Pyramids

### Gaussian Pyramid
```python
# Down-sampling (reduce size)
lower_reso = cv2.pyrDown(img)
lower_reso2 = cv2.pyrDown(lower_reso)

# Up-sampling (increase size)
higher_reso = cv2.pyrUp(img)
```

### Laplacian Pyramid
```python
# Create Laplacian pyramid
gaussian = img.copy()
laplacian_pyramid = []

for i in range(6):
    gaussian = cv2.pyrDown(gaussian)
    laplacian = cv2.subtract(img, cv2.pyrUp(gaussian))
    laplacian_pyramid.append(laplacian)
    img = gaussian
```

---

## 23. Background Subtraction

### MOG2 Background Subtractor
```python
backSub = cv2.createBackgroundSubtractorMOG2(history=500, varThreshold=16, 
                                              detectShadows=True)

while True:
    ret, frame = cap.read()
    if not ret:
        break
    
    fgMask = backSub.apply(frame)
    cv2.imshow('Frame', frame)
    cv2.imshow('FG Mask', fgMask)
    
    if cv2.waitKey(30) & 0xFF == ord('q'):
        break
```

### KNN Background Subtractor
```python
backSub = cv2.createBackgroundSubtractorKNN(history=500, dist2Threshold=400, 
                                             detectShadows=True)
fgMask = backSub.apply(frame)
```

---

## 24. Hough Transforms

### Hough Line Transform
```python
edges = cv2.Canny(gray, 50, 150)
lines = cv2.HoughLines(edges, rho=1, theta=np.pi/180, threshold=200)

if lines is not None:
    for line in lines:
        rho, theta = line[0]
        a = np.cos(theta)
        b = np.sin(theta)
        x0 = a * rho
        y0 = b * rho
        x1 = int(x0 + 1000 * (-b))
        y1 = int(y0 + 1000 * (a))
        x2 = int(x0 - 1000 * (-b))
        y2 = int(y0 - 1000 * (a))
        cv2.line(img, (x1, y1), (x2, y2), (0, 0, 255), 2)
```

### Probabilistic Hough Line Transform
```python
lines = cv2.HoughLinesP(edges, rho=1, theta=np.pi/180, threshold=50,
                        minLineLength=50, maxLineGap=10)

if lines is not None:
    for line in lines:
        x1, y1, x2, y2 = line[0]
        cv2.line(img, (x1, y1), (x2, y2), (0, 255, 0), 2)
```

### Hough Circle Transform
```python
gray = cv2.medianBlur(gray, 5)
circles = cv2.HoughCircles(gray, cv2.HOUGH_GRADIENT, dp=1, minDist=20,
                           param1=50, param2=30, minRadius=0, maxRadius=0)

if circles is not None:
    circles = np.uint16(np.around(circles))
    for i in circles[0, :]:
        cv2.circle(img, (i[0], i[1]), i[2], (0, 255, 0), 2)  # Draw outer circle
        cv2.circle(img, (i[0], i[1]), 2, (0, 0, 255), 3)     # Draw center
```

---

## 25. Image Segmentation

### Watershed Algorithm
```python
# Convert to grayscale
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Threshold
ret, thresh = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)

# Noise removal
kernel = np.ones((3, 3), np.uint8)
opening = cv2.morphologyEx(thresh, cv2.MORPH_OPEN, kernel, iterations=2)

# Sure background area
sure_bg = cv2.dilate(opening, kernel, iterations=3)

# Finding sure foreground area
dist_transform = cv2.distanceTransform(opening, cv2.DIST_L2, 5)
ret, sure_fg = cv2.threshold(dist_transform, 0.7 * dist_transform.max(), 255, 0)

# Finding unknown region
sure_fg = np.uint8(sure_fg)
unknown = cv2.subtract(sure_bg, sure_fg)

# Marker labelling
ret, markers = cv2.connectedComponents(sure_fg)
markers = markers + 1
markers[unknown == 255] = 0

# Apply watershed
markers = cv2.watershed(img, markers)
img[markers == -1] = [255, 0, 0]
```

### GrabCut
```python
mask = np.zeros(img.shape[:2], np.uint8)
bgdModel = np.zeros((1, 65), np.float64)
fgdModel = np.zeros((1, 65), np.float64)

# Define rectangle containing foreground
rect = (50, 50, 450, 290)

# Apply GrabCut
cv2.grabCut(img, mask, rect, bgdModel, fgdModel, 5, cv2.GC_INIT_WITH_RECT)

# Modify mask
mask2 = np.where((mask == 2) | (mask == 0), 0, 1).astype('uint8')
result = img * mask2[:, :, np.newaxis]
```

### K-Means Clustering
```python
# Reshape image to 2D array of pixels
pixel_values = img.reshape((-1, 3))
pixel_values = np.float32(pixel_values)

# Define criteria and apply kmeans
criteria = (cv2.TERM_CRITERIA_EPS + cv2.TERM_CRITERIA_MAX_ITER, 100, 0.2)
k = 3
_, labels, centers = cv2.kmeans(pixel_values, k, None, criteria, 10, 
                                 cv2.KMEANS_RANDOM_CENTERS)

# Convert back to 8 bit values
centers = np.uint8(centers)
segmented = centers[labels.flatten()]
segmented_image = segmented.reshape(img.shape)
```

---

## 26. Object Tracking

### MeanShift
```python
# Setup initial location of window
x, y, w, h = 300, 200, 100, 50
track_window = (x, y, w, h)

# Setup ROI for tracking
roi = frame[y:y+h, x:x+w]
hsv_roi = cv2.cvtColor(roi, cv2.COLOR_BGR2HSV)
mask = cv2.inRange(hsv_roi, np.array((0., 60., 32.)), np.array((180., 255., 255.)))
roi_hist = cv2.calcHist([hsv_roi], [0], mask, [180], [0, 180])
cv2.normalize(roi_hist, roi_hist, 0, 255, cv2.NORM_MINMAX)

# Setup termination criteria
term_crit = (cv2.TERM_CRITERIA_EPS | cv2.TERM_CRITERIA_COUNT, 10, 1)

while True:
    ret, frame = cap.read()
    if ret:
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
        dst = cv2.calcBackProject([hsv], [0], roi_hist, [0, 180], 1)
        
        # Apply meanshift
        ret, track_window = cv2.meanShift(dst, track_window, term_crit)
        
        # Draw on image
        x, y, w, h = track_window
        cv2.rectangle(frame, (x, y), (x+w, y+h), 255, 2)
        cv2.imshow('Tracking', frame)
        
        if cv2.waitKey(60) & 0xFF == ord('q'):
            break
```

### CamShift
```python
# Similar setup as MeanShift, but use:
ret, track_window = cv2.CamShift(dst, track_window, term_crit)

# Draw rotated rectangle
pts = cv2.boxPoints(ret)
pts = np.int0(pts)
cv2.polylines(frame, [pts], True, 255, 2)
```

### Optical Flow (Lucas-Kanade)
```python
# Parameters for ShiTomasi corner detection
feature_params = dict(maxCorners=100, qualityLevel=0.3, minDistance=7, blockSize=7)

# Parameters for Lucas-Kanade optical flow
lk_params = dict(winSize=(15, 15), maxLevel=2,
                 criteria=(cv2.TERM_CRITERIA_EPS | cv2.TERM_CRITERIA_COUNT, 10, 0.03))

old_gray = cv2.cvtColor(old_frame, cv2.COLOR_BGR2GRAY)
p0 = cv2.goodFeaturesToTrack(old_gray, mask=None, **feature_params)

while True:
    ret, frame = cap.read()
    if not ret:
        break
    
    frame_gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    
    # Calculate optical flow
    p1, st, err = cv2.calcOpticalFlowPyrLK(old_gray, frame_gray, p0, None, **lk_params)
    
    # Select good points
    if p1 is not None:
        good_new = p1[st == 1]
        good_old = p0[st == 1]
    
    # Draw tracks
    for i, (new, old) in enumerate(zip(good_new, good_old)):
        a, b = new.ravel()
        c, d = old.ravel()
        a, b, c, d = int(a), int(b), int(c), int(d)
        cv2.line(frame, (a, b), (c, d), (0, 255, 0), 2)
        cv2.circle(frame, (a, b), 5, (0, 0, 255), -1)
    
    cv2.imshow('Optical Flow', frame)
    
    # Update previous frame and points
    old_gray = frame_gray.copy()
    p0 = good_new.reshape(-1, 1, 2)
    
    if cv2.waitKey(30) & 0xFF == ord('q'):
        break
```

### Dense Optical Flow (Farneback)
```python
old_gray = cv2.cvtColor(old_frame, cv2.COLOR_BGR2GRAY)

while True:
    ret, frame = cap.read()
    if not ret:
        break
    
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    
    # Calculate dense optical flow
    flow = cv2.calcOpticalFlowFarneback(old_gray, gray, None, 0.5, 3, 15, 3, 5, 1.2, 0)
    
    # Compute magnitude and angle
    magnitude, angle = cv2.cartToPolar(flow[..., 0], flow[..., 1])
    
    # Visualization (HSV)
    hsv = np.zeros_like(frame)
    hsv[..., 1] = 255
    hsv[..., 0] = angle * 180 / np.pi / 2
    hsv[..., 2] = cv2.normalize(magnitude, None, 0, 255, cv2.NORM_MINMAX)
    bgr = cv2.cvtColor(hsv, cv2.COLOR_HSV2BGR)
    
    cv2.imshow('Dense Optical Flow', bgr)
    
    old_gray = gray
    
    if cv2.waitKey(30) & 0xFF == ord('q'):
        break
```

---

## 27. Face Detection (Haar Cascades)

### Load and Use Cascade Classifier
```python
# Load pre-trained classifiers
face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')
eye_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_eye.xml')

# Detect faces
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
faces = face_cascade.detectMultiScale(gray, scaleFactor=1.1, minNeighbors=5, 
                                      minSize=(30, 30))

# Draw rectangles around faces
for (x, y, w, h) in faces:
    cv2.rectangle(img, (x, y), (x+w, y+h), (255, 0, 0), 2)
    roi_gray = gray[y:y+h, x:x+w]
    roi_color = img[y:y+h, x:x+w]
    
    # Detect eyes within face region
    eyes = eye_cascade.detectMultiScale(roi_gray)
    for (ex, ey, ew, eh) in eyes:
        cv2.rectangle(roi_color, (ex, ey), (ex+ew, ey+eh), (0, 255, 0), 2)
```

### Other Haar Cascades
```python
# Available cascades in cv2.data.haarcascades:
# haarcascade_frontalface_default.xml
# haarcascade_frontalface_alt.xml
# haarcascade_frontalface_alt2.xml
# haarcascade_frontalface_alt_tree.xml
# haarcascade_eye.xml
# haarcascade_eye_tree_eyeglasses.xml
# haarcascade_smile.xml
# haarcascade_fullbody.xml
# haarcascade_upperbody.xml
```

---

## 28. Image Inpainting
```python
# Create mask (damaged areas in white)
mask = cv2.imread('mask.png', 0)

# Inpaint using Telea method
inpainted_telea = cv2.inpaint(img, mask, inpaintRadius=3, flags=cv2.INPAINT_TELEA)

# Inpaint using NS (Navier-Stokes) method
inpainted_ns = cv2.inpaint(img, mask, inpaintRadius=3, flags=cv2.INPAINT_NS)
```

---

## 29. Image Denoising
```python
# Non-Local Means Denoising (grayscale)
denoised = cv2.fastNlMeansDenoising(gray, None, h=10, templateWindowSize=7, 
                                     searchWindowSize=21)

# Non-Local Means Denoising (color)
denoised = cv2.fastNlMeansDenoisingColored(img, None, h=10, hColor=10,
                                            templateWindowSize=7, searchWindowSize=21)

# Multi-frame denoising
denoised = cv2.fastNlMeansDenoisingMulti(img_list, imgToDenoiseIndex=2, 
                                          temporalWindowSize=5)
```

---

## 30. Image Quality Metrics

### PSNR (Peak Signal-to-Noise Ratio)
```python
psnr_value = cv2.PSNR(img1, img2)
print(f'PSNR: {psnr_value} dB')
```

### SSIM (Structural Similarity Index) - requires scikit-image
```python
from skimage.metrics import structural_similarity as ssim
ssim_value = ssim(img1, img2, multichannel=True, channel_axis=2)
```

---

## 31. QR Code and Barcode Detection

### QR Code Detection
```python
# Create QR Code detector
qr_detector = cv2.QRCodeDetector()

# Detect and decode
data, bbox, straight_qrcode = qr_detector.detectAndDecode(img)

if bbox is not None:
    print(f"QR Code data: {data}")
    # Draw bounding box
    n_lines = len(bbox)
    for i in range(n_lines):
        point1 = tuple(bbox[i][0].astype(int))
        point2 = tuple(bbox[(i+1) % n_lines][0].astype(int))
        cv2.line(img, point1, point2, (255, 0, 0), 3)
```

---

## 32. Image Normalization
```python
# Min-Max normalization
normalized = cv2.normalize(img, None, alpha=0, beta=255, norm_type=cv2.NORM_MINMAX)

# L2 normalization
normalized = cv2.normalize(img, None, alpha=1, beta=0, norm_type=cv2.NORM_L2)

# L1 normalization
normalized = cv2.normalize(img, None, alpha=1, beta=0, norm_type=cv2.NORM_L1)
```

---

## 33. Geometric Transformations

### Translation
```python
tx, ty = 100, 50  # Translation values
M = np.float32([[1, 0, tx], [0, 1, ty]])
translated = cv2.warpAffine(img, M, (img.shape[1], img.shape[0]))
```

### Shearing
```python
M = np.float32([[1, 0.5, 0], [0, 1, 0]])
sheared = cv2.warpAffine(img, M, (int(img.shape[1] * 1.5), img.shape[0]))
```

---

## 34. Color Detection and Tracking
```python
# Convert to HSV
hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

# Define color range (example: blue)
lower_blue = np.array([110, 50, 50])
upper_blue = np.array([130, 255, 255])

# Create mask
mask = cv2.inRange(hsv, lower_blue, upper_blue)

# Bitwise-AND mask and original image
result = cv2.bitwise_and(frame, frame, mask=mask)

# Find contours of the detected color
contours, _ = cv2.findContours(mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

for cnt in contours:
    area = cv2.contourArea(cnt)
    if area > 500:
        x, y, w, h = cv2.boundingRect(cnt)
        cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)
```

---

## 35. Image Stitching (Panorama)
```python
# Read images
img1 = cv2.imread('image1.jpg')
img2 = cv2.imread('image2.jpg')

# Create stitcher
stitcher = cv2.Stitcher_create()

# Stitch images
status, panorama = stitcher.stitch([img1, img2])

if status == cv2.Stitcher_OK:
    cv2.imshow('Panorama', panorama)
else:
    print("Stitching failed")
```

---

## 36. Performance Optimization

### Check OpenCV Build Information
```python
print(cv2.getBuildInformation())
```

### Get/Set Number of Threads
```python
# Get number of threads
num_threads = cv2.getNumThreads()

# Set number of threads
cv2.setNumThreads(4)
```

### Use UMat (OpenCL acceleration)
```python
# Convert to UMat for GPU processing
umat_img = cv2.UMat(img)
gray_umat = cv2.cvtColor(umat_img, cv2.COLOR_BGR2GRAY)
result = gray_umat.get()  # Convert back to numpy array
```

### Measure Performance
```python
e1 = cv2.getTickCount()
# Your code here
e2 = cv2.getTickCount()
time = (e2 - e1) / cv2.getTickFrequency()
print(f"Time taken: {time} seconds")
```

---

## 37. Image Compression
```python
# JPEG compression
encode_param = [int(cv2.IMWRITE_JPEG_QUALITY), 90]
result, encimg = cv2.imencode('.jpg', img, encode_param)

# PNG compression
encode_param = [int(cv2.IMWRITE_PNG_COMPRESSION), 9]
result, encimg = cv2.imencode('.png', img, encode_param)

# Decode
decimg = cv2.imdecode(encimg, 1)
```

---

## 38. Common Color Formats (BGR)
```python
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
RED = (0, 0, 255)
GREEN = (0, 255, 0)
BLUE = (255, 0, 0)
YELLOW = (0, 255, 255)
CYAN = (255, 255, 0)
MAGENTA = (255, 0, 255)
ORANGE = (0, 165, 255)
PURPLE = (128, 0, 128)
PINK = (203, 192, 255)
BROWN = (42, 42, 165)
GRAY = (128, 128, 128)
```

---

## 39. Useful Tips and Best Practices

### General Tips
- OpenCV uses **BGR** color format by default (not RGB)
- Image coordinates: **(0,0)** is top-left corner
- Array indexing: **img[y, x]** or **img[row, column]**
- Always release video capture: **cap.release()**
- Always destroy windows: **cv2.destroyAllWindows()**
- Use **cv2.waitKey(1)** in loops for real-time processing
- For videos, check if **ret** is True before processing frame

### Performance Tips
- Convert to grayscale when color is not needed
- Resize images for faster processing
- Use appropriate data types (uint8 vs float32)
- Process ROI instead of full image when possible
- Use **cv2.UMat** for GPU acceleration
- Avoid unnecessary copies with **.copy()**

### Common Issues
- **Image not loading**: Check file path and extension
- **Window not responding**: Add **cv2.waitKey()** after **imshow()**
- **Video not playing**: Check codec compatibility
- **Colors look wrong**: Remember BGR vs RGB
- **Memory issues**: Release resources and destroy windows

---

## 40. Keyboard Shortcuts and Event Handling

### Common Key Codes
```python
ESC = 27
ENTER = 13
SPACE = 32
BACKSPACE = 8
TAB = 9

# Usage
key = cv2.waitKey(0) & 0xFF
if key == 27:  # ESC
    break
elif key == ord('s'):  # 's' key
    cv2.imwrite('saved.jpg', img)
elif key == ord('q'):  # 'q' key
    cv2.destroyAllWindows()
```

---

## Quick Reference: Common Operations

```python
# Read image
img = cv2.imread('image.jpg')

# Display image
cv2.imshow('Image', img)
cv2.waitKey(0)

# Save image
cv2.imwrite('output.jpg', img)

# Convert to grayscale
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Resize image
resized = cv2.resize(img, (width, height))

# Blur image
blurred = cv2.GaussianBlur(img, (5, 5), 0)

# Edge detection
edges = cv2.Canny(gray, 100, 200)

# Draw rectangle
cv2.rectangle(img, (x1, y1), (x2, y2), (0, 255, 0), 2)

# Video capture
cap = cv2.VideoCapture(0)
ret, frame = cap.read()
cap.release()

# Threshold
ret, thresh = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)

# Find contours
contours, hierarchy = cv2.findContours(thresh, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
```

---

**Happy Coding with OpenCV! 📸**