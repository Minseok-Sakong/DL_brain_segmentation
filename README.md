# Fully automatic deep learning brain region segmentation

Project source code is in a private repository

# 1. Project Significance

## a) Drawbacks of current human brain processing pipeline

- Uses FreeSurfer to segment each subject’s brain
- Computationally intensive and time consuming
- Not optimal for large cohort studies where hundreds of subjects need to be processed

## b) Advantages of deep learning based brain processing pipeline

- Fast (under 30 seconds per subject)
- Accurate(84% Dice Score & 72% IoU Score for 100 labels)
- Fully automatic (preprocessing -> prediction -> volume extraction by each brain region)
- Flexible and modular

# 2. Project goal

## Build fully automatic Deep Learning based brain processing pipeline

- Design 3D Unet for brain region segmentation
    
    → 68 Cortical labels and 32 Subcortical labels
    
- Validate model accuracy using random sampling
    
    → Dataset for validation: 30 CN(Controlled subjects), 30 MCI(Mild Cognitive Impairment
    subjects), 30 AD(Alzheimer’s Disease subjects)
    
- Extract volumetric measurements for each brain region

# 3. Dataset

<img width="643" alt="Screen Shot 2022-05-08 at 2 28 55 PM" src="https://user-images.githubusercontent.com/74872384/167310257-d54834d0-1117-47cc-a5d8-732149b4ad71.png">


# 4. 3D Unet Model

![Untitled](Fully%20automatic%20deep%20learning%20brain%20region%20segment%2007da07ed16684b72b8d8107a97529c8c/Untitled.png)

## 3D Unet

- Encoder + Decoder structure with skip connections
- Encoding path : feature extraction
- Decoding path : upsample & concatenate feature maps

# 5. Steps

## 1) Preprocessing

![Screen Shot 2022-05-08 at 1.49.15 PM.png](Fully%20automatic%20deep%20learning%20brain%20region%20segment%2007da07ed16684b72b8d8107a97529c8c/Screen_Shot_2022-05-08_at_1.49.15_PM.png)

a) Raw MRI T1 scans are skull-stripped using FreeSurfer
b) Skull-stripped brains are standardized using Numpy (Intensity-wise)
c) Label masks are filtered only for regions of interest using Python.

## 2) Training with custom data loader

![Screen Shot 2022-05-08 at 1.54.06 PM.png](Fully%20automatic%20deep%20learning%20brain%20region%20segment%2007da07ed16684b72b8d8107a97529c8c/Screen_Shot_2022-05-08_at_1.54.06_PM.png)

### Programmed custom data loader for 3D CNN model

a) Detailed explanation can be found in : https://github.com/Minseok-Sakong/custom_dataloader_tensorflow

b) On each training step, randomly cropped 64X64X64 patches(number of patches depends on the batch size) from one subject T1 scan are fed into the CNN model.

c) There are 148 steps(=number of training dataset) in one epoch.

d) **Cropping patches for training and prediction are necessary for 3D image data (limited computational resource: GPU cannot handle large 3D dataset)**

## 3) Prediction with partially overlapping patches

### → Developed soft voting system for prediction with multiple overlapping patches

Soft voting system: 

1) Multiple label classification predictions on each pixel point

2) Aggregate all the prediction probability in the point

3) Picks the final classification label based on the highest number of votes (highest probability)

### → Prediction with non-overlapping patches VS overlapping patches

![Screen Shot 2022-05-08 at 2.14.08 PM.png](Fully%20automatic%20deep%20learning%20brain%20region%20segment%2007da07ed16684b72b8d8107a97529c8c/Screen_Shot_2022-05-08_at_2.14.08_PM.png)

→ Same model, same weights, but different methods of prediction

→ Soft voting system with overlapping patches increased model prediction along edges of patches

→ Overall mean IOU (Intersection Over Union) score improved by 7%

## 4) Result

![Screen Shot 2022-05-08 at 2.15.43 PM.png](Fully%20automatic%20deep%20learning%20brain%20region%20segment%2007da07ed16684b72b8d8107a97529c8c/Screen_Shot_2022-05-08_at_2.15.43_PM.png)

### 84% Dice Score & 72% IoU Score for 68 cortical and 32 subcortical labels

## 5) Evaluation using random sampling

![Untitled](Fully%20automatic%20deep%20learning%20brain%20region%20segment%2007da07ed16684b72b8d8107a97529c8c/Untitled%201.png)

→ Validated using randomly selected 30 CN, 30 AD, 30 MCI brain scans

→ Dice score : CN(84%), AD(83%), MCI(84%)

## 6) Volume extraction of each brain region for research

![Screen Shot 2022-05-08 at 2.19.40 PM.png](Fully%20automatic%20deep%20learning%20brain%20region%20segment%2007da07ed16684b72b8d8107a97529c8c/Screen_Shot_2022-05-08_at_2.19.40_PM.png)

→ The pipeline automatically extracts volumetric measurements for each brain region after each prediction : Extracted volumetric measures for 404 MCI subjects
