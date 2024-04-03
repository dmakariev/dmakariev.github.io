---
title: "A Java Neural Network Tutorial: Classifying MNIST Handwritten Digits"
tags: [ai, jbang, java, neural network, mnist, popular]
thumbnail-img: "/assets/img/blog/mnist-coffee-1.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
mermaid: true
mathjax: true
---

Understanding neural networks is one thing, but seeing them in action is quite another. The MNIST database, a large collection of handwritten digits, is the perfect playground to train and test a neural network for image recognition. In this blog post, we will build upon our previous ["Crafting a Neural Network in Just One File! Java with JBang"](https://www.makariev.com/blog/crafting-neural-network-in-one-file-java-with-jbang/) and use the MNIST Dataset described in ["Exploring the Classic MNIST: A Benchmark for Machine Learning Models"](https://www.makariev.com/blog/exploring-the-classic-MNIST-Benchmark-for-machine-learning-models/) to develop a fully functioning neural network for digit classification.

* toc
{:toc}

# Prerequisites
Before we dive into the development process, ensure you have:
* JBang installed on your system. You can install it from [JBang's official website](https://www.jbang.dev/download/).

* You can clone the `https://github.com/dmakariev/examples` repository.
```bash
git clone https://github.com/dmakariev/examples.git
cd examples/artificial-intelligence/neural-network-compare
```

# The MNIST Playground
The MNIST dataset is a cornerstone of machine learning that contains 70,000 images of handwritten digits, each 28x28 pixels. Each image is a monochrome with pixel-values ranging from 0 to 255, representing the intensity of black. Training a model on such a dataset gives a straightforward benchmark for learning and performance assessment.
For simplicity, we are going to use the CSV version provided on the kaggle website :
 
 [https://www.kaggle.com/datasets/oddrationale/mnist-in-csv](https://www.kaggle.com/datasets/oddrationale/mnist-in-csv)
 
The dataset consists of two files:

* `mnist_train.csv`
* `mnist_test.csv`

The `mnist_train.csv` file contains the 60,000 training examples and labels. The `mnist_test.csv` contains 10,000 test examples and labels. Each row consists of 785 values: the first value is the label (a number from 0 to 9) and the remaining 784 values are the pixel values (a number from 0 to 255).

Due to the size of the files `mnist_train.csv` is ~110Mb and `mnist_test.csv` is ~18Mb , we are going to use them from inside a zip file `dataset/dataset-MNIST.zip` ( ~ 16Mb )

# Crafting the Neural Network with Java and JBang
We'll use Java as our language of choice, combined with the JBang scripting tool, to construct our neural network from scratch. JBang allows us to run our Java programs with a script-like feel, removing much of the boilerplate associated with traditional Java applications.

## Step 1: Setting Up the Neural Network Architecture
Using the `NeuralNetTutorial` class we've crafted before, we establish the neural network's architecture. For the MNIST dataset, we choose a simple yet effective layout:

An input layer with 784 neurons, corresponding to the 28x28 pixels of the MNIST images.
A hidden layer with 64 neurons, a figure that offers a balance between complexity and computational efficiency.
An output layer with 10 neurons, representing the digits 0 through 9.
We apply activation functions like the **leaky ReLU** to introduce non-linearity, which helps the network learn complex patterns in the dataset.

In the updated version of our `NeuralNetTutorial`, we've modularized the code by extracting the `TrainingData` into a distinct utility class. Additionally, we have shifted from employing **Random Initialization** to the **He Initialization** method for setting up our network's weights. Initialization in the context of neural networks refers to the process of setting the initial weights and biases of the network before training begins. Proper initialization is crucial because it can significantly affect the convergence rate and the quality of the final solution that the training process yields.
**He Initialization** is named after Kaiming He, it is specifically designed for layers with ReLU activation functions, but could also be used with other [activation functions](https://www.makariev.com/blog/crafting-neural-network-in-one-file-java-with-jbang/#common-activation-functions). The weights are initialized keeping in mind the size of the previous layer which helps in attaining a global optimum during the training process.
```java
    double stddev = Math.sqrt(2.0 / numOutputs); // Standard deviation for He initialization

    for (int c = 0; c < numOutputs; ++c) {
        outputWeights.add(new Connection(rand.nextGaussian() * stddev, 0));
    }
```

 While the default activation function is now the **leaky ReLU**, the code allows for easy substitution of this function, enabling users to experiment with different activation functions and observe their corresponding outcomes.
 
 ```java
     private final static ActivationFunction ACTIVATION_FUNCTION = ActivationFunction.leakyReLU();
    //private final static ActivationFunction ACTIVATION_FUNCTION = ActivationFunction.ReLU();
    //private final static ActivationFunction ACTIVATION_FUNCTION = ActivationFunction.tanh();
    //private final static ActivationFunction ACTIVATION_FUNCTION = ActivationFunction.sigmoid();
 ```

## Step 2: Training on the MNIST Data
The `trainNNT_Mnist.java` is our training ground. 

```java
//usr/bin/env jbang "$0" "$@" ; exit $?
//SOURCES NeuralNetTutorial.java
//SOURCES TrainingData.java
package com.makariev.examples.ai.neuralnet;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class trainNNT_Mnist {

    public static void main(String[] args) {
        final long startTime = System.currentTimeMillis();

        // Example: Assuming the data has 784 inputs, 64 hidden neurons, and 10 output
        final NeuralNetTutorial.Net myNet = new NeuralNetTutorial.Net(Arrays.asList(784, 64, 10));

        final TrainingData trainData = TrainingData.mnistTrainData();

        // Train
        for (int epoch = 0; epoch < 5; epoch++) {

            trainData.trainLine((inputVals, targetVals) -> {

                double[] input = inputVals;
                for (int n = 0; n < input.length; n++) {
                    // normalization
                    // Scale the pixel values to the range [0,1]
                    input[n] = input[n] / 255;
                }

                double[] target = new double[10];
                target[(int) targetVals[0]] = 1;  // One-hot encoding

                final List<Double> inputValues = new ArrayList<>();
                for (int i = 0; i < input.length; i++) {
                    inputValues.add(input[i]);
                }

                // Train the MLP with the current sample
                // Get new input data and feed it forward:
                myNet.feedForward(inputValues);

                final List<Double> targetValues = new ArrayList<>();
                for (int i = 0; i < target.length; i++) {
                    targetValues.add(target[i]);
                }

                // Train the net what the outputs should have been:
                myNet.backProp(targetValues);

                return true;
            });
            
            // Test and Calculate Accuracy
            trainData.testPredictChunk(
                    10_000,
                    "Epoch: %d, ".formatted(epoch),
                    (inputVals, targetVals) -> {
                        final List<Double> inputValues = new ArrayList<>();
                        for (int n = 0; n < inputVals.length; n++) {
                            // normalization
                            // Scale the pixel values to the range [0,1]
                            inputValues.add(inputVals[n] / 255);
                        }
                        myNet.feedForward(inputValues);
                        final double[] predictions = myNet.getResults().stream().mapToDouble(Double::doubleValue).toArray();
                        final int predictedLabel = TrainingData.getMaxIndex(predictions);
                        return predictedLabel == (int) targetVals[0];
                    }
            );
        }

        System.out.println();

        trainData.testPredictChunk(10_000, (inputVals, targetVals) -> {
            final List<Double> inputValues = new ArrayList<>();
            for (int n = 0; n < inputVals.length; n++) {
                // normalization
                // Scale the pixel values to the range [0,1]
                inputValues.add(inputVals[n] / 255);
            }
            myNet.feedForward(inputValues);
            final double[] predictions = myNet.getResults().stream().mapToDouble(Double::doubleValue).toArray();
            final int predictedLabel = TrainingData.getMaxIndex(predictions);
            return predictedLabel == (int) targetVals[0];
        });

        System.out.println("\nexecution time: " + (System.currentTimeMillis() - startTime) + "ms\n");
    }

}
```


Here we load the MNIST training data, normalize it by scaling pixel values to the range [0,1], and then feed it into our network. We use the [backpropagation algorithm](https://www.makariev.com/blog/crafting-neural-network-in-one-file-java-with-jbang/#backpropagation) to adjust the weights of the network, using the mean squared error as our loss function to gauge performance.

Training is done over multiple epochs, with each pass the network learning a bit more about the handwritten digits it sees. We also implement a mechanism to track and reduce the error over time.

## Step 3: Testing and Validating the Network
After training, we don't just want a network that memorizes the digits; we want one that generalizes well to new, unseen data. This is where our testing phase comes in. We test the network's accuracy against a set of data it hasn't seen before, making predictions and comparing them to the true values.

## Seeing the Results in Action
As we run our training script, we print out the network's accuracy at the end of each epoch, observing as it (hopefully) increases with each pass. The excitement comes in watching the network's predictions improve, transforming from random guesses to confident, accurate classification.

### using **Random Initialization** with [hyperbolic tangent activation function](https://www.makariev.com/blog/crafting-neural-network-in-one-file-java-with-jbang/#hyperbolic-tangent-tanh-activation-function) 
```bash
jbang trainNNT_Mnist.java 
[jbang] Building jar for trainNNT_Mnist.java...
finished initialization
Epoch: 0, Accuracy: 63.04% 
Epoch: 1, Accuracy: 75.37% 
Epoch: 2, Accuracy: 79.64% 
Epoch: 3, Accuracy: 81.57% 
Epoch: 4, Accuracy: 82.38% 

Accuracy: 82.38% 

execution time: 145985ms
```

### using **He Initialization** with [hyperbolic tangent activation function](https://www.makariev.com/blog/crafting-neural-network-in-one-file-java-with-jbang/#hyperbolic-tangent-tanh-activation-function) 
```bash
jbang trainNNT_Mnist.java 
[jbang] Building jar for trainNNT_Mnist.java...
finished initialization
Epoch: 0, Accuracy: 82.02% 
Epoch: 1, Accuracy: 87.44% 
Epoch: 2, Accuracy: 88.69% 
Epoch: 3, Accuracy: 89.21% 
Epoch: 4, Accuracy: 89.81% 

Accuracy: 89.81% 

execution time: 145787ms
```

### using **He Initialization** with [Leaky ReLU](https://www.makariev.com/blog/crafting-neural-network-in-one-file-java-with-jbang/#leaky-relu-activation-function) 
```bash
jbang trainNNT_Mnist.java 
[jbang] Building jar for trainNNT_Mnist.java...
finished initialization
Epoch: 0, Accuracy: 81.50% 
Epoch: 1, Accuracy: 89.11% 
Epoch: 2, Accuracy: 90.78% 
Epoch: 3, Accuracy: 91.88% 
Epoch: 4, Accuracy: 92.33% 

Accuracy: 92.33% 

execution time: 147053ms
```
As you can see by the results above, the Neural Network is quite sensitive to both initialization and activation function. 

# Conclusion
This foray into neural networks with Java and JBang takes you through the process of building, training, and validating a neural network for a classic machine learning task. It showcases the strength of Java in handling complex tasks like neural network computation, while also demonstrating how with tools like JBang, we can streamline the process, making it accessible and manageable.

As you dive into the code, experiment, and tweak the parameters, you'll gain a deeper understanding of neural networks' inner workings and the beauty of machine learning. The MNIST dataset offers a fantastic playground for beginners and seasoned practitioners alike, providing immediate visual feedback and a benchmark that's stood the test of time.

So grab your favorite Java IDE, install JBang, and get ready to embark on an exciting journey into the world of neural networks and handwritten digit classification.

---

[![Coffee Time!](/assets/img/blog/mnist-coffee-1.jpg)](/assets/img/blog/mnist-coffee-1.jpg)

Happy coding!



