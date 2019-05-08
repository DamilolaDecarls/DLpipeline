## DLpipeline
### A Guide on Deep Learning for Genomic Prediction: A Keras based pipeline to implement deep learning
#### M Pérez-Enciso & LM Zingaretti
#### miguel.perez@uab.es, laura.zingaretti@cragenomica.es

If you find this resource useful, please cite:

Pérez-Enciso M, Zingaretti LM, 2019. A Guide on Deep Learning for Genomic Prediction. submitted

 * * *
 
Implementing DL, despite all its theoretical and computational complexities, is rather easy. This is thanks to Keras API (https://keras.io/) and TensorFlow (https://www.tensorflow.org/), which allow all intricacies to be encapsulated through very simple statements. TensorFlow is a machine-learning library developed by Google. In addition, the machine-learning python library scikit-learn (https://scikit-learn.org) is highly useful. Directly implementing DL in TensorFlow requires some knowledge of DL algorithms, and understanding the philosophy behind tensor (i.e., n-dimensional objects) manipulations. Fortunately, this can be avoided using Keras, a high-level python interface for TensorFlow and other DL libraries. Although alternatives to TensorFlow and Keras exist, we believe these two tools combined are currently the best options: they are simple to use and are well documented. 

Here we describe some Keras implementation details. Complete code is in [jupyter notebook](https://github.com/miguelperezenciso/DLpipeline/blob/master/PDL.ipynb), and example data are [DATA](https://github.com/miguelperezenciso/DLpipeline/tree/master/DATA) folder. To run the script, you need to have installed Keras and TensorFlow, preferably in a computer with GPU architecture. Installing TensorFlow, especially for the GPU architecture, may not be a smooth experience. If unsolved, an alternative is using a docker (i.e., a virtual machine) with all functionalities built-in, or a cloud-based machine already configured. One option is https://github.com/floydhub/dl-docker. 

### A Generic Keras Pipeline

An analysis pipeline in Keras requires of five main steps:
* A model is instantiated: The most usual model is ```Sequential```, which allows adding layers with different properties step by step.
* The architecture is defined: Here, each layer and its properties are defined. For each layer, number of neurons, activation functions, regularization and initialization methods are specified.
* The model is compiled: Optimizer algorithm with associated parameters (e.g., learning rate) and loss function are specified. This step allows us to symbolically define the operations (‘graphs’) to be performed later with actual numbers.
* Training: The model is fitted to the data and parameters are estimated. The number of iterations (‘epochs’) and batch size are specified, input and target variables need to be provided. The input data size must match that defined in step 2.
* Model predictions are validated via cross-validation.

A generic Keras script would look like:

```
# Load modules needed
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split

# keras items 
from keras.models import Sequential
from keras.layers import Dense, Activation

# Load the dataset as a pandas data frame
# X is a N by nSNP array with SNP genotypes
X = pd.read_csv('DATA/wheat.X', header=None, sep='\s+')
# Y is a N b nTRAIT array with phenotypes
Y = pd.read_csv('DATA/wheat.Y', header=None, sep='\s+')
# The first trait is analyzed
y = Y[0] 

# Data partitioning into train and test (20%)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# no. of SNPs in data
nSNP = X_train.shape[1] 

# Instantiate model
model = Sequential()

# Add first layer containing 64 neurons
model.add(Dense(64, input_dim=nSNP))
model.add(Activation('relu'))
# Add second layer, with 32 neurons
model.add(Dense(32))
model.add(Activation('softplus'))
# Last, output layer contains one neuron (ie, target is a real numeric value)
model.add(Dense(1))

# Model Compiling 
model.compile(loss='mean_squared_error', optimizer='sgd')

# list some properties of the network
model.summary()

# Training
model.fit(X_train, y_train, epochs=100)

# Cross-validation: get predicted target values
y_hat = model.predict(X_test)

# Computes squared error in prediction
mse_prediction = model.evaluate(X_test, y_test)
```

### Implementing Multilayer Perceptrons (MLPs)
In Keras, a MLP is implemented by adding ‘dense’ layers. In the following code, a two layer MLP with 64 and 32 neurons is defined, where the input dimension is 200 (i.e., the number of SNPs):

```
from keras.models import Sequential
from keras.layers import Dense, Activation

nSNP=200 # no. of SNPs in data
# Instantiate
model = Sequential()
# Add first layer
model.add(Dense(64, input_dim=nSNP))
model.add(Activation('relu'))
# Add second layer
model.add(Dense(32))
model.add(Activation('softplus'))
# Last, output layer with linear activation (default)
model.add(Dense(1))
```

As is clear from the code, activation functions are ‘relu’ and ‘softplus’ in the first and second layer, respectively.

### Implementing Convolutional Neural Networks (CNNs)
The following Keras code illustrates how a convolutional layer with max pooling is applied prior to the MLP described above:

```
from keras.models import Sequential
from keras.layers import Dense, Activation 
from keras.layers import Flatten, Conv1D, MaxPooling1D

nSNP=200 # no. of SNPs in data
nStride=3 # stride between convolutions
nFilter=32 # no. of convolutions

model = Sequential()
# add convolutional layer
model.add(Conv1D(nFilter, 
kernel_size=3, 
strides=nStride, 		
input_shape=(nSNP,1)))
# add pooling layer: here takes maximum of two consecutive values
model.add(MaxPooling1D(pool_size=2))
# Solutions above are linearized to accommodate a standard layer
model.add(Flatten())
model.add(Dense(64))
model.add(Activation('relu'))
model.add(Dense(32))
model.add(Activation('softplus'))
model.add(Dense(1))
```

### Implementing Generative Networks 
A Keras implementation of GANs can be found at https://github.com/eriklindernoren/Keras-GAN. 

### Implementing Recurrent Neural Neiworks (RNNs)
The following model is a simple implementation of 3 layers of LSTM with 256 neurons per layer:
 
```
from keras.models import Sequential
from keras.layers import Dense, Activation

nSNP=200 # no. of SNPs in data

# Instantiate
model = Sequential()
model.add(LSTM(256,return_sequences=True, input_shape=(None,1), activation=’tanh’))
model.add(Dropout(0.1))
model.add(LSTM(256, return_sequences=True, activation=’tanh’))
model.add(Dropout(0.1))
model.add(LSTM(256, activation=’tanh’))
model.add(Dropout(0.1))
model.add(Dense(units=1))
model.add(Activation(’tanh’))
model.compile(loss=mse, optimizer=adam, metrics=['mae'])

# prints some details
model.summary()
```

### Loss
The loss is a measure of how differences between observed and predicted target variables are quantified. Keras allows three simple metrics to deal with quantitative, binary or multiclass outcome variables: mean squared error, binary cross entropy and multiclass cross entropy, respectively. Several other losses are also possible or can be manually specified. 

Categorical cross-entropy is defined, for *M* classes, as 
 
&sum;<sub>i=1</sub>&sum;<sub>c=1</sub>&gamma;log(p<sub>ic</sub>), with i=1..N, c=1..M

where *N*is the number of observations, *&gamma* is an indicator variable taking value 1 if i-th observation pertains to c-th class and 0 otherwise, and *Pic* is the predicted probability for i-th observation of being of class c. 

Losses are declared in compiling the model:

```
# Stochastic Gradient Descent (‘sgd’) as optimization algorithm
# quantitative variable, regression
model.compile(loss='mean_squared_error', optimizer=’sgd’)

# binary classification
model.compile(loss='binary_crossentropy', optimizer=’sgd’)

# multi class classification
model.compile(loss='categorical_crossentropy', optimizer=’sgd’)

When using categorical losses, your targets should be in categorical format. In order to convert integer targets into categorical targets, you can use the Keras utility to_categorical:

from keras.utils import to_categorical
categorical_labels = to_categorical(int_labels, num_classes=None)
```

See https://keras.io/utils/#to_categorical. 

### Activation Functions
In Keras, activation is defined for every Dense layer as

```model.add(Activation(‘activation’))```

where ```‘activation’``` can take values ‘sigmoid’, ‘relu’, etc (https://keras.io/activations/). The activation by default in Keras is ‘linear’, i.e., no function.  

### Protection against Overfitting
Keras allows implementing **early stopping** via the callback procedure. The user needs to provide a monitored quantity, say test loss, and the program stops when it stops improving (https://keras.io/callbacks/#earlystopping):

```
from keras.callbacks import EarlyStopping, Callback

early_stopper = EarlyStopping(monitor='val_loss', 					
                min_delta=0.1, 
                patience=2, 
                verbose=0, 
                mode='auto')
		
model.fit(X_train, 
          y_train, 
          epochs=100, 		
          verbose=1, 
          validation_data(X_test, y_test), 
          callbacks=[early_stopper])
```

In Keras, the available **regularizers** are L1 and L2 norm regularizers, which can also be combined in the so called ‘Elastic Net’ procedure, i.e., a mixed L1 and L2 regularization. In Keras, regularizers are applied to either kernels (weights), bias or activity (neuron output) and are specified together with the rest of layer properties, e.g.:

```
from keras.models import Sequential
from keras.layers import Dense, Activation 
from keras import regularizers

model.add(Dense(64, input_dim=64,
kernel_regularizer=regularizers.l2(0.01), 
activity_regularizer=regularizers.l1(0.01)))
```

In Keras, different **dropout** rates can be specified for each layer, after its definition, e.g.:

```
model.add(Dense(30, activation='relu'))
model.add(Dropout(0.2))
```
