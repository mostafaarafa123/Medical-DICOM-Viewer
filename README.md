
# DICOM Viewer Project 
 Hi,Names:mostafa tayel :Aly saad   :Omar tawfik  :Abdallah hisham: Mohamed abdelfattah
    ID:202402876        :202403246  :202401118    :202400592       : 202400144
   and this was our first big project using PyDICOM and WE really struggled with this project.

## Overview
This project is a DICOM viewer built with Streamlit. It supports uploading multiple DICOM files, viewing images with brightness/contrast control, applying filters, zooming, and creating GIFs.

---

## Detailed Problems we faced and Fixes by Code Section

### 1. Import Statements
```python
import streamlit as st
import pydicom
import numpy as np
import cv2
from PIL import Image
import io
```
- **Problems faced:**  
  - Initially missed installing `opencv-python`, which caused `ModuleNotFoundError`.  
  - Forgot to import `io`, which caused errors when handling uploaded files.  
- **Fix:** Installed required libraries with `pip install opencv-python pydicom streamlit pillow` and added missing imports.

---

### 2. `st.set_page_config()` Call
```python
st.set_page_config(page_title="PyDICOM Web", layout="wide")
```
- **Issue:**  
  - Called `set_page_config()` after some Streamlit display commands caused a runtime error.  
- **Fix:** Made sure this is the very first Streamlit command to set page config.

---

### 3. File Uploader and Reading Files
```python
uploaded_files = st.file_uploader("Upload multiple DICOM (.dcm) files", type="dcm", accept_multiple_files=True)
```
- **Issue:**  
  - Using `pydicom.dcmread()` directly on `uploaded_file` caused errors because it expects a filename or file-like object.  
  - Passing Streamlit's `UploadedFile` directly caused `TypeError`.  
- **Fix:** Wrapped file bytes in `io.BytesIO()` before calling `dcmread()`, like this:  
```python
file_like = io.BytesIO(uploaded_file.read())
ds = pydicom.dcmread(file_like)
```

---

### 4. Reading and Sorting DICOM Datasets
```python
dcm_datasets.sort(key=lambda ds: ds.InstanceNumber)
```
- **Problem:**  
  - Some DICOM files were missing `InstanceNumber`, causing an `AttributeError`.  
- **Fix:** Added nested try-except blocks:  
```python
try:
    dcm_datasets.sort(key=lambda ds: ds.InstanceNumber)
except:
    try:
        dcm_datasets.sort(key=lambda ds: ds.SliceLocation)
    except:
        st.warning("Warning: Using default file order")
```
This avoids crashes when metadata is incomplete.

---

### 5. Brightness and Contrast Adjustment in `process_image()`
```python
img_normalized = (img - img.min()) / (img.max() - img.min())
img_processed = np.clip(img_normalized * contrast + (brightness / 100), 0, 1)
```
- **Problems:**  
  - Without normalization, images were either too dark or washed out after brightness/contrast change.  
  - Applying brightness/contrast without clipping caused values to go outside the valid range (0-1), leading to display issues.  
- **Fix:** Normalized the image first, then clipped pixel values to keep them valid.

---

### 6. Filters (e.g., CLAHE, Edge, Threshold)
```python
if filter_type == 'CLAHE':
    img_processed = apply_clahe(img_processed)
```
- **Issues:**  
  - CLAHE needs 8-bit images; passing floats caused unexpected results.  
  - Edge detection (Canny) requires 8-bit images, so input had to be converted correctly.  
- **Fix:**  
  - Converted images to 8-bit (`np.uint8`) before applying filters.  
  - Used `img_8bit = np.uint8(img_processed * 255)` where needed.

---

### 7. Zoom Implementation
```python
if zoom != 1.0:
    # cropping logic
    img_processed = img_processed[y0:y1, x0:x1]
    img_processed = cv2.resize(img_processed, (w, h))
```
- **Issue:**  
  - When zoom was high, crop indices sometimes went out of bounds causing errors or wrong crops.  
- **Fix:**  
  - Added boundary checks with `max(0, …)` and `min(w, …)` to keep cropping inside image dimensions.

---

### 8. Creating GIF Animation
```python
images[0].save(
    gif_bytes,
    format="GIF",
    save_all=True,
    append_images=images[1:],
    duration=100,
    loop=0
)
```
- **Problem:**  
  - If image sizes varied due to zoom/filter, saving GIF raised errors or distorted animation.  
- **Fix:**  
  - Resized all images to the same size before adding to GIF.

---

### 9. Metadata Display and Safety
```python
def safe_str(value):
    try:
        return str(value)
    except:
        return "Unknown"
```
- **Issue:**  
  - Some metadata tags were missing or had unusual data types, causing display errors or crashes.  
- **Fix:** Wrapped metadata conversion in try-except to return `"Unknown"` for problematic values.

---

### 10. General Debugging Notes
- Many runtime errors happened when DICOM files were corrupted or incomplete — had to add error handling around file reading and image processing.
- Took a lot of trial and error to get pixel intensity normalization right so that filters and brightness/contrast worked well together.
- Zoom cropping and resizing was tricky to get right without breaking image display or causing empty images.
- Handling multiple slices smoothly in GIF creation took multiple debugging sessions to fix size mismatches.

---

## in the end

Building this DICOM viewer was a real struggle. I faced many errors from the file handling layer up to image processing. But step by step, I fixed them by carefully reading error messages, researching DICOM file structure, and learning about image normalization and filters.

Now the app works reliably, but this experience taught me how fragile handling medical images can be, especially for a beginner like me. I hope this README helps anyone else who gets stuck on similar problems!##Please send this file to the next batches because I wrote down every silly mistake I made. I really hope nobody else falls into these silly traps again! Consider it a friendly warning and a lesson on what not to do.
*WE really struggled with debugging but finally made it work.*
