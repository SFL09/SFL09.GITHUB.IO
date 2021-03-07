---
layout: post
title: "Deep Learning - Word Embedding"
description: "Produced word embeddings by using Neural Network."
categories: [DataScience]
tags: [Deep Learning, Python]
redirect_from:
  - /2018/05/01/
---

## Deep Learning


### Word Embedding
- Produced word embeddings by using Neural Network;
- Trained deep neural network autoencoder for feature extraction to processing the data efficiently;
- Compared the performance and code readability of TensorFlow and PyTorch with the CUDA platform.

### TensorFlow
![TF1](https://user-images.githubusercontent.com/76184559/108605851-b5542500-7384-11eb-8504-39ef838ba115.jpg)

![TF2](https://user-images.githubusercontent.com/76184559/108605871-d7e63e00-7384-11eb-8c20-af94da4bcdc4.jpg)

### Autoencoder
![Autoencoder](https://user-images.githubusercontent.com/76184559/108605883-e2083c80-7384-11eb-877a-79f81506d65f.jpg)

### Results
- The `accuracy` is `97.9600%` after `45 epoch.`

```python
   Extracting MNIST_data/train-images-idx3-ubyte.gz
   Extracting MNIST_data/train-labels-idx1-ubyte.gz
   Extracting MNIST_data/t10k-images-idx3-ubyte.gz
   Extracting MNIST_data/t10k-labels-idx1-ubyte.gz
   The accuracy is 0.9796000123023987 after 45 epoch with learning rate 0.001 and batch size 100.
```

- The `final accuracy` is `99.0100%` after `20 epoch.`

```python
Extracting MNIST_data/train-images-idx3-ubyte.gz
Extracting MNIST_data/train-labels-idx1-ubyte.gz
Extracting MNIST_data/t10k-images-idx3-ubyte.gz
Extracting MNIST_data/t10k-labels-idx1-ubyte.gz
The final accuracy on test set is 99.0100%.
```
