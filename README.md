

# Writeup for ARetail
ARetail allows users to generate a 3D model of a chair from a single 2D image using machine learning and see the generated chair in Augmented Reality.

ARetail is developed by a team from NUS High class of 2019: `Ang Kang Rong, Roy`, `Kyle Zheng Ching Chan` and `Andrew Yapp Wei Rong`.

# Interaction Lifecycle
1. Take a photo of a chair that you want to generate a 3D model of
2. Upload the image to our cloud server
3. Open the ARetail mobile app and choose your chair
4. Place it in Augmented Reality

<p align="center">
	<img src="https://raw.githubusercontent.com/royangkr/ARetail/master/userupload.JPG" height="300"> <img src="https://raw.githubusercontent.com/royangkr/ARetail/master/userAR.JPG" height="300">
</p>

# Documentation of code
This project consists of two parts: 
a **`server`** running a Python script that takes an image of a chair as input and generates a 3D model of that chair as output, and 
a **`mobile app`** that allows users to upload images of chairs and plot 3D models of chairs in Augmented Reality.

## Server running a python script
### Listening to Firebase Storage
The server implements a listener to our [`Firebase Storage`](https://firebase.google.com/docs/storage). When the mobile app uploads an image, the server downloads this image as the input.
### Pre-processing
The background of the image is first removed using the [`openCV`](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_geometric_transformations/py_geometric_transformations.html#geometric-transformations) library. Empty space is added to either the length of width to resize the image into a square.
### 3D Reconstruction 
<p align="center">
<img src="https://raw.githubusercontent.com/royangkr/ARetail/master/reconstruction.JPG" height="200">
</p>

With the 2D image of a chair as input, we perform 3D reconstruction. We first approximate a triangular mesh that resembles the image using a [`Perspective Transformer Network`](https://arxiv.org/abs/1612.00814). This `Perspective Transformer` Network uses a machine learning model that has been pre-trained on more than a thousand images of chairs. We then render silhouettes of this mesh using a [`neural renderer`](https://arxiv.org/abs/1711.07566). The silhouettes and the input image are compared for evaluation and the loss is backpropagated to improve the likeness of the mesh to the input image. This process is iterated to eventually produce a 3D mesh that resembles the input image of a chair from every perspective.
<p align="center">
<img src="https://raw.githubusercontent.com/royangkr/ARetail/master/output.JPG" height="100">
</p>

### Style transfer
From the pre-processed input image of the chair, we extract the colors of the chair. We then perform style transfer by "wrapping" the colors into the texture of the mesh. We then save this colored mesh into a 3D model.
<p align="center">
<img src="https://raw.githubusercontent.com/royangkr/ARetail/master/style_transfer.JPG" height="300">
</p>

### Upload 3D model to Firebase Storage
The generated 3D model is uploaded to our `Firebase Storage` and is now accessible by users through the mobile app.
## Mobile App for User Interface
The ARetail mobile app is coded in Java, using many Android libraries.
### Import a photo to generate a 3D model of
The Import Activity makes use of [`intents`](https://developer.android.com/training/sharing/receive) to allow users to import images either by taking photos in-app or by sharing images to the ARetail app. The user can input a name, price(optional) and dimensions(optional) for the chair. This image is then uploaded to our `Firebase Storage` for the server to generate the 3D model.
### Plot the generated 3D models of chairs in Augmented Reality.
The AR Activity makes use of the [`ARCore`](https://developers.google.com/ar/develop/java/quickstart) library that allows the plotting of 3D models in Augmented Reality(AR). The user can first choose a chair from a dropdown list, which is populated by the available 3D models in our `Firebase Storage`. The user can then place multiple 3D models of chairs in AR in their surroundings. There are 3 key parts of AR: 
`motion tracking` to anchor 3D models in spite of phone motion,
<p align="center">
<img src="https://raw.githubusercontent.com/royangkr/ARetail/master/MotionTracking.jpg" height="150">
</p>
 
 `environmental understanding` which looks for planes where the model can be placed and 
 <p align="center">
 <img src="https://raw.githubusercontent.com/royangkr/ARetail/master/EnvUnderstanding.jpg" height="150">
 </p>
 
 `light estimation` which determines where the shadows will be cast on the object. 
 <p align="center">
 <img src="https://raw.githubusercontent.com/royangkr/ARetail/master/LightEstimation.jpg" height="150">
 </p>
 
