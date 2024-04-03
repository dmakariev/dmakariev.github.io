---
title: "Fashion MNIST in CSV"
tags: [ai, mnist, neural network, popular]
thumbnail-img: "/assets/img/blog/fashion-mnist-coffee.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
mermaid: false
mathjax: false
---

The `Fashion-MNIST` is a dataset of Zalando's article images. It serves as a direct drop-in replacement for the original [MNIST dataset](https://www.makariev.com/blog/exploring-the-classic-MNIST-Benchmark-for-machine-learning-models/) for benchmarking machine learning algorithms. This dataset includes 60,000 training examples and 10,000 test examples, each being a 28x28 grayscale image associated with a label from 10 classes.

* toc
{:toc}

## Description of `Fashion-MNIST`
The classes in the `Fashion-MNIST` dataset represent different types of clothing and fashion items. These include:

| Label | Description |
|-------|-------------|
| 0     | T-shirt/top |
| 1     | Trouser     |
| 2     | Pullover    |
| 3     | Dress       |
| 4     | Coat        |
| 5     | Sandal      |
| 6     | Shirt       |
| 7     | Sneaker     |
| 8     | Bag         |
| 9     | Ankle boot  |


Here's an example of how the data looks

[![Coffee Time!](/assets/img/blog/fashion-mnist-sprite-15x15.jpg)](/assets/img/blog/fashion-mnist-sprite-15x15.jpg)

`Fashion-MNIST` was created with the intention of providing a more challenging benchmark dataset for machine learning and computer vision algorithms, as the original [MNIST dataset](https://www.makariev.com/blog/exploring-the-classic-MNIST-Benchmark-for-machine-learning-models/) (comprising hand-written digits) was considered too easy. The dataset aims to reflect real-world scenarios better and provide a more difficult challenge in image classification tasks.

## Where to Download the Fashion-MNIST Dataset
* Access the MNIST dataset at its origin [https://github.com/zalandoresearch/fashion-mnist/tree/master/data/fashion](https://github.com/zalandoresearch/fashion-mnist/tree/master/data/fashion)
* For those who prefer working with tabular data, a CSV version of the Fashion MNIST dataset is available here [dataset-MNIST-fashion.zip](https://github.com/dmakariev/examples/blob/main/artificial-intelligence/neural-network-compare/dataset/dataset-MNIST-fashion.zip)

## Fashion MNIST in CSV
Here's the zip file [dataset-MNIST-fashion.zip](https://github.com/dmakariev/examples/blob/main/artificial-intelligence/neural-network-compare/dataset/dataset-MNIST-fashion.zip)

inside it there are two csv files 
* `mnist_fashion_train.csv` (60,000 images)
* `mnist_fashion_test.csv` (10,000 images)

The format is:

**label, 1x1, 1x2, 1x3, ... 28x27, 28x28**
where `i`x`j` is the pixel in the `i`-th row and `j`-th column.
the label is a number between 0 and 9 

Here is the converter 

```java

import java.io.FileInputStream;
import java.io.FileWriter;
import java.io.IOException;

public class MNISTConverter {

    public static void convert(String imgFileName, String labelFileName, String outFileName, int n) throws IOException {
        try (FileInputStream f = new FileInputStream(imgFileName); 
             FileWriter o = new FileWriter(outFileName); 
             FileInputStream l = new FileInputStream(labelFileName)) {

            final StringBuilder firstLine = new StringBuilder("label,");
            for (int i = 1; i < 29; i++) {
                for (int j = 1; j < 29; j++) {
                    firstLine.append(i).append("x").append(j);
                    if (i != 28 || j != 28) {
                        firstLine.append(",");
                    }
                }
            }
            o.write(firstLine.toString() + "\n");

            byte[] bytes = new byte[16];
            f.read(bytes, 0, 16); // skip the first 16 bytes
            l.read(bytes, 0, 8);  // skip the first 8 bytes

            for (int i = 0; i < n; i++) {
                StringBuilder image = new StringBuilder();
                image.append(l.read() & 0xFF); // read label

                for (int j = 0; j < 28 * 28; j++) {
                    image.append(",").append(f.read() & 0xFF); // read image bytes
                }

                o.write(image.toString() + "\n");
            }
        }
    }

    public static void main(String[] args) throws IOException {
        convert("train-images-idx3-ubyte", "train-labels-idx1-ubyte", "mnist_fashion_train.csv", 60000);
        convert("t10k-images-idx3-ubyte", "t10k-labels-idx1-ubyte", "mnist_fashion_test.csv", 10000);
    }
}

```

if you have the uncompressed files (`train-images-idx3-ubyte`, `train-labels-idx1-ubyte`, `t10k-images-idx3-ubyte`, `t10k-labels-idx1-ubyte`) in the same directory as the script, you could execute it with jbang 
```bash
jbang MNISTConverter.java
```

and the  `mnist_fashion_train.csv` and `mnist_fashion_test.csv` would be generated. 

The conversion is inspired by the one from Joseph Redmon [https://pjreddie.com/projects/mnist-in-csv/](https://pjreddie.com/projects/mnist-in-csv/)


## Usage and Benchmarks

Here is a link to a nice comparison of different implementations agains `MNIST` and `Fashion-MNIST `: [http://fashion-mnist.s3-website.eu-central-1.amazonaws.com/](http://fashion-mnist.s3-website.eu-central-1.amazonaws.com/)

a nice article by the Fashion-MNIST creator : [https://hanxiao.io/2018/09/28/Fashion-MNIST-Year-In-Review/](https://hanxiao.io/2018/09/28/Fashion-MNIST-Year-In-Review/)

# Conclusion

In conclusion, the `Fashion-MNIST` dataset serves as an invaluable resource for both beginners and seasoned practitioners in the field of machine learning. Offering a more complex and realistic challenge than its predecessor, the original `MNIST`, it pushes the boundaries of image classification techniques. Whether you are conducting academic research, developing commercial applications, or just exploring the fascinating world of machine learning, `Fashion-MNIST` in its **CSV** form provides a perfect starting point for your endeavors. 

---

[![Coffee Time!](/assets/img/blog/fashion-mnist-coffee.jpg)](/assets/img/blog/fashion-mnist-coffee.jpg)

Happy coding!
