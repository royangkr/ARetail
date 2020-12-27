

# Writeup for ARetail
ARetail allows users to generate a 3D model of a chair from a single 2D image using machine learning and see the generated chair in Augmented Reality.

ARetail is developed by students from NUS High School class of 2019: `Ang Kang Rong Roy`, `Kyle Zheng Ching Chan` and `Andrew Yapp Wei Rong`.

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
### Pre-processing
After an image is uploaded by the mobile app to our [`Firebase Storage`](https://firebase.google.com/docs/storage), the server downloads it as the input. The background of the image is first removed using the [`openCV`](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_geometric_transformations/py_geometric_transformations.html#geometric-transformations) library. Empty space is added to either the length or width to resize the image into a square. 
*\*\*Challenges faced\*\**
*We needed to improve the reconstruction to make better-looking 3D models. Pre-processing immensely improved accuracy by removing irrelevant information that would "distract" the neural network.*
### 3D Reconstruction 
<p align="center">
<img src="https://raw.githubusercontent.com/royangkr/ARetail/master/reconstruction.JPG" height="200">
</p>

With the 2D image of a chair as input, we perform 3D reconstruction. We first approximate a triangular mesh that resembles the image using a [`Perspective Transformer Network`](https://arxiv.org/abs/1612.00814). This `Perspective Transformer` Network uses a machine learning model that has been pre-trained on more than a thousand images of chairs from the ShapeNetCore dataset[[1]](#1). We then render silhouettes of this mesh using a `neural renderer`[[2]](#2). For evaluation, the silhouettes and the input image are compared as well as the smoothness of the mesh and the loss is backpropagated to improve the likeness of the mesh to the input image. 
This process is iterated to eventually produce a 3D mesh that resembles the input image of a chair from every perspective.
<p align="center">
<img src="https://raw.githubusercontent.com/royangkr/ARetail/master/output.JPG" height="100">
</p>

### Style transfer
From the pre-processed input image of the chair, we extract the colors of the chair. We then perform style transfer by "wrapping" the colors into the texture of the mesh. We save this colored mesh into a 3D model as a `.glb` file and upload it to our `Firebase Storage` where it is now accessible by users through the mobile app.
*\*\*Challenges faced\*\**
*After much trial and error and weeks of frustration, we found out that we had to save the model as a scene instead of an object (which was so counter-intuitive) to retain its colors. A `.glb` file contains all data about the mesh and textures, as opposed to a `.gltf` file which is not self-contained or a `.obj` file that does not contain textures*
<p align="center">
<img src="https://raw.githubusercontent.com/royangkr/ARetail/master/style_transfer.JPG" height="300">
</p>

## Mobile App for User Interface
### Import a photo to generate a 3D model
The Import Activity makes use of [`intents`](https://developer.android.com/training/sharing/receive) to allow users to import images either by taking photos in-app or by sharing images to the ARetail app. The user can input a name, price(optional) and dimensions(optional) for the chair. This image is then uploaded to our `Firebase Storage` for the server to generate the 3D model.
### Plot the generated 3D models of chairs in Augmented Reality
The AR Activity makes use of the [`ARCore`](https://developers.google.com/ar/develop/java/quickstart) library that allows the plotting of 3D models in Augmented Reality(AR). Live camera feed is used for the AR and flat surfaces such as floors and tables are detected. The user can first choose a chair from a dropdown list, which is populated by the available 3D models in our `Firebase Storage`. The user can then place multiple 3D models of chairs on these flat surfaces in AR, with realistic dimensions and textures (reflections and shadows) to see if the chair would fit into their surroundings.

## References
<a id="1">[1]</a> Chang, Angel X., et al. "Shapenet: An information-rich 3d model repository." _arXiv preprint arXiv:1512.03012_ (2015).
<a id="2">[2]</a> Kato, Hiroharu, Yoshitaka Ushiku, and Tatsuya Harada. "Neural 3d mesh renderer." _Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition_ (2018).

