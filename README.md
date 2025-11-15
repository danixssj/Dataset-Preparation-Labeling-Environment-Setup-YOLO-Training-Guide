# Dataset Preparation, Labeling, Environment Setup & YOLO Training Guide
This guide walks through the **complete workflow** of collecting your images, labeling them using Label Studio, preparing a YOLO-compatible dataset, setting up your environment, validating everything, training a YOLO model, and finally running real-time detection.


## 🧰 Technical Stack

### 🧪 Core Machine Learning & Vision Frameworks  
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=ffdd54)
![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white)
![Ultralytics YOLO](https://img.shields.io/badge/YOLO-FF6F00?style=for-the-badge&logo=yolo&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-27338E?style=for-the-badge&logo=opencv&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white)
![Label Studio](https://img.shields.io/badge/Label%20Studio-FF6B6B?style=for-the-badge&logo=labelstudio&logoColor=white)

---

### 🖥️ Compute, Drivers & GPU Acceleration  
![CUDA](https://img.shields.io/badge/CUDA-76B900?style=for-the-badge&logo=nvidia&logoColor=white)
![cuDNN](https://img.shields.io/badge/cuDNN-005A9C?style=for-the-badge)
![NVIDIA GPU](https://img.shields.io/badge/NVIDIA_GPU-76B900?style=for-the-badge&logo=nvidia&logoColor=white)
![RTX Series](https://img.shields.io/badge/RTX%20Series-000000?style=for-the-badge&logo=nvidia&logoColor=76B900)

---

### 🧩 Developer Tools & IDEs  
![Anaconda](https://img.shields.io/badge/Anaconda-44A833?style=for-the-badge&logo=anaconda&logoColor=white)
![Conda Env](https://img.shields.io/badge/Conda_Environment-3D4C6A?style=for-the-badge)
![VS Code](https://img.shields.io/badge/VS%20Code-007ACC?style=for-the-badge&logo=visual-studio-code&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

---

### 📂 Dataset & Annotation Tools  
![Label Studio](https://img.shields.io/badge/Label%20Studio-FF6B6B?style=for-the-badge&logo=labelstudio&logoColor=white)
![COCO Format](https://img.shields.io/badge/COCO_Format-FF9A00?style=for-the-badge)
![YOLO Format](https://img.shields.io/badge/YOLO_Format-FF6F00?style=for-the-badge)

---

### 🧵 Operating Systems & Deployment Environments  
![Windows](https://img.shields.io/badge/Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)
---

### 📚 Documentation & Workflow Tool  
![Markdown](https://img.shields.io/badge/Markdown-000000?style=for-the-badge&logo=markdown&logoColor=white)

---

All steps are written **generically**, you may replace folder names with whatever you prefer.


## 1. Collect Images
1. Click pictures of the objects you want the model to learn.
2. Move all images into a dedicated folder, e.g.: my_yolo_project/
images_raw/
3. Select all images → rename them with a common prefix  
   (e.g., `image_1.jpg`, `image_2.jpg`, …).  
   This helps Label Studio import them properly.

## 2. Install & Launch Label Studio
You can use any Conda environment name; here we use `image_env` as an example.

```bash
# Create environment
conda create -n image_env python=3.11

# Activate environment
conda activate image_env

# Install Label Studio
pip install label-studio

# Start Label Studio
label-studio start
```

## 3. Configure Labeling (Object Detection)

- Open the link shown in the terminal.  
- Sign up (local account).  
- Create **New Project**, give it a name.  
- Click **Import** and upload images in batches (30–40 at a time recommended).  
- For very large datasets (1000+ images), consider using cloud storage.  

### Configure Labeling

- Go to **Labeling Setup**.  
- Select **Object Detection with Bounding Box**.  
- Delete the default classes.  
- Add your custom class names (press **Enter** after each).  
- Click **Add** → **Save**.  


## 4. Label the Images

- Open the first image.  
- Choose a class at the bottom.  
- Click-and-drag to create a bounding box.  
- Adjust the box if needed.  
- Use class hotkeys (numbers) for faster work.  
- Press **Submit** → continue to next image.  

After all images are labeled:

- Go back to the project page  
- Click **Export** → choose **YOLO (with images)**  
- Download the generated ZIP file  

Move the ZIP into your project: my_yolo_project/data.zip


Unzip it inside the same folder.


## 5. Create the YOLO Dataset Folder Structure

Inside your project directory, create:

dataset/

│── train/ \
│ ├── images/ \
│ └── labels/ \
│── valid/ \
│ ├── images/ \
│ └── labels/ 


Move **70%** of the images + labels into `train/` and **30%** into `valid/`.


## 6. Create a `data.yaml` File

Inside the `dataset/` folder, create:


Add:

```yaml
train: dataset/train/images
val: dataset/valid/images

nc: 1 # 1 for one object class and 2 for two object class
names: ["object"] # List out the object you name or use in labeling here
```
You may modify: 
* nc: number of classes
* names: list of class names

## 7. Validate YAML File

Create: `test_yaml.py`
``` python
import yaml

with open("dataset/data.yaml", "r") as f: 
    data = yaml.safe_load(f)
    print(data)
```
Run on terminal : 
```bash
python test_yaml.py
```
If the YAML contents print correctly, you're good.


## 8. Validate Image–Label Matching

Create: `check_dataset.py`

``` python
import os

train_img_dir = "dataset/train/images"
train_lbl_dir = "dataset/train/labels"
val_img_dir = "dataset/valid/images"
val_lbl_dir = "dataset/valid/labels"

def count_and_check(img_dir, lbl_dir):
    imgs = sorted([f for f in os.listdir(img_dir) if f.lower().endswith(('.jpg','.jpeg','.png'))])
    lbls = sorted([f for f in os.listdir(lbl_dir) if f.lower().endswith('.txt')])

    print(f"{img_dir}: {len(imgs)} images")
    print(f"{lbl_dir}: {len(lbls)} labels")

    img_bases = {os.path.splitext(f)[0] for f in imgs}
    lbl_bases = {os.path.splitext(f)[0] for f in lbls}

    only_imgs = sorted(img_bases - lbl_bases)[:10]
    only_lbls = sorted(lbl_bases - img_bases)[:10]

    if only_imgs:
        print("Images without labels (examples):", only_imgs)
    if only_lbls:
        print("Labels without images (examples):", only_lbls)
    if not only_imgs and not only_lbls:
        print("All images and labels match.\n")

print()
count_and_check(train_img_dir, train_lbl_dir)
count_and_check(val_img_dir, val_lbl_dir)
```
Run on terminal : `python check_dataset.py`

> If you see classes.txt under “labels without images,” delete it.

## 9. Install YOLO & Dependencies

```bash
conda activate droneenv
```
## 10 Install PyTorch (CUDA 12.1):

```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

## 11 Install YOLO and dependencies:
```bash
pip install ultralytics
pip install opencv-python numpy
```

## 12. Verify GPU
``` python 
python -c "import torch; print(torch.cuda.is_available())"
```

> Expected output: True (You will also see your GPU name when checking device 0.)

## 13. Train the Model

Run:
``` bash
yolo detect train data="dataset/data.yaml" model=yolov8n.pt epochs=50 imgsz=640 batch=8 device=0
```

After training, YOLO saves the best weights to:

> runs/detect/train/weights/best.pt

## 14. Real-Time Detection Script

Create a file named:

> realtime_detection.py

Add the following:
```python
from ultralytics import YOLO
import cv2
import torch

#Load model (update path to your best.pt)
model = YOLO("runs/detect/train/weights/best.pt")

device = 0 if torch.cuda.is_available() else "cpu"
print(f"Using device: {device}")

cap = cv2.VideoCapture(0)
cap.set(3, 1280)
cap.set(4, 720)

while True:
    ret, frame = cap.read()
    if not ret:
        print("Unable to read camera.")
        break

    results = model.predict(source=frame, device=device, verbose=False, conf=0.5)

    for result in results:
        for box in result.boxes:
            x1, y1, x2, y2 = map(int, box.xyxy[0])
            conf = float(box.conf[0])
            label = model.names[int(box.cls[0])]
            conf_txt = f"{label} {int(conf * 100)}%"

            cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 0, 255), 2)
            cv2.putText(
                frame,
                conf_txt,
                (x1, y1 - 5),
                cv2.FONT_HERSHEY_SIMPLEX,
                0.7,
                (255, 255, 255),
                2,
            )

    cv2.imshow("Real-Time Detection", frame)
    if cv2.waitKey(1) & 0xFF == ord("q"):
        break

cap.release()
cv2.destroyAllWindows()
```

> Run: python realtime_detection.py

> Press ctrl + c to exit.

## Conclusion

This document outlines a comprehensive, reproducible, and technically rigorous workflow for developing a custom YOLO-based object detection system. The process covers dataset creation, annotation, validation, environment configuration, model training, and real-time inference. Each step adheres to professional and academic standards for computer vision experimentation, ensuring the pipeline is suitable for both applied industry environments and research-oriented development.

By following this workflow, you now possess a standardized foundation that allows you to:

- **Systematically scale** your datasets and retrain models as performance requirements evolve  
- **Experiment reproducibly** with different YOLO variants, hyperparameters, and augmentation strategies  
- **Maintain dataset integrity**, ensuring label–image alignment and minimizing annotation-related noise  
- **Deploy trained models** into real-time systems such as surveillance pipelines, human–computer interaction interfaces, robotic platforms, or UAV-based imaging solutions  
- **Extend and customize** the workflow into advanced multi-object, multi-task, or domain-specific detection systems  

---

## 📘 Selecting the Appropriate Number of Epochs

Choosing the correct number of training epochs is essential for balancing underfitting and overfitting:

### **When to use fewer epochs (10–30 epochs):**
- The dataset is **large**, diverse, and well-annotated  
- The model already converges early (loss curves flatten quickly)  
- You want to avoid overfitting or are doing rapid experimentation  

### **When to use moderate epochs (40–80 epochs):**
- The dataset is of **moderate size** (300–1500 images)  
- Classes are balanced and annotation quality is consistent  
- You want stable convergence without excessive training cost  

### **When to use higher epochs (100–200+ epochs):**
- The dataset is **small**, and the model needs more iterations to generalize  
- There is high variability in lighting, background, or object scale  
- The model improves gradually over time (slow convergence)  

> In academic and industrial settings, it is recommended to **monitor validation performance**, not simply increase epochs arbitrarily. Overtraining can degrade accuracy.

---

## 📈 Understanding YOLO Training Graphs

YOLO generates multiple diagnostic curves during training. Knowing how to interpret them is critical for evaluating model behavior.

### **1. Box Loss Curve**
- Measures how well bounding box predictions match ground-truth coordinates  
- A smoothly declining curve indicates improving localization ability  
- Plateauing too early may indicate:
  - insufficient data  
  - poor augmentation  
  - overly complex model for the dataset size  

### **2. Objectness Loss Curve**
- Reflects how effectively the model distinguishes object vs. non-object regions  
- Should decrease progressively  
- High fluctuations suggest:
  - inconsistent annotations  
  - inaccurate bounding boxes  
  - mislabeled or unlabeled objects  

### **3. Classification Loss Curve**
- Evaluates correctness of predicted class labels  
- Should steadily decrease, approaching near-zero for single-class problems  
- Spikes may indicate class imbalance or annotation errors  

### **4. mAP (Mean Average Precision) Curve**
- The primary metric for detection performance  
- mAP@0.5 or mAP@0.5:0.95 increases as the model improves  
- Plateau or decline implies:
  - model saturation  
  - overfitting  
  - insufficient training data  

### **5. Precision–Recall Curves**
- Tell how reliably the model detects objects (precision) vs. how completely it finds all objects (recall)  
- A “bowed-out” curve (close to top-right) indicates strong performance  
- Sharp drop-offs suggest poor thresholding or inconsistent labeling  

---

## 🎯 Final Remarks

The workflow defined here provides a robust, research-friendly baseline for developing high-quality object detection systems. It emphasizes reproducibility, data integrity, modular design, and interpretability — all essential components in modern computer vision development.

This structure enables you to:

- Conduct controlled experiments  
- Scale datasets and models systematically  
- Integrate detection models into real-world applications  
- Communicate results in a scientifically rigorous manner  

Your YOLO pipeline is now fully operational, academically sound, and prepared for further optimization, publication, or deployment in production-grade systems.
