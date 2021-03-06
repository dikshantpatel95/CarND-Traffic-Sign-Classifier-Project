
# Self-Driving Car Engineer Nanodegree

## Deep Learning

## Project: Build a Traffic Sign Recognition Classifier

In this notebook, a template is provided for you to implement your functionality in stages, which is required to successfully complete this project. If additional code is required that cannot be included in the notebook, be sure that the Python code is successfully imported and included in your submission if necessary. 

> **Note**: Once you have completed all of the code implementations, you need to finalize your work by exporting the iPython Notebook as an HTML document. Before exporting the notebook to html, all of the code cells need to have been run so that reviewers can see the final implementation and output. You can then export the notebook by using the menu above and navigating to  \n",
    "**File -> Download as -> HTML (.html)**. Include the finished document along with this notebook as your submission. 

In addition to implementing code, there is a writeup to complete. The writeup should be completed in a separate file, which can be either a markdown file or a pdf document. There is a [write up template](https://github.com/udacity/CarND-Traffic-Sign-Classifier-Project/blob/master/writeup_template.md) that can be used to guide the writing process. Completing the code template and writeup template will cover all of the [rubric points](https://review.udacity.com/#!/rubrics/481/view) for this project.

The [rubric](https://review.udacity.com/#!/rubrics/481/view) contains "Stand Out Suggestions" for enhancing the project beyond the minimum requirements. The stand out suggestions are optional. If you decide to pursue the "stand out suggestions", you can include the code in this Ipython notebook and also discuss the results in the writeup file.


>**Note:** Code and Markdown cells can be executed using the **Shift + Enter** keyboard shortcut. In addition, Markdown cells can be edited by typically double-clicking the cell to enter edit mode.


```python
import numpy as np
import matplotlib.pyplot as plt
import random
import tensorflow as tf
from scipy.ndimage import rotate
from tensorflow.contrib.layers import flatten
from sklearn.utils import shuffle
from sklearn.model_selection import train_test_split
```

---
## Step 0: Load The Data


```python
# Load pickled data
import pickle

# TODO: Fill this in based on where you saved the training and testing data

training_file = "../data/train.p"
validation_file= "../data/valid.p"
testing_file = "../data/test.p"

with open(training_file, mode='rb') as f:
    train = pickle.load(f)
with open(validation_file, mode='rb') as f:
    valid = pickle.load(f)
with open(testing_file, mode='rb') as f:
    test = pickle.load(f)
    
X_train, y_train = train['features'], train['labels']
X_valid, y_valid = valid['features'], valid['labels']
X_test, y_test = test['features'], test['labels']

```

---

## Step 1: Dataset Summary & Exploration

The pickled data is a dictionary with 4 key/value pairs:

- `'features'` is a 4D array containing raw pixel data of the traffic sign images, (num examples, width, height, channels).
- `'labels'` is a 1D array containing the label/class id of the traffic sign. The file `signnames.csv` contains id -> name mappings for each id.
- `'sizes'` is a list containing tuples, (width, height) representing the original width and height the image.
- `'coords'` is a list containing tuples, (x1, y1, x2, y2) representing coordinates of a bounding box around the sign in the image. **THESE COORDINATES ASSUME THE ORIGINAL IMAGE. THE PICKLED DATA CONTAINS RESIZED VERSIONS (32 by 32) OF THESE IMAGES**

Complete the basic data summary below. Use python, numpy and/or pandas methods to calculate the data summary rather than hard coding the results. For example, the [pandas shape method](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.shape.html) might be useful for calculating some of the summary results. 

### Provide a Basic Summary of the Data Set Using Python, Numpy and/or Pandas


```python
### Replace each question mark with the appropriate value. 
### Use python, pandas or numpy methods rather than hard coding the results

# TODO: Number of training examples
n_train = len(X_train)

# TODO: Number of validation examples
n_validation =len(X_valid)

# TODO: Number of testing examples.
n_test = len(X_test)

# TODO: What's the shape of an traffic sign image?
image_shape = X_train[0].shape

# TODO: How many unique classes/labels there are in the dataset.
n_classes = len(np.unique(y_train))

print("Number of training examples =", n_train)
print("Number of testing examples =", n_validation)
print("Number of testing examples =", n_test)
print("Image data shape =", image_shape)
print("Number of classes =", n_classes)
```

    Number of training examples = 34799
    Number of testing examples = 4410
    Number of testing examples = 12630
    Image data shape = (32, 32, 3)
    Number of classes = 43


### Include an exploratory visualization of the dataset

Visualize the German Traffic Signs Dataset using the pickled file(s). This is open ended, suggestions include: plotting traffic sign images, plotting the count of each sign, etc. 

The [Matplotlib](http://matplotlib.org/) [examples](http://matplotlib.org/examples/index.html) and [gallery](http://matplotlib.org/gallery.html) pages are a great resource for doing visualizations in Python.

**NOTE:** It's recommended you start with something simple first. If you wish to do more, come back to it after you've completed the rest of the sections. It can be interesting to look at the distribution of classes in the training, validation and test set. Is the distribution the same? Are there more examples of some classes than others?


```python
### Data exploration visualization code goes here.
### Feel free to use as many code cells as needed.
import matplotlib.pyplot as plt
# Visualizations will be shown in the notebook.
%matplotlib inline

fig, axs = plt.subplots(2,5, figsize=(15, 6))
fig.subplots_adjust(hspace = .2, wspace=.001)
axs = axs.ravel()
for i in range(10):
    index = random.randint(0, len(X_train))
    image = X_train[index]
    axs[i].axis('off')
    axs[i].imshow(image)
    axs[i].set_title(y_train[index])
```


![png](output_9_0.png)



```python
#To find number of images in each class
# find number of images for each class
x = np.arange(n_classes)
num_train = np.bincount(train['labels'])
plt.bar(x,num_train)
num_valid = np.bincount(valid['labels'])
plt.bar(x,num_valid)
plt.ylabel('samples')
plt.xlabel('classes')
```




    Text(0.5,0,'classes')




![png](output_10_1.png)



```python
hist, bins = np.histogram(y_train, bins=n_classes)
width = 0.7 * (bins[1] - bins[0])
center = (bins[:-1] + bins[1:]) / 2
plt.bar(center, hist, align='center', width=width)
plt.show()
```


![png](output_11_0.png)


----

## Step 2: Design and Test a Model Architecture

Design and implement a deep learning model that learns to recognize traffic signs. Train and test your model on the [German Traffic Sign Dataset](http://benchmark.ini.rub.de/?section=gtsrb&subsection=dataset).

The LeNet-5 implementation shown in the [classroom](https://classroom.udacity.com/nanodegrees/nd013/parts/fbf77062-5703-404e-b60c-95b78b2f3f9e/modules/6df7ae49-c61c-4bb2-a23e-6527e69209ec/lessons/601ae704-1035-4287-8b11-e2c2716217ad/concepts/d4aca031-508f-4e0b-b493-e7b706120f81) at the end of the CNN lesson is a solid starting point. You'll have to change the number of classes and possibly the preprocessing, but aside from that it's plug and play! 

With the LeNet-5 solution from the lecture, you should expect a validation set accuracy of about 0.89. To meet specifications, the validation set accuracy will need to be at least 0.93. It is possible to get an even higher accuracy, but 0.93 is the minimum for a successful project submission. 

There are various aspects to consider when thinking about this problem:

- Neural network architecture (is the network over or underfitting?)
- Play around preprocessing techniques (normalization, rgb to grayscale, etc)
- Number of examples per label (some have more than others).
- Generate fake data.

Here is an example of a [published baseline model on this problem](http://yann.lecun.com/exdb/publis/pdf/sermanet-ijcnn-11.pdf). It's not required to be familiar with the approach used in the paper but, it's good practice to try to read papers like these.

### Pre-process the Data Set (normalization, grayscale, etc.)
####1. Describe how, and identify where in your code, you preprocessed the image data. What tecniques were chosen and why did you choose these techniques? Consider including images showing the output of each preprocessing technique. Pre-processing refers to techniques such as converting to grayscale, normalization, etc

Minimally, the image data should be normalized so that the data has mean zero and equal variance. For image data, `(pixel - 128)/ 128` is a quick way to approximately normalize the data and can be used in this project. Normalization is important in ANNs because real data obtained from experiments and analysis most times are distant from each other. The effect is great because the common activation functions such as sigmoid, hyperbolic tangent and gaussian produce result that ranges between [0,1] or [-1,1]. It is important to normalise the values to be in that range. By normalizing all of our inputs to a standard scale, we're allowing the network to more quickly learn the optimal parameters for each input node.

Additionally, it's useful to ensure that our inputs are roughly in the range of -1 to 1 to avoid weird mathematical artifacts associated with floating point number precision. In short, computers lose accuracy when performing math operations on really large or really small numbers. Moreover, if your inputs and target outputs are on a completely different scale than the typical -1 to 1 range, the default parameters for your neural network (ie. learning rates) will likely be ill-suited for your data.

Other pre-processing steps are optional. You can try different techniques to see if it improves performance. 

Use the code cell (or multiple code cells, if necessary) to implement the first step of your project.


```python
### Preprocess the data here. It is required to normalize the data. Other preprocessing steps could include 
### converting to grayscale, etc.
### Feel free to use as many code cells as needed.
f128 = np.float32(128)
X_train = (X_train.astype(np.float32)-f128)/f128
X_valid = (X_valid.astype(np.float32)-f128)/f128
X_test = (X_test.astype(np.float32)-f128)/f128

```


```python
X_train, y_train = shuffle(X_train, y_train)
X_valid, y_valid = shuffle(X_valid, y_valid)

##hyperparameters:
EPOCHS = 35
BATCH_SIZE = 60
rate = 0.001
```

### Model Architecture

The code for calculating the accuracy of the model is in cell 10, 11, 12.

My final model results were:

training set accuracy of 0.989 (overfitting the cross validation)
validation set accuracy of 0.93

To train the model, I started from a a well known architecture (LeNet) because of simplicity of implementation and because it performs well on recognition task with tens of classes (such as carachter recognition). After a few runs with this architecture I noted that the model tended to overfit to the original training set.

I used the same parameter given in LeNet lab. Its training accuracy initially was around 90%, so I thought the filter depth was not large enough to capture images' shapes and contents. Previously the filter depth was 6 for the first layer and 12 for the second.

I then added a drop out layer, which is supposed to used to prevent overfitting, but I found a drop out layer could sometimes increase the accuracy to 95%. The final model includes
mu = 0
sigma = 0.1
```python
Layer 1: Convolutional(5 X 5). Input = 32x32x3. Output = 28x28x6.
Activation.(ReLU)
Pooling(2 X 2). Input = 28x28x6. Output = 14x14x6.
Layer 2: Convolutional(5 X 5). Output = 10x10x16.
Activation.(ReLU)
Pooling(2 X 2). Input = 10x10x16. Output = 5x5x16.
Flatten. Input = 5x5x16. Output = 400.
Layer 3: Fully Connected. Input = 400. Output = 120.
Activation.(ReLU)
Layer 4: Fully Connected. Input = 120. Output = 84.
Activation.(ReLU)
Layer 5: Fully Connected. Input = 84. Output = 10.
```


I also tuned epoch, batch_size, and rate parameters, and settled at

##hyperparameters:
EPOCHS = 35
BATCH_SIZE = 60
rate = 0.001

I have my explainations of the effect of the drop out layer after I've seen some of the training data. Some images are too dark to see the sign, so it seems that these images act as noises in the training data and drop out layer can reduce the negative effects on learning.

The final accuracy in validation set is around 0.936

```python
### Define your architecture here.
### Feel free to use as many code cells as needed.
def LeNet(x):    
    # Arguments used for tf.truncated_normal, randomly defines variables for the weights and biases for each layer
    mu = 0
    sigma = 0.1
    
    # SOLUTION: Layer 1: Convolutional. Input = 32x32x3. Output = 28x28x6.
    conv1_W = tf.Variable(tf.truncated_normal(shape=(5, 5, 3, 6), mean = mu, stddev = sigma))
    conv1_b = tf.Variable(tf.zeros(6))
    conv1   = tf.nn.conv2d(x, conv1_W, strides=[1, 1, 1, 1], padding='VALID') + conv1_b

    # SOLUTION: Activation.
    conv1 = tf.nn.relu(conv1)

    # SOLUTION: Pooling. Input = 28x28x6. Output = 14x14x6.
    conv1 = tf.nn.max_pool(conv1, ksize=[1, 2, 2, 1], strides=[1, 2, 2, 1], padding='VALID')

    # SOLUTION: Layer 2: Convolutional. Output = 10x10x16.
    conv2_W = tf.Variable(tf.truncated_normal(shape=(5, 5, 6, 16), mean = mu, stddev = sigma))
    conv2_b = tf.Variable(tf.zeros(16))
    conv2   = tf.nn.conv2d(conv1, conv2_W, strides=[1, 1, 1, 1], padding='VALID') + conv2_b
    
    # SOLUTION: Activation.
    conv2 = tf.nn.relu(conv2)

    # SOLUTION: Pooling. Input = 10x10x16. Output = 5x5x16.
    conv2 = tf.nn.max_pool(conv2, ksize=[1, 2, 2, 1], strides=[1, 2, 2, 1], padding='VALID')

    # SOLUTION: Flatten. Input = 5x5x16. Output = 400.
    fc0   = flatten(conv2)
    
    # SOLUTION: Layer 3: Fully Connected. Input = 400. Output = 120.
    fc1_W = tf.Variable(tf.truncated_normal(shape=(400, 120), mean = mu, stddev = sigma))
    fc1_b = tf.Variable(tf.zeros(120))
    fc1   = tf.matmul(fc0, fc1_W) + fc1_b
    
    # SOLUTION: Activation.
    fc1    = tf.nn.relu(fc1)

    # SOLUTION: Layer 4: Fully Connected. Input = 120. Output = 84.
    fc2_W  = tf.Variable(tf.truncated_normal(shape=(120, 84), mean = mu, stddev = sigma))
    fc2_b  = tf.Variable(tf.zeros(84))
    fc2    = tf.matmul(fc1, fc2_W) + fc2_b
    
    # SOLUTION: Activation.
    fc2    = tf.nn.relu(fc2)

    # SOLUTION: Layer 5: Fully Connected. Input = 84. Output = 10.
    fc3_W  = tf.Variable(tf.truncated_normal(shape=(84, 43), mean = mu, stddev = sigma))
    fc3_b  = tf.Variable(tf.zeros(43))
    logits = tf.matmul(fc2, fc3_W) + fc3_b
    
    return logits
```

### Train, Validate and Test the Model

A validation set can be used to assess how well the model is performing. A low accuracy on the training and validation
sets imply underfitting. A high accuracy on the training set but low accuracy on the validation set implies overfitting.


```python
### Train your model here.
### Calculate and report the accuracy on the training and validation set.
### Once a final model architecture is selected, 
### the accuracy on the test set should be calculated and reported as well.
### Feel free to use as many code cells as needed.
x = tf.placeholder(tf.float32, (None, 32, 32, 3))
y = tf.placeholder(tf.int32, (None))
keep_prob = tf.placeholder(tf.float32)
one_hot_y = tf.one_hot(y, n_classes)
```


```python
logits = LeNet(x)
cross_entropy = tf.nn.softmax_cross_entropy_with_logits(labels=one_hot_y, logits=logits)
loss_operation = tf.reduce_mean(cross_entropy)
optimizer = tf.train.AdamOptimizer(learning_rate = rate)
training_operation = optimizer.minimize(loss_operation)
```


```python
correct_prediction = tf.equal(tf.argmax(logits, 1), tf.argmax(one_hot_y, 1))
accuracy_operation = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
saver = tf.train.Saver()

def evaluate(X_data, y_data):
    num_examples = len(X_data)
    total_accuracy = 0
    sess = tf.get_default_session()
    for offset in range(0, num_examples, BATCH_SIZE):
        batch_x, batch_y = X_data[offset:offset+BATCH_SIZE], y_data[offset:offset+BATCH_SIZE]
        accuracy = sess.run(accuracy_operation, feed_dict={x: batch_x, y: batch_y})
        total_accuracy += (accuracy * len(batch_x))
    return total_accuracy / num_examples
```


```python
## train and evaluate
with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    num_examples = len(X_train)
    t_acc=[]
    v_acc=[]
    print("Training...")
    print()
    for i in range(EPOCHS):
        X_train, y_train = shuffle(X_train, y_train)
        for offset in range(0, num_examples, BATCH_SIZE):
            end = offset + BATCH_SIZE
            batch_x, batch_y = X_train[offset:end], y_train[offset:end]
            sess.run(training_operation, feed_dict={x: batch_x, y: batch_y})
            
        training_accuracy = evaluate(X_train, y_train)
        validation_accuracy = evaluate(X_valid, y_valid)
        t_acc.append(training_accuracy)
        v_acc.append(validation_accuracy)
        print("EPOCH {} ...".format(i+1))
        print("Training Accuracy = {:.3f}".format(training_accuracy))
        print("Validation Accuracy = {:.3f}".format(validation_accuracy))
    x_ax = np.arange(EPOCHS)
    plt.plot(x_ax, t_acc)
    plt.plot(x_ax, v_acc)
    plt.ylabel('accuracy')
    plt.xlabel('epochs')
    
    saver.save(sess, './lenet')
    print("Model saved")
```

    Training...
    
    EPOCH 1 ...
    Training Accuracy = 0.879
    Validation Accuracy = 0.802
    EPOCH 2 ...
    Training Accuracy = 0.958
    Validation Accuracy = 0.862
    EPOCH 3 ...
    Training Accuracy = 0.978
    Validation Accuracy = 0.890
    EPOCH 4 ...
    Training Accuracy = 0.968
    Validation Accuracy = 0.874
    EPOCH 5 ...
    Training Accuracy = 0.979
    Validation Accuracy = 0.883
    EPOCH 6 ...
    Training Accuracy = 0.992
    Validation Accuracy = 0.903
    EPOCH 7 ...
    Training Accuracy = 0.977
    Validation Accuracy = 0.885
    EPOCH 8 ...
    Training Accuracy = 0.991
    Validation Accuracy = 0.889
    EPOCH 9 ...
    Training Accuracy = 0.994
    Validation Accuracy = 0.895
    EPOCH 10 ...
    Training Accuracy = 0.989
    Validation Accuracy = 0.902
    EPOCH 11 ...
    Training Accuracy = 0.992
    Validation Accuracy = 0.902
    EPOCH 12 ...
    Training Accuracy = 0.994
    Validation Accuracy = 0.917
    EPOCH 13 ...
    Training Accuracy = 0.990
    Validation Accuracy = 0.901
    EPOCH 14 ...
    Training Accuracy = 0.993
    Validation Accuracy = 0.906
    EPOCH 15 ...
    Training Accuracy = 0.997
    Validation Accuracy = 0.911
    EPOCH 16 ...
    Training Accuracy = 0.993
    Validation Accuracy = 0.904
    EPOCH 17 ...
    Training Accuracy = 0.989
    Validation Accuracy = 0.909
    EPOCH 18 ...
    Training Accuracy = 0.998
    Validation Accuracy = 0.916
    EPOCH 19 ...
    Training Accuracy = 0.999
    Validation Accuracy = 0.927
    EPOCH 20 ...
    Training Accuracy = 0.994
    Validation Accuracy = 0.914
    EPOCH 21 ...
    Training Accuracy = 0.999
    Validation Accuracy = 0.930
    EPOCH 22 ...
    Training Accuracy = 0.998
    Validation Accuracy = 0.926
    EPOCH 23 ...
    Training Accuracy = 0.998
    Validation Accuracy = 0.919
    EPOCH 24 ...
    Training Accuracy = 0.995
    Validation Accuracy = 0.910
    EPOCH 25 ...
    Training Accuracy = 0.996
    Validation Accuracy = 0.920
    EPOCH 26 ...
    Training Accuracy = 0.996
    Validation Accuracy = 0.921
    EPOCH 27 ...
    Training Accuracy = 0.997
    Validation Accuracy = 0.923
    EPOCH 28 ...
    Training Accuracy = 0.996
    Validation Accuracy = 0.910
    EPOCH 29 ...
    Training Accuracy = 0.991
    Validation Accuracy = 0.905
    EPOCH 30 ...
    Training Accuracy = 0.993
    Validation Accuracy = 0.915
    EPOCH 31 ...
    Training Accuracy = 0.997
    Validation Accuracy = 0.920
    EPOCH 32 ...
    Training Accuracy = 0.999
    Validation Accuracy = 0.931
    EPOCH 33 ...
    Training Accuracy = 0.998
    Validation Accuracy = 0.933
    EPOCH 34 ...
    Training Accuracy = 0.992
    Validation Accuracy = 0.912
    EPOCH 35 ...
    Training Accuracy = 0.989
    Validation Accuracy = 0.905
    Model saved



![png](output_24_1.png)



```python
### Load the images and plot them here.
### Feel free to use as many code cells as needed.
with tf.Session() as sess:
    saver.restore(sess, tf.train.latest_checkpoint('.'))

    test_accuracy = evaluate(X_test, y_test)
    print("Test Accuracy = {:.3f}".format(test_accuracy))
```

    INFO:tensorflow:Restoring parameters from ./lenet
    Test Accuracy = 0.936


---

## Step 3: Test a Model on New Images

To give yourself more insight into how your model is working, download at least five pictures of German traffic signs from the web and use your model to predict the traffic sign type.

You may find `signnames.csv` useful as it contains mappings from the class id (integer) to the actual sign name.

### Load and Output the Images

I want to see how the classifier performs on similar type of traffic signs. I Chose one sign which reads clear and one side slightly tilted since when blurred the number in the sign is not very ligible. Also i chode different types of caution sign like deer jumping, slippery road and the bike crossing. I wanted to see if the classifier coulf actually differntiate between similar looking images.


```python
import os
import matplotlib.image as mpimg
import matplotlib.pyplot as plt
import cv2
import numpy as np

image_list = os.listdir("my_test/")
print(image_list)
web_imgs = np.zeros((len(image_list)-1,32,32,3))    
# iterate through the images
for i in range(0,len(image_list)-1):
    image_orig = mpimg.imread("my_test/" + image_list[i])
    image = cv2.resize(image_orig, dsize=(32, 32), interpolation=cv2.INTER_CUBIC)
    web_imgs[i] = image
    import numpy as np

web_imgs_class = np.array([1, 11, 31, 23, 3])
    
#display images
fig=plt.figure()
for i in range(1, len(web_imgs)+1):
    img = web_imgs[i-1]
    fig.add_subplot(len(web_imgs),1, i)
    plt.imshow(img)
plt.show()   
```

    ['30.jpg', '60.jpg', 'slippery_road.jpg', 'animal_crossing.jpg', '1x.png', 'bikes.jpg']



![png](output_28_1.png)



```python
# preprocess
##normalize data
f128 = np.float32(128)
web_imgs = (web_imgs.astype(np.float32)-f128)/f128
print(np.min(web_imgs))

prediction = tf.argmax( logits, 1)
```

    -1.0


### Predict the Sign Type for Each Image


```python
with tf.Session() as sess:
    saver.restore(sess, tf.train.latest_checkpoint('.'))
    output = sess.run(prediction, feed_dict={x: web_imgs, y: web_imgs_class})
    
print("labels:", web_imgs_class )
print("result", output)
```

    INFO:tensorflow:Restoring parameters from ./lenet
    labels: [ 1 11 31 23  3]
    result [41 13 23 30 38]


### Analyze Performance


```python
### Calculate the accuracy for these 5 new images. 
### For example, if the model predicted 1 out of 5 signs correctly, it's 20% accurate on these new images.
count=0
for i in range(0, len(output)):
    
    if output[i] == web_imgs_class[i]:
        count = count+1
        
print("accuracy = ", (count/len(output)*100), "%")
```

    accuracy =  0.0 %
 
### Question 7
Is your model able to perform equally well on captured pictures when compared to testing on the dataset? The simplest way to do this check the accuracy of the predictions. For example, if the model predicted 1 out of 5 signs correctly, it's 20% accurate.

NOTE: You could check the accuracy manually by using signnames.csv (same directory). This file has a mapping from the class id (0-42) to the corresponding sign name. So, you could take the class id the model outputs, lookup the name in signnames.csv and see if it matches the sign from the image.

Answer:

Umm...  I do not think the model performed accurately. I guess it is a fault on my side too i should have tried it with different sets of images. I guess i did not have enough time to try on multiple sets of images and also i guess the existing set is a little bit challenging. I tried 5 images it predicted 1 out of 5 correctly for the initial set but the images i submitted were predicted 0% accurately. 


### Output Top 5 Softmax Probabilities For Each Image Found on the Web

For each of the new images, print out the model's softmax probabilities to show the **certainty** of the model's predictions (limit the output to the top 5 probabilities for each image). [`tf.nn.top_k`](https://www.tensorflow.org/versions/r0.12/api_docs/python/nn.html#top_k) could prove helpful here. 

The example below demonstrates how tf.nn.top_k can be used to find the top k predictions for each image.

`tf.nn.top_k` will return the values and indices (class ids) of the top k predictions. So if k=3, for each sign, it'll return the 3 largest probabilities (out of a possible 43) and the correspoding class ids.

Take this numpy array as an example. The values in the array represent predictions. The array contains softmax probabilities for five candidate images with six possible classes. `tf.nn.top_k` is used to choose the three classes with the highest probability:

```
# (5, 6) array
a = np.array([[ 0.24879643,  0.07032244,  0.12641572,  0.34763842,  0.07893497,
         0.12789202],
       [ 0.28086119,  0.27569815,  0.08594638,  0.0178669 ,  0.18063401,
         0.15899337],
       [ 0.26076848,  0.23664738,  0.08020603,  0.07001922,  0.1134371 ,
         0.23892179],
       [ 0.11943333,  0.29198961,  0.02605103,  0.26234032,  0.1351348 ,
         0.16505091],
       [ 0.09561176,  0.34396535,  0.0643941 ,  0.16240774,  0.24206137,
         0.09155967]])
```

Running it through `sess.run(tf.nn.top_k(tf.constant(a), k=3))` produces:

```
TopKV2(values=array([[ 0.34763842,  0.24879643,  0.12789202],
       [ 0.28086119,  0.27569815,  0.18063401],
       [ 0.26076848,  0.23892179,  0.23664738],
       [ 0.29198961,  0.26234032,  0.16505091],
       [ 0.34396535,  0.24206137,  0.16240774]]), indices=array([[3, 0, 5],
       [0, 1, 4],
       [0, 5, 1],
       [1, 3, 5],
       [1, 4, 3]], dtype=int32))
```

Looking just at the first row we get `[ 0.34763842,  0.24879643,  0.12789202]`, you can confirm these are the 3 largest probabilities in `a`. You'll also notice `[3, 0, 5]` are the corresponding indices.


```python
### Print out the top five softmax probabilities for the predictions on the German traffic sign images found on the web. 
### Feel free to use as many code cells as needed.
import tensorflow as tf
with tf.Session() as sess:
    # Load the weights and bias
    saver.restore(sess, tf.train.latest_checkpoint('.'))
    top5 = sess.run(tf.nn.top_k(tf.nn.softmax(logits), k=5, sorted=True), 
                    feed_dict={x: web_imgs, y: web_imgs_class})
     #print(top5)
for i in range (0, len(web_imgs_class)):
    print("expected class for picture:", web_imgs_class[i] )
    print("top 5 classes: ", top5[1][i])
    print("softmax probabilities", top5[0][i])
```

    INFO:tensorflow:Restoring parameters from ./lenet
    expected class for picture: 1
    top 5 classes:  [41 42 17 20  9]
    softmax probabilities [  1.00000000e+00   1.67463876e-09   5.37759663e-11   7.23488023e-12
       7.11000920e-12]
    expected class for picture: 11
    top 5 classes:  [13  3  2  9  7]
    softmax probabilities [  9.93856370e-01   3.10786022e-03   2.81627499e-03   2.18298257e-04
       1.12667533e-06]
    expected class for picture: 31
    top 5 classes:  [23 24 25 19 29]
    softmax probabilities [  7.72361696e-01   2.27638304e-01   1.59286717e-09   1.08934024e-25
       5.25846745e-27]
    expected class for picture: 23
    top 5 classes:  [30 31 25 21 11]
    softmax probabilities [  9.94270086e-01   4.06771293e-03   1.59097149e-03   6.58869176e-05
       5.27392194e-06]
    expected class for picture: 3
    top 5 classes:  [38 33 20 35 30]
    softmax probabilities [ 0.40232021  0.30905971  0.13223937  0.053587    0.04113337]


For the softmax probablities i guess they are fairly inaccurately accurate. It really mispredicts the signs with a high probability, atleat for the 1st second and the 4th image. I guess i might have to improve my model and also have other set of images and try to find what the output looks like.

