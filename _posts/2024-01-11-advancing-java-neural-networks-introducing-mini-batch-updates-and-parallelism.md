---
title: "Advancing Java Neural Networks: Introducing Mini-Batch Updates and Parallelism"
tags: [ai, jbang, java, neural network, mnist]
thumbnail-img: "/assets/img/blog/mnist-coffee-5a.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
mermaid: true
mathjax: true
---

In the evolving landscape of machine learning, efficiency and scalability are paramount. Java developers leveraging neural networks have long sought ways to speed up the training process and manage resource utilization more effectively. The transformation from `SimpleMLP` to `SimpleMLPBatch` represents a significant leap in achieving these goals by introducing mini-batch updates and parallel execution with Java virtual threads.

In this blog post, we will build upon our previous [“Enhancing Java Neural Networks: Boosting MNIST Digit Classification with Apache Commons Math”](https://www.makariev.com/blog/enhancing-java-neural-networks-boosting-MNIST-digit-classification-with-apache-commons-math/) and use the MNIST Dataset described in [“Exploring the Classic MNIST: A Benchmark for Machine Learning Models”](https://www.makariev.com/blog/exploring-the-classic-MNIST-Benchmark-for-machine-learning-models/). 

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

# From Stochastic to Mini-Batch Gradient Descent
The original `SimpleMLP` model implemented a stochastic gradient descent approach, updating weights after each training example. While effective for smaller datasets, this method showed its limitations when it came to larger-scale problems. The `SimpleMLPBatch` modification embraces the concept of mini-batch gradient descent, where updates are made after processing a batch of training examples, offering a balance between the efficiency of batch gradient descent and the robustness of stochastic approaches.

By integrating mini-batch updates, `SimpleMLPBatch` improves convergence, allowing for more stable and faster training over batches of data rather than individual data points. This is particularly advantageous when training on massive datasets, as it provides a middle ground for computational efficiency and training stability.

# Implementing a Learning Rate Scheduler 
A constant learning rate can be suboptimal, leading either to slow convergence or overshooting the minima in the loss landscape. `SimpleMLPBatch` and `trainMLP_batch_Mnist` introduce a learning rate scheduler, a dynamic adjustment strategy that fine-tunes the learning rate during training. This allows the network to make larger updates to the weights when the error gradient is steep and smaller updates when approaching a minimum, facilitating a faster and more precise convergence.


# Leveraging Java Virtual Threads for Parallelism
One of the most exciting features of `trainMLP_batch_Mnist` is its utilization of Java virtual threads. This takes advantage of the lightweight threads introduced in recent Java versions for parallel execution of training operations. The concurrency model offered by virtual threads efficiently handles numerous tasks with minimal overhead, making them ideal for parallelizing batch processing.

This parallel execution ensures that each mini-batch update can be processed independently and simultaneously, substantially reducing the training time. It also allows the neural network to scale with the hardware, utilizing multiple cores and threads, which is particularly beneficial for training on multicore processors.

# Implementation Details 
### `SimpleMLPBatch` with Mini-Batch Gradient Descent
```java
//usr/bin/env jbang "$0" "$@" ; exit $?
//DEPS org.apache.commons:commons-math3:3.6.1
//SOURCES ActivationFunction.java
package com.makariev.examples.ai.neuralnet;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import org.apache.commons.math3.analysis.UnivariateFunction;
import org.apache.commons.math3.linear.ArrayRealVector;
import org.apache.commons.math3.linear.MatrixUtils;
import org.apache.commons.math3.linear.RealMatrix;
import org.apache.commons.math3.linear.RealVector;

public class SimpleMLPBatch {

    private final RealMatrix[] weights;
    private final RealMatrix[] biases;
    private final Random random = new Random();

    private static final ActivationFunction ACTIVATION_FUNCTION = ActivationFunction.leakyReLU();
    //private static final ActivationFunction ACTIVATION_FUNCTION = ActivationFunction.ReLU();
    //private final static ActivationFunction ACTIVATION_FUNCTION = ActivationFunction.tanh();
    //private final static ActivationFunction ACTIVATION_FUNCTION = ActivationFunction.sigmoid();

    private static final UnivariateFunction FUNCTION = ACTIVATION_FUNCTION::function;
    private static final UnivariateFunction FUNCTION_DERIVATIVE = ACTIVATION_FUNCTION::functionDerivative;

    public SimpleMLPBatch(int... layerSizes) {
        final long startTime = System.currentTimeMillis();
        this.weights = new RealMatrix[layerSizes.length - 1];
        this.biases = new RealMatrix[layerSizes.length - 1];

        for (int i = 0; i < layerSizes.length - 1; i++) {
            weights[i] = MatrixUtils.createRealMatrix(layerSizes[i + 1], layerSizes[i]);
            biases[i] = MatrixUtils.createColumnRealMatrix(new double[layerSizes[i + 1]]);
            double stddev = Math.sqrt(2.0 / layerSizes[i]); // Standard deviation for He initialization
            for (int row = 0; row < layerSizes[i + 1]; row++) {
                for (int col = 0; col < layerSizes[i]; col++) {
                    weights[i].setEntry(row, col, random.nextGaussian() * stddev);
                }
                biases[i].setEntry(row, 0, 0.0);  // Biases can be initialized to 0
            }
        }
        System.out.printf("finished initialization in %dms\n", (System.currentTimeMillis() - startTime));
    }

    public double[] predict(double[] input) {
        return feedforward(new ArrayRealVector(input)).toArray();
    }

    private RealVector feedforward(RealVector input) {
        RealVector a = input;
        for (int i = 0; i < weights.length; i++) {
            a = weights[i].operate(a).add(biases[i].getColumnVector(0));
            a.mapToSelf(FUNCTION);
        }
        return a;
    }

    public void trainBatch(List<double[]> inputBatch, List<double[]> targetBatch, double learningRate) {
        if (inputBatch.size() != targetBatch.size()) {
            throw new IllegalStateException("inputBatch and targetBatch should have the same size");
        }
        if (inputBatch.isEmpty()) {
            throw new IllegalStateException("inputBatch and targetBatch should not be empty");
        }
        RealMatrix[] weightGradientsSum = new RealMatrix[weights.length];
        RealMatrix[] biasGradientsSum = new RealMatrix[biases.length];

        for (int i = 0; i < weights.length; i++) {
            weightGradientsSum[i] = MatrixUtils.createRealMatrix(weights[i].getRowDimension(), weights[i].getColumnDimension());
            biasGradientsSum[i] = MatrixUtils.createColumnRealMatrix(new double[biases[i].getRowDimension()]);
        }

        for (int n = 0; n < inputBatch.size(); n++) {
            RealVector input = new ArrayRealVector(inputBatch.get(n));
            RealVector target = new ArrayRealVector(targetBatch.get(n));
            Pair<RealMatrix[], RealMatrix[]> gradients = backprop(input, target);
            for (int i = 0; i < weights.length; i++) {
                weightGradientsSum[i] = weightGradientsSum[i].add(gradients.first()[i]);
                biasGradientsSum[i] = biasGradientsSum[i].add(gradients.second()[i]);
            }
        }

        for (int i = 0; i < weights.length; i++) {
            RealMatrix avgWeightGradient = weightGradientsSum[i].scalarMultiply(1.0 / inputBatch.size());
            RealMatrix avgBiasGradient = biasGradientsSum[i].scalarMultiply(1.0 / inputBatch.size());

            // Update weights and biases
            weights[i] = weights[i].subtract(avgWeightGradient.scalarMultiply(learningRate));
            biases[i] = biases[i].subtract(avgBiasGradient.scalarMultiply(learningRate));
        }
    }

    private static record Pair<F, S>(F first, S second) {

    }

    private Pair<RealMatrix[], RealMatrix[]> backprop(RealVector input, RealVector target) {
        List<RealVector> activations = new ArrayList<>();
        activations.add(input);

        List<RealVector> zs = new ArrayList<>();
        RealVector a = input;

        // Forward pass
        for (int i = 0; i < weights.length; i++) {
            RealVector z = weights[i].operate(a).add(biases[i].getColumnVector(0));
            zs.add(z);
            a = z.map(FUNCTION);
            activations.add(a);
        }

        // Backward pass
        RealMatrix[] weightGradients = new RealMatrix[weights.length];
        RealMatrix[] biasGradients = new RealMatrix[biases.length];

        RealVector delta = activations.get(activations.size() - 1).subtract(target);
        for (int i = weights.length - 1; i >= 0; i--) {
            RealMatrix weightGradient = delta.outerProduct(activations.get(i));

            weightGradients[i] = weightGradient;
            biasGradients[i] = MatrixUtils.createColumnRealMatrix(delta.toArray());

            if (i > 0) {
                RealVector sp = zs.get(i - 1).map(FUNCTION_DERIVATIVE);
                delta = weights[i].transpose().operate(delta).ebeMultiply(sp);
            }
        }

        return new Pair<>(weightGradients, biasGradients);
    }
}

```

### `trainMLP_batch_Mnist` with Learning Rate Scheduler and Virtual Threads
```java
//usr/bin/env jbang "$0" "$@" ; exit $?
//SOURCES SimpleMLPBatch.java
//SOURCES TrainingData.java
package com.makariev.examples.ai.neuralnet;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class trainMLP_batch_Mnist {

    public static void main(String[] args) {
        final long startTime = System.currentTimeMillis();

        final TrainingData trainData = TrainingData.mnistTrainData();

        // Example: Assuming the data has 784 inputs, 64 hidden neurons, and 10 output
        final SimpleMLPBatch myMLP = new SimpleMLPBatch(784, 64, 10);

        double initialLearningRate = 0.03;
        double decayRate = 0.95;  // e.g., reduce learning rate by 5%
        int decayStep = 2;  // e.g., reduce learning rate every 2 epochs

        // Train
        for (int epoch = 0; epoch < 5; epoch++) {
            if (epoch % decayStep == 0 && epoch != 0) {
                initialLearningRate *= decayRate;
            }
            final double learningRate = initialLearningRate;

            final ExecutorService executorService = Executors.newVirtualThreadPerTaskExecutor();

            trainData.trainChunk(5, false, (inputVals, targetVals) -> {

                executorService.submit(() -> {
                    final List<double[]> inputBatch = new ArrayList<>();
                    final List<double[]> targetBatch = new ArrayList<>();

                    for (int i = 0; i < inputVals.size(); i++) {
                        double[] input = inputVals.get(i);
                        for (int n = 0; n < input.length; n++) {
                            // normalization
                            // Scale the pixel values to the range [0,1]
                            input[n] = input[n] / 255;
                        }
                        inputBatch.add(input);

                        double[] target = new double[10];
                        target[(int) targetVals.get(i)[0]] = 1.0;  // One-hot encoding
                        targetBatch.add(target);
                    }

                    // Train the MLP with the current sample
                    myMLP.trainBatch(inputBatch, targetBatch, learningRate);
                });
            });

            // Wait for all threads to finish processing the current mini-batch
            try {
                executorService.shutdown();
                executorService.awaitTermination(Long.MAX_VALUE, TimeUnit.NANOSECONDS);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            // Test and Calculate Accuracy
            trainData.testPredictChunk(
                    10_000,
                    "Epoch %d, Learning Rate: %.4f ".formatted(epoch, learningRate),
                    (inputVals, targetVals) -> {

                        double[] target = new double[10];
                        target[(int) targetVals[0]] = 1.0;  // One-hot encoding

                        for (int i = 0; i < inputVals.length; i++) {
                            inputVals[i] = inputVals[i] / 255;
                        }

                        final double[] predictions = myMLP.predict(inputVals);
                        final int predictedLabel = TrainingData.getMaxIndex(predictions);
                        final int targetLabel = TrainingData.getMaxIndex(target);
                        return predictedLabel == targetLabel;
                    }
            );

        }

        System.out.println();

        trainData.testPredictChunk(10_000, (inputVals, targetVals) -> {
            double[] target = new double[10];
            target[(int) targetVals[0]] = 1.0;  // One-hot encoding

            for (int i = 0; i < inputVals.length; i++) {
                inputVals[i] = inputVals[i] / 255;
            }

            final double[] predictions = myMLP.predict(inputVals);
            final int predictedLabel = TrainingData.getMaxIndex(predictions);
            final int targetLabel = TrainingData.getMaxIndex(target);
            return predictedLabel == targetLabel;
        });

        System.out.println("\nexecution time: " + (System.currentTimeMillis() - startTime) + "ms\n");
    }

}

```

# Seeing the Results in Action
As we run our training script `trainMLP_batch_Mnist`, we print out the network's accuracy at the end of each epoch, observing as it (hopefully) increases with each pass. The excitement comes in watching the network's predictions improve, transforming from random guesses to confident, accurate classification.

Using **He Initialization** with [Leaky ReLU](https://www.makariev.com/blog/crafting-neural-network-in-one-file-java-with-jbang/#leaky-relu-activation-function) with Mini-Batch Gradient Descent, Learning Rate Scheduler and Virtual Threads
```bash
jbang trainMLP_batch_Mnist.java
finished initialization in 17ms
Epoch 0, Learning Rate: 0.0300 Accuracy: 94.65% 
Epoch 1, Learning Rate: 0.0300 Accuracy: 95.80% 
Epoch 2, Learning Rate: 0.0285 Accuracy: 96.20% 
Epoch 3, Learning Rate: 0.0285 Accuracy: 96.49% 
Epoch 4, Learning Rate: 0.0271 Accuracy: 96.70% 

Accuracy: 96.70% 

execution time: 10391ms
```

Here are the results from the 'SimpleMLP' class using **He Initialization** with [Leaky ReLU](https://www.makariev.com/blog/crafting-neural-network-in-one-file-java-with-jbang/#leaky-relu-activation-function) for details you could read ["Enhancing Java Neural Networks: Boosting MNIST Digit Classification with Apache Commons Math"](https://www.makariev.com/blog/enhancing-java-neural-networks-boosting-MNIST-digit-classification-with-apache-commons-math/)
```bash
jbang trainMLP_Mnist.java
[jbang] Building jar for trainMLP_Mnist.java...
finished initialization in 17ms
Epoch: 0, Learning Rate: 0.001000, Accuracy: 92.53% 
Epoch: 1, Learning Rate: 0.001000, Accuracy: 94.37% 
Epoch: 2, Learning Rate: 0.001000, Accuracy: 95.00% 
Epoch: 3, Learning Rate: 0.001000, Accuracy: 95.51% 
Epoch: 4, Learning Rate: 0.001000, Accuracy: 95.75% 

Accuracy: 95.75% 

execution time: 52415ms
```

Here are the results from the `NeuralNetTutorial` Class using exactly the same **He Initialization** with [Leaky ReLU](https://www.makariev.com/blog/crafting-neural-network-in-one-file-java-with-jbang/#leaky-relu-activation-function) for details you could read ["A Java Neural Network Tutorial: Classifying MNIST Handwritten Digits"](https://www.makariev.com/blog/java-neural-network-tutorial-classifying-MNIST-handwritten-digits/)
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
As you could see by the results above, the `SimpleMLPBatch` is 5x times faster then `SimpleMLP` and about 14x times faster then the `NeuralNetTutorial` implementation.

# Achieving `98.51%` accuracy with `SimpleMLPBatch` 
With the following setup of the `SimpleMLPBatch` 784 inputs, 400 hidden neurons, and 10 output and running the training for 30 epochs we've achieved `98.51%` accuracy
```bash
jbang trainMLP_batch_Mnist.java
finished initialization in 36ms
Epoch 0, Learning Rate: 0.0300 Accuracy: 96.28% 
Epoch 1, Learning Rate: 0.0300 Accuracy: 96.91% 
Epoch 2, Learning Rate: 0.0285 Accuracy: 97.47% 
Epoch 3, Learning Rate: 0.0285 Accuracy: 97.73% 
Epoch 4, Learning Rate: 0.0271 Accuracy: 97.80% 
Epoch 5, Learning Rate: 0.0271 Accuracy: 97.98% 
Epoch 6, Learning Rate: 0.0257 Accuracy: 97.81% 
Epoch 7, Learning Rate: 0.0257 Accuracy: 97.94% 
Epoch 8, Learning Rate: 0.0244 Accuracy: 98.09% 
Epoch 9, Learning Rate: 0.0244 Accuracy: 98.06% 
Epoch 10, Learning Rate: 0.0232 Accuracy: 98.16% 
Epoch 11, Learning Rate: 0.0232 Accuracy: 98.14% 
Epoch 12, Learning Rate: 0.0221 Accuracy: 98.18% 
Epoch 13, Learning Rate: 0.0221 Accuracy: 98.21% 
Epoch 14, Learning Rate: 0.0210 Accuracy: 98.21% 
Epoch 15, Learning Rate: 0.0210 Accuracy: 98.28% 
Epoch 16, Learning Rate: 0.0199 Accuracy: 98.27% 
Epoch 17, Learning Rate: 0.0199 Accuracy: 98.37% 
Epoch 18, Learning Rate: 0.0189 Accuracy: 98.37% 
Epoch 19, Learning Rate: 0.0189 Accuracy: 98.34% 
Epoch 20, Learning Rate: 0.0180 Accuracy: 98.37% 
Epoch 21, Learning Rate: 0.0180 Accuracy: 98.39% 
Epoch 22, Learning Rate: 0.0171 Accuracy: 98.40% 
Epoch 23, Learning Rate: 0.0171 Accuracy: 98.40% 
Epoch 24, Learning Rate: 0.0162 Accuracy: 98.44% 
Epoch 25, Learning Rate: 0.0162 Accuracy: 98.45% 
Epoch 26, Learning Rate: 0.0154 Accuracy: 98.47% 
Epoch 27, Learning Rate: 0.0154 Accuracy: 98.50% 
Epoch 28, Learning Rate: 0.0146 Accuracy: 98.45% 
Epoch 29, Learning Rate: 0.0146 Accuracy: 98.51% 

Accuracy: 98.51% 

execution time: 311667ms
```
if you want to play with the pre-trained model it is stored in `model/mlp_batch_mnist-400-30x.zip`
and you could run it against the test MNIST dataset like this 
```bash
jbang loadMLP_batch_Mnist.java 
[jbang] Building jar for loadMLP_batch_Mnist.java...
Loading model/mlp_batch_mnist-400-30x.zip
finished loading in 14ms

Accuracy: 98.51% 

execution time: 1816ms
```

# Conclusion
The update from `SimpleMLP` to `SimpleMLPBatch` significantly improves Java-based neural network training by accelerating and stabilizing the process with mini-batch gradient descent, enhancing the handling of large datasets. The integration of a learning rate scheduler makes the training adaptable, promoting more efficient learning and ensuring steady progression towards optimal solutions. Moreover, leveraging Java virtual threads introduces parallelism, dramatically boosting scalability and reducing training times to meet the demands of high-performance applications. These improvements underscore Java's capability for practical, in-system implementation of complex, AI-driven algorithms.

---

I encourage you to explore the code, tinker with the updated `SimpleMLPBatch`, and experience firsthand the remarkable improvements in training efficiency and performance. Whether it's to sharpen your existing skills or to foster new ones, now is the time to harness these breakthroughs and see what you can achieve with the simple, but powerful code at your disposal. 

---

[![Coffee Time!](/assets/img/blog/mnist-coffee-5a.jpg)](/assets/img/blog/mnist-coffee-5a.jpg)

Happy coding!



