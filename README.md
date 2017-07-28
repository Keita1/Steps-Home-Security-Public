# Steps-Home-Security

[Motivation](#motivation)  
[Data Collection & Preprocessing](#data-collection-and-preprocessing)  
[Dimensionality Reduction & Classification](#dimensionality-reduction-and-classification)  



A security system that uses data science (NMF and SVM) and signal processing (discrete wavelet packets) methods to detect an intruders in a home or business.

# Motivation
<img src="https://github.com/Keita1/Steps-Home-Security-Public/blob/master/awake.png" alt="Drawing" style="width: 300px;"/>  

A noise startled me awake at 3AM. As I gathered my wits, I could clearly hear someone clammering about downstairs. I was pretty sure it wasn't my son, and my wife was beside me, asleep. With an adrenaline rush, I snuck downstairs to investigate. I discovered a malfunctioning Roomba whose frequent furniture bumping caused the disturbing clatter.

This experience led motivated me to ask what if I could program my phone to tell me if the noise downstairs was benign or not? And what if I could use such a system to remotely alert me about goings-on in my home? Security cameras are one solution, but have limitations. By strategically placing vibration sensors (accelerometers) on the ground around the home, IoT technology can be used to push alerts about who is in the home and what they are doing.


# Data Collection and Preprocessing
Accelerometer data (in one of 3 directions) is collected via iPhone accelerometer and an Arduino-based accelerometer. This time-series data split into small, overlapping frames. Here is what that framed, raw data looks like:  

<img src="https://i.imgsafe.org/532b512.png" alt="Drawing" style="width: 600px;"/>
The above figure takes a few seconds of data and turns it into eight overlapping pieces.  

Using just the time-series accelerometer data for our task would be hard, since many events and activities could be mixed in with this data. To pull out the required detail, we want to study the signal in both time and frequency space.  

To do this, each of these pieces (or frames of data) is transformed into a time-frequency format using a discrete wavelet packet transform (WPT). The WPT is converted into a 2-D spectrogram, the pixels from which the features are derived.  

In my code, I use what's called a discrete WPT. An example of raw data, the DWPT, and the corresponding spectrogram is shown below. Each peak in the raw data shows the impact of a foot on the ground in a step.
<img src="https://i.imgsafe.org/c8ae4c8.png" alt="Drawing" style="width: 450px;"/>  

More pleasing to the human eye is a continuous wavelet packet transform, shown below with the corresponding raw accelerometer data. Again, each peak in the raw data shows the impact of a foot on the ground in a step.
<img src="https://i.imgsafe.org/603e05a.png" alt="Drawing" style="width: 500px;"/>   

A last step in pre-processing is aimed at data reduction. An entire spectrogram is created for a short span of time, say a second. I take a small slice of the WPT for each second and take its reduced spectrogram. I then bind a few of these smaller spectrograms together. In the figure below, each slice corresponds to one second. So, if I want to build a 10 second snapshot, I concatenate 10 such slices.  

<img src="https://i.imgsafe.org/b6c9cd5.png" alt="Drawing" style="width: 300px;"/>   

# Dimensionality Reduction and Classification
Each combined spectrogram can contain hundreds of features. The typical spectrogram I used contained 1120 features. To make the model more tractable, we reduce the dimensionality of these observations.  

Non-Negative Factorization (NMF) was used to reduce the features from 1120 to 8 components.

The decomposed observations were then used to train classification models. These models were based on support vector machine (SVM) methodology. This particular SVC used a degree 3 polynomial kernal.

Three models were trained:  
Model 1. Whether or not there is activity (2 Classes)  

If there was activity:  
Model 2. Counting the number of people present (2 Classes)  
Model 3. Determining whether or not an intruder is present (6 Classes)

For Model 3, initially a one-class SVM method was tried, but due to the overlap of observation data, it was unsuitable. The optimum method so far is to take a sample observation and calculate the probability is comes from any of the classes in a house hold. If this probability is below a certain threshold, this will signify an intruder.


### Results

Below are the results from a 6 model classification.
<img src="https://i.imgsafe.org/1ca0336.png" alt="Drawing" style="width: 500px;"/>


### What's Next?

The results can be improved in several ways that will improve the performance of the classification model. Here are a few that will be tried in the near future:

 - Featurization: Using a Continuous Wavelet Transform.   
 - Refine Classification Model by using a custom kernel. I believe that this will allow a OCSVM to be used for intruder detection.
 - Create models to distinguish multiple people, and be invariant to footwear/surface
 - Add Behavior Detection: e.g., enter room, leave room, get a snack, rob house
 - Sensor/HW  Setup: use a setup that allows a continuous Data Stream from Arduino Sensors. And utilize multiple, simultaneous sensors


### Packages and Software Used

##### Data Collection:
Accel Pro - (iphone App - from Wavefront Labs; Allows collection of accelerometer data directly from iPhone)  
Arduino HW and SW  
Socket  
Traceback  
CSV    

##### Pre-processing
PyWavelets (Wavelet package, used for discrete wavelet packet transformation (DWPT)
Obspy (package for processing Seismic data, used for continuous wavelet packet transformation (CWPT)

##### Decomposition & Modeling
SKLearn (NMF for Decomposition and modeling )



### References

1. "Kernel Non-Negative Matrix Factorization for Seismic Signature Separation" - Mehmood and Damarla (2013)
2. "Seismic Target Classification Using a Wavelet Packet Manifoldin Unattended Ground Sensors Systems" - Huang, et al. (2013)3. "Wavelet Packet Feature Extraction for Vibration Monitoring" - Yen and Lin (1999)
