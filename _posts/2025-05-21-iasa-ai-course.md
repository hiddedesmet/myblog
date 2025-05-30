---
layout: post
title: A practical guide to Machine Learning for image classification
description: An overview of a typical machine learning workflow for image classification, covering problem definition, ML type, tooling, data preparation, model training, and deployment using TensorFlow, Flask, and Docker.
date: 2025-05-21 10:00:00 +0300
author: hidde
image: '/images/machinelearning.png' 
tags: [AI, Machine Learning, Image Classification, IASA]
featured: false
toc: true
---

I recently started the AI Architecture course by Zach Gardner from IASA Global, which aims to equip professionals with the knowledge to implement AI effectively within businesses. The course delves into AI principles, frameworks, MLOps, governance, and best practices, emphasizing a business-first approach to security, scalability, and performance in AI architectures. Inspired by this, I wanted to share a practical walkthrough of a typical machine learning project.

# A practical guide to Machine Learning for image classification

Many real-world problems involve classifying items based on visual features. Identifying these categories is important for various applications. Often, these classification tasks are performed manually, a process that can be slow and prone to inconsistencies. Machine learning (ML) offers an alternative, enabling computers to learn from examples and automate this process, leading to increased speed, efficiency, and reliability. This post will walk through a common machine learning project focused on image classification, explaining each step from defining the problem to deploying a solution. We'll see how ML can be used to analyze images and assign them to predefined categories.

Computers can analyze vast numbers of images quickly without fatigue or distraction. For instance, manually sorting hundreds or thousands of images can lead to errors over time. An ML model, once trained, can maintain consistent performance, ensuring uniform quality in classification tasks.

## Defining the problem: image classification

The main challenge in image classification is to analyze an image and determine which predefined category it belongs to. For example, we might need to classify images into:

*   Object type 1
*   Object type 2
*   Object type 3

Each category typically possesses distinct visual characteristics. Differentiating these by eye can be difficult, especially when dealing with a large volume of images or when the visual differences are subtle.

**Figure 1: Basic image classification process**
```mermaid
flowchart TD
    A[Input Images] --> B{Classification}
    B --> C[Object type 1]
    B --> D[Object type 2]
    B --> E[Object type 3]
```

Using images for classification is often more efficient than manual inspection. Consider an automated system where items pass by a camera; the camera captures images, and a computer instantly sorts them. This not only saves time but also minimizes errors that might occur due to human fatigue or haste.

## Choosing the right approach: supervised learning and CNNs

To tackle image classification, we typically turn to **supervised learning**. In this approach, we provide the computer with a large dataset of examples where the correct answer (the category label) is already known. The model learns to recognize patterns from these labeled examples.

**Figure 2: Supervised learning with CNNs**
```mermaid
graph LR
    Input[Input: Labeled images] --> Model[Convolutional Neural Network]
    Model --> Output[Output: Category label]
```

Supervised learning with CNNs is like teaching a child with flashcards: "This image is object type 1," "This one is object type 2," and so on. CNNs are effective because they can automatically learn hierarchical features from images, such as edges, textures, and complex shapes, which are important for accurate classification.

## Essential tools for the workflow

A machine learning project relies on a set of tools to manage the various stages of development. Here are some common categories and examples:

*   **ML frameworks**: These provide the building blocks for creating and training models.
    *   TensorFlow (often with Keras API)
    *   PyTorch
*   **Data labeling tools**: Used to annotate images with their correct categories.
    *   LabelImg
    *   Roboflow
    *   CVAT (Computer Vision Annotation Tool)
*   **Experiment tracking**: Helps monitor and compare different model versions and training runs.
    *   MLflow
    *   TensorBoard (especially for TensorFlow)
    *   Weights & Biases

The typical workflow involving these tools can be visualized as follows:

**Figure 3: Data preparation workflow**
```mermaid
flowchart LR
    A[Data collection] --> B[Labeling tool]
    B --> C[ML framework]
    C --> D[Experiment tracking]
```

First, we collect the necessary images. Then, using a labeling tool, we assign the correct category to each image. With the labeled dataset, we use an ML framework like TensorFlow or PyTorch to design and train our CNN model. Throughout this process, experiment tracking tools log metrics, parameters, and artifacts, allowing us to reproduce results and understand what works best. These tools are like a scientist's lab notebook, helpful for systematic improvement.

## Preparing the data: collection, splitting, and augmentation

The quality and quantity of data are very important in machine learning. For our image classification model to learn effectively, it needs to see a diverse set of examples.

Key steps in data preparation include:

1.  **Collect diverse, labeled images**: Gather a wide variety of images for each category, ensuring they represent different conditions (lighting, angles, backgrounds) the model might encounter in the real world.
2.  **Split data**: Divide the dataset into three distinct subsets:
    *   **Training set (e.g., 70%)**: Used to train the model.
    *   **Validation set (e.g., 15%)**: Used to tune model parameters and monitor for overfitting during training.
    *   **Test set (e.g., 15%)**: Used for a final, unbiased evaluation of the trained model's performance on unseen data.
3.  **Use data augmentation**: Artificially increase the size and diversity of the training set by applying random transformations to existing images (e.g., rotations, flips, brightness adjustments). This helps the model become more robust and generalize better to new, unseen images.

Here's an example of how you can set up data augmentation using `ImageDataGenerator` in TensorFlow/Keras:

```python
from tensorflow.keras.preprocessing.image import ImageDataGenerator

# Create an ImageDataGenerator instance with desired augmentations
datagen = ImageDataGenerator(
    rotation_range=20,      # Randomly rotate images by up to 20 degrees
    width_shift_range=0.2,  # Randomly shift images horizontally by up to 20% of the width
    height_shift_range=0.2, # Randomly shift images vertically by up to 20% of the height
    shear_range=0.2,        # Apply shear transformations
    zoom_range=0.2,         # Randomly zoom into images
    horizontal_flip=True,   # Randomly flip images horizontally
    fill_mode='nearest'     # Strategy for filling newly created pixels
)

# Example: Applying it to a training data generator
# train_generator = datagen.flow_from_directory(
#     'path/to/train_data',
#     target_size=(224, 224),
#     batch_size=32,
#     class_mode='categorical'
# )
```

**Figure 3: Data preparation workflow**
```mermaid
flowchart TD
    A[Raw Images] --> B[Labeling]
    B --> C[Dataset split]
    C --> D1[Training set]
    C --> D2[Validation set]
    C --> D3[Test set]
```

Splitting the data is important to ensure the model isn't just "memorizing" the training examples but is actually learning to generalize. Data augmentation acts as a regularizer, preventing the model from becoming too specialized to the training data and improving its performance on real-world data.

## Building and training the model

With the data prepared, the next step is to define the model architecture and train it.

*   **Choose a CNN architecture**: Select a CNN architecture suitable for image classification. This could be a custom-built network or a pre-trained model using **transfer learning**. Transfer learning is a powerful technique where a model developed for a task (e.g., classifying a large dataset like ImageNet) is reused as the starting point for a model on a second task. This approach can significantly reduce training time and improve performance, especially when your dataset is relatively small, as the model has already learned general features from the larger dataset.
*   **Example architecture**: A simple CNN might consist of:
    *   Input layer (receiving image data)
    *   Convolutional layers (Conv2D) with activation functions (e.g., ReLU)
    *   Pooling layers (MaxPooling) to reduce dimensionality
    *   Flatten layer (to convert 2D feature maps to a 1D vector)
    *   Dense layers (fully connected layers) for classification
    *   Output layer with an activation function (e.g., softmax for multi-class classification)
*   **Compile the model**: Configure the learning process by specifying:
    *   **Optimizer** (e.g., Adam, SGD): Algorithm to update model weights.
    *   **Loss function** (e.g., `categorical_crossentropy` for multi-class): Measures how well the model is performing.
    *   **Metrics** (e.g., `accuracy`): Used to monitor training and testing steps.
*   **Train the model**: Fit the model to the training data, using the validation set to monitor its performance and prevent overfitting.

Here\'s a simplified example of defining and compiling a CNN model using TensorFlow/Keras:

```python
import tensorflow as tf
from tensorflow.keras import layers, models

# Assuming 3 categories and input images of size 224x224x3 (RGB)
model = models.Sequential([
    layers.Input(shape=(224, 224, 3)),
    layers.Conv2D(32, (3, 3), activation='relu'),
    layers.MaxPooling2D((2, 2)),
    layers.Conv2D(64, (3, 3), activation='relu'),
    layers.MaxPooling2D((2, 2)),
    layers.Flatten(),
    layers.Dense(64, activation='relu'),
    layers.Dense(3, activation='softmax') # Output layer for 3 classes
])

model.compile(optimizer='adam',
              loss='categorical_crossentropy',
              metrics=['accuracy'])

# model.fit(training_data, validation_data=validation_data, epochs=N) # Actual training step
```

The model\'s architecture dictates its capacity to learn. Convolutional layers act as feature extractors, learning to identify patterns like edges and textures. Pooling layers help to make the learned features more robust to variations in object scale and position. Dense layers then use these high-level features to make the final classification. The training process iteratively adjusts the model\'s weights to minimize the chosen loss function.

## Saving your trained model

Once the model is trained to a satisfactory performance level, it\'s important to save its learned parameters (weights) and architecture. This allows you to reuse the model later for predictions without needing to retrain it from scratch.

In TensorFlow/Keras, saving a model is straightforward:

```python
# Assume 'model' is your trained Keras model
model.save('image_classifier_model')
```

This command saves the entire model (architecture, weights, and training configuration) to a directory named `image_classifier_model`. This saved model can then be loaded into other applications or deployed to a server. It’s like saving your progress in a complex task, ensuring your efforts are preserved for future use.

## Making the model accessible: serving with Flask

To make your trained image classification model usable by other applications or users, you can expose it as a web API. Flask is a lightweight Python web framework that is excellent for this purpose.

Here’s a conceptual example of a Flask app that loads the saved TensorFlow model and provides a `/predict` endpoint:

```python
from flask import Flask, request, jsonify
import tensorflow as tf
from PIL import Image # Pillow library for image manipulation
import numpy as np

app = Flask(__name__)

# Load the saved model
model = tf.keras.models.load_model('image_classifier_model')
# Define the class names (ensure order matches model output)
CLASSES = ['Object type 1', 'Object type 2', 'Object type 3']

def preprocess_image(image_file):
    img = Image.open(image_file.stream).convert('RGB') # Ensure 3 channels
    img = img.resize((224, 224)) # Resize to model's expected input size
    img_array = np.array(img) / 255.0 # Normalize pixel values
    img_array = np.expand_dims(img_array, axis=0) # Add batch dimension
    return img_array

@app.route('/predict', methods=['POST'])
def predict():
    if 'file' not in request.files:
        return jsonify({'error': 'No file part'}), 400
    file = request.files['file']
    if file.filename == '':
        return jsonify({'error': 'No selected file'}), 400

    try:
        img_array = preprocess_image(file)
        prediction = model.predict(img_array)
        class_idx = np.argmax(prediction, axis=1)[0]
        return jsonify({'class': CLASSES[class_idx], 'confidence': float(prediction[0][class_idx])})
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

This Flask application creates an endpoint that accepts an image file, preprocesses it to match the model's input requirements, gets a prediction from the loaded TensorFlow model, and returns the predicted class as a JSON response. This makes the model accessible over the network.

## Ensuring portability: dockerizing the application

To ensure that your Flask application (and the ML model it serves) runs consistently across different environments (development, testing, production), containerization with Docker is highly recommended. Docker packages the application and all its dependencies into a standardized unit called a container.

Here’s an example `Dockerfile` for the Flask application:

```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.10-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Copy requirements.txt and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Make port 5000 available to the world outside this container
EXPOSE 5000

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

You would create a `requirements.txt` file in the same directory as your `Dockerfile` and `app.py`. For this project, it would look like this:

```txt
flask
tensorflow
pillow
numpy
```

This `Dockerfile` defines the steps to build a Docker image. It starts from a base Python image, copies the application code (including `app.py`, the `image_classifier_model` directory, and a `requirements.txt` file), installs dependencies, exposes the port Flask is running on, and specifies the command to run the application. This container can then be deployed on any system with Docker installed, resolving the "it works on my machine" problem.

## The complete workflow

The overall workflow, from a user or system providing an image to receiving a classification, can be summarized with the following diagram:

**Figure 4: Complete image classification and serving workflow**
```mermaid
flowchart TD
    A["User Uploads Image / Image from System"] --> B["Flask API (via HTTP)"]
    B --> C["Docker Container hosting Flask App & TensorFlow Model"]
    C -- Preprocesses Image --> D[TensorFlow Model Inference]
    D -- Returns Prediction --> C
    C -- Sends JSON Response --> A["Prediction (Category) returned to User/System"]
```

A user or an automated system sends an image to the Flask API. The API, running inside a Docker container, receives the image. The Flask application preprocesses the image and feeds it to the TensorFlow model for inference. The model returns a prediction, which the Flask app then formats as a JSON response and sends back to the requester.

## Conclusion and key takeaways

This post highlighted a common and effective machine learning workflow for image classification. The key stages include:

1.  **Problem definition**: Clearly understanding the classification task.
2.  **Data management**: Collecting, labeling, splitting, and augmenting image data.
3.  **Model development**: Choosing an appropriate architecture (like a CNN), training it with frameworks such as TensorFlow, and saving the trained model.
4.  **Deployment**: Serving the model via a web API using Flask.
5.  **Packaging**: Containerizing the application with Docker for portability and scalability.

This structured approach can be adapted for a wide array of applications, from identifying different types of flora and fauna to detecting defects in manufacturing or recognizing landmarks in photographs. By following these steps and leveraging the right tools, you can build AI systems capable of understanding and interpreting visual information.
