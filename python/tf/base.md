# TensorFlow 2.19 Beginner Cheatsheet

## Installation and Setup

```bash
# Install latest stable TensorFlow (CPU version)
pip install tensorflow

# Install TensorFlow with GPU support (Linux/WSL2)
pip install tensorflow[and-cuda]

# Install TensorFlow CPU-only version
pip install tensorflow-cpu

# Install nightly build (preview)
pip install tf-nightly
```

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np
import matplotlib.pyplot as plt

# Check TensorFlow version
print(f"TensorFlow version: {tf.__version__}")

# Check GPU availability
print(f"GPUs available: {tf.config.list_physical_devices('GPU')}")
```

## Tensor Fundamentals

### Creating Tensors
```python
# From Python lists/arrays
tensor_1d = tf.constant([1, 2, 3, 4])
tensor_2d = tf.constant([[1, 2], [3, 4]])
tensor_3d = tf.constant([[[1, 2], [3, 4]], [[5, 6], [7, 8]]])

# Initialized tensors
zeros = tf.zeros([3, 4])              # 3x4 tensor of zeros
ones = tf.ones([2, 3, 4])             # 2x3x4 tensor of ones
fill = tf.fill([2, 3], 7)             # 2x3 tensor filled with 7
identity = tf.eye(3)                  # 3x3 identity matrix

# Random tensors
random_normal = tf.random.normal([3, 4], mean=0, stddev=1)
random_uniform = tf.random.uniform([3, 4], minval=0, maxval=1)
random_int = tf.random.uniform([3, 4], minval=0, maxval=10, dtype=tf.int32)

# From NumPy arrays
numpy_array = np.array([[1, 2], [3, 4]])
tf_tensor = tf.constant(numpy_array)
```

### Variables
```python
# Create variables (trainable parameters)
var = tf.Variable([1.0, 2.0, 3.0])
weight = tf.Variable(tf.random.normal([3, 2]))
bias = tf.Variable(tf.zeros([2]))

# Update variables
var.assign([4.0, 5.0, 6.0])          # Assign new values
var.assign_add([1.0, 1.0, 1.0])      # Add to existing values
var.assign_sub([0.5, 0.5, 0.5])      # Subtract from existing values

# Variable properties
print(f"Shape: {var.shape}")
print(f"Data type: {var.dtype}")
print(f"Trainable: {var.trainable}")
```

### Tensor Properties and Operations
```python
tensor = tf.constant([[1, 2, 3], [4, 5, 6]])

# Properties
print(f"Shape: {tensor.shape}")           # TensorShape([2, 3])
print(f"Rank (dimensions): {tf.rank(tensor)}")  # 2
print(f"Size (total elements): {tf.size(tensor)}")  # 6
print(f"Data type: {tensor.dtype}")       # <dtype: 'int32'>

# Convert to NumPy
numpy_version = tensor.numpy()

# Reshape operations
reshaped = tf.reshape(tensor, [3, 2])     # Reshape to 3x2
flattened = tf.reshape(tensor, [-1])      # Flatten to 1D
expanded = tf.expand_dims(tensor, axis=0)  # Add dimension at axis 0
squeezed = tf.squeeze(expanded)           # Remove dimensions of size 1
```

## Basic Operations and Math

### Arithmetic Operations
```python
a = tf.constant([1, 2, 3])
b = tf.constant([4, 5, 6])

# Element-wise operations
add = tf.add(a, b)              # or a + b
subtract = tf.subtract(a, b)    # or a - b
multiply = tf.multiply(a, b)    # or a * b
divide = tf.divide(a, b)        # or a / b
power = tf.pow(a, 2)            # or a ** 2

# Mathematical functions
sqrt = tf.sqrt(tf.cast(a, tf.float32))
exp = tf.exp(tf.cast(a, tf.float32))
log = tf.math.log(tf.cast(a, tf.float32))
abs = tf.abs(tf.constant([-1, -2, 3]))

# Reduction operations
matrix = tf.constant([[1, 2, 3], [4, 5, 6]], dtype=tf.float32)
sum_all = tf.reduce_sum(matrix)           # Sum of all elements
sum_axis0 = tf.reduce_sum(matrix, axis=0) # Sum along axis 0
mean = tf.reduce_mean(matrix)             # Mean of all elements
max_val = tf.reduce_max(matrix)           # Maximum value
min_val = tf.reduce_min(matrix)           # Minimum value

# Matrix operations
matrix_a = tf.constant([[1, 2], [3, 4]], dtype=tf.float32)
matrix_b = tf.constant([[5, 6], [7, 8]], dtype=tf.float32)
matmul = tf.matmul(matrix_a, matrix_b)    # Matrix multiplication
transpose = tf.transpose(matrix_a)        # Transpose
```

## Keras Sequential API

### Basic Neural Network
```python
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense, Activation, Dropout

# Create sequential model
model = Sequential([
    Dense(128, activation='relu', input_shape=(784,)),
    Dropout(0.2),
    Dense(64, activation='relu'),
    Dense(10, activation='softmax')
])

# Alternative way to build
model = Sequential()
model.add(Dense(128, activation='relu', input_shape=(784,)))
model.add(Dropout(0.2))
model.add(Dense(64, activation='relu'))
model.add(Dense(10, activation='softmax'))

# Model summary
model.summary()
```

### Activation Functions
```python
# Common activation functions as layers
relu = layers.ReLU()
sigmoid = layers.Activation('sigmoid')
tanh = layers.Activation('tanh')
softmax = layers.Activation('softmax')
leaky_relu = layers.LeakyReLU(alpha=0.1)

# Activation functions in Dense layers
dense_relu = layers.Dense(64, activation='relu')
dense_sigmoid = layers.Dense(1, activation='sigmoid')
dense_softmax = layers.Dense(10, activation='softmax')

# Custom activation
def swish(x):
    return x * tf.sigmoid(x)

custom_activation = layers.Dense(64, activation=swish)
```

## Data Loading and Preprocessing

### Using tf.data.Dataset
```python
# From tensors
features = tf.random.normal([1000, 10])
labels = tf.random.uniform([1000], maxval=2, dtype=tf.int32)
dataset = tf.data.Dataset.from_tensor_slices((features, labels))

# From NumPy arrays
x_train = np.random.random((1000, 32))
y_train = np.random.random((1000, 10))
dataset = tf.data.Dataset.from_tensor_slices((x_train, y_train))

# Dataset operations
dataset = dataset.batch(32)           # Batch data
dataset = dataset.shuffle(1000)       # Shuffle data
dataset = dataset.prefetch(tf.data.AUTOTUNE)  # Prefetch for performance
dataset = dataset.repeat()            # Repeat dataset

# Data preprocessing
def preprocess(x, y):
    x = tf.cast(x, tf.float32) / 255.0  # Normalize to [0, 1]
    return x, y

dataset = dataset.map(preprocess)
```

### Built-in Datasets
```python
# Load MNIST dataset
(x_train, y_train), (x_test, y_test) = tf.keras.datasets.mnist.load_data()

# Normalize pixel values
x_train = x_train.astype('float32') / 255.0
x_test = x_test.astype('float32') / 255.0

# One-hot encode labels
y_train = tf.keras.utils.to_categorical(y_train, 10)
y_test = tf.keras.utils.to_categorical(y_test, 10)

# Other built-in datasets
# tf.keras.datasets.cifar10.load_data()
# tf.keras.datasets.fashion_mnist.load_data()
# tf.keras.datasets.imdb.load_data()
# tf.keras.datasets.reuters.load_data()
```

## Model Compilation and Training

### Compile Model
```python
# Compile with different optimizers
model.compile(
    optimizer='adam',  # or tf.keras.optimizers.Adam(learning_rate=0.001)
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# Common optimizers
adam = tf.keras.optimizers.Adam(learning_rate=0.001)
sgd = tf.keras.optimizers.SGD(learning_rate=0.01, momentum=0.9)
rmsprop = tf.keras.optimizers.RMSprop(learning_rate=0.001)

# Common loss functions
sparse_categorical_crossentropy = 'sparse_categorical_crossentropy'
categorical_crossentropy = 'categorical_crossentropy'
binary_crossentropy = 'binary_crossentropy'
mse = 'mean_squared_error'
mae = 'mean_absolute_error'

# Common metrics
accuracy = 'accuracy'
sparse_accuracy = 'sparse_categorical_accuracy'
precision = tf.keras.metrics.Precision()
recall = tf.keras.metrics.Recall()
f1_score = tf.keras.metrics.F1Score()
```

### Training the Model
```python
# Simple training
history = model.fit(
    x_train, y_train,
    epochs=10,
    batch_size=32,
    validation_data=(x_test, y_test),
    verbose=1
)

# Training with validation split
history = model.fit(
    x_train, y_train,
    epochs=10,
    batch_size=32,
    validation_split=0.2,
    verbose=1
)

# Training with tf.data.Dataset
train_dataset = tf.data.Dataset.from_tensor_slices((x_train, y_train))
train_dataset = train_dataset.batch(32).prefetch(tf.data.AUTOTUNE)

val_dataset = tf.data.Dataset.from_tensor_slices((x_test, y_test))
val_dataset = val_dataset.batch(32).prefetch(tf.data.AUTOTUNE)

history = model.fit(
    train_dataset,
    epochs=10,
    validation_data=val_dataset,
    verbose=1
)
```

### Model Evaluation and Prediction
```python
# Evaluate model
test_loss, test_accuracy = model.evaluate(x_test, y_test, verbose=0)
print(f"Test accuracy: {test_accuracy:.4f}")

# Make predictions
predictions = model.predict(x_test)
predicted_classes = tf.argmax(predictions, axis=1)

# Predict single sample
single_prediction = model.predict(x_test[0:1])
predicted_class = tf.argmax(single_prediction, axis=1)

# Get prediction probabilities
probabilities = tf.nn.softmax(predictions)
confidence = tf.reduce_max(probabilities, axis=1)
```

## Model Saving and Loading

### Save and Load Models
```python
# Save entire model
model.save('my_model.h5')
model.save('my_model')  # SavedModel format (recommended)

# Load model
loaded_model = tf.keras.models.load_model('my_model.h5')
loaded_model = tf.keras.models.load_model('my_model')

# Save/load only weights
model.save_weights('model_weights.h5')
model.load_weights('model_weights.h5')

# Save/load model architecture only
model_json = model.to_json()
with open('model_architecture.json', 'w') as json_file:
    json_file.write(model_json)

# Load architecture
from tensorflow.keras.models import model_from_json
with open('model_architecture.json', 'r') as json_file:
    loaded_model_json = json_file.read()
loaded_model = model_from_json(loaded_model_json)
```

### Checkpoints
```python
# Save checkpoints during training
checkpoint_callback = tf.keras.callbacks.ModelCheckpoint(
    filepath='model_checkpoint.h5',
    save_best_only=True,
    monitor='val_accuracy',
    mode='max',
    verbose=1
)

history = model.fit(
    x_train, y_train,
    epochs=10,
    validation_data=(x_test, y_test),
    callbacks=[checkpoint_callback]
)
```

## Complete Example: MNIST Classification

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np
import matplotlib.pyplot as plt

# Load and preprocess data
(x_train, y_train), (x_test, y_test) = keras.datasets.mnist.load_data()
x_train = x_train.astype('float32') / 255.0
x_test = x_test.astype('float32') / 255.0

# Reshape data for fully connected layers
x_train = x_train.reshape(-1, 28*28)
x_test = x_test.reshape(-1, 28*28)

# Build model
model = keras.Sequential([
    layers.Dense(128, activation='relu', input_shape=(784,)),
    layers.Dropout(0.2),
    layers.Dense(64, activation='relu'),
    layers.Dropout(0.2),
    layers.Dense(10, activation='softmax')
])

# Compile model
model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# Train model
history = model.fit(
    x_train, y_train,
    epochs=5,
    batch_size=32,
    validation_data=(x_test, y_test),
    verbose=1
)

# Evaluate model
test_loss, test_accuracy = model.evaluate(x_test, y_test, verbose=0)
print(f"Test accuracy: {test_accuracy:.4f}")

# Make predictions
predictions = model.predict(x_test[:5])
predicted_classes = tf.argmax(predictions, axis=1)
print(f"Predicted classes: {predicted_classes.numpy()}")
print(f"Actual classes: {y_test[:5]}")
```

## Visualization and Debugging

```python
# Plot training history
def plot_history(history):
    plt.figure(figsize=(12, 4))
    
    plt.subplot(1, 2, 1)
    plt.plot(history.history['loss'], label='Training Loss')
    plt.plot(history.history['val_loss'], label='Validation Loss')
    plt.title('Model Loss')
    plt.xlabel('Epoch')
    plt.ylabel('Loss')
    plt.legend()
    
    plt.subplot(1, 2, 2)
    plt.plot(history.history['accuracy'], label='Training Accuracy')
    plt.plot(history.history['val_accuracy'], label='Validation Accuracy')
    plt.title('Model Accuracy')
    plt.xlabel('Epoch')
    plt.ylabel('Accuracy')
    plt.legend()
    
    plt.tight_layout()
    plt.show()

# Visualize model architecture
tf.keras.utils.plot_model(model, show_shapes=True, show_layer_names=True)

# Check for NaN values
def check_for_nan(tensor, name="tensor"):
    if tf.reduce_any(tf.math.is_nan(tensor)):
        print(f"NaN detected in {name}")
    if tf.reduce_any(tf.math.is_inf(tensor)):
        print(f"Inf detected in {name}")

# Debug model weights
for i, layer in enumerate(model.layers):
    if hasattr(layer, 'weights') and layer.weights:
        for j, weight in enumerate(layer.weights):
            print(f"Layer {i} ({layer.name}) - Weight {j}: {weight.shape}")
            check_for_nan(weight, f"Layer {i} Weight {j}")
```

## Common Troubleshooting

```python
# GPU memory management
gpus = tf.config.experimental.list_physical_devices('GPU')
if gpus:
    try:
        # Enable memory growth
        for gpu in gpus:
            tf.config.experimental.set_memory_growth(gpu, True)
    except RuntimeError as e:
        print(e)

# Set memory limit
if gpus:
    try:
        tf.config.experimental.set_virtual_device_configuration(
            gpus[0],
            [tf.config.experimental.VirtualDeviceConfiguration(memory_limit=1024)]
        )
    except RuntimeError as e:
        print(e)

# Enable mixed precision (for supported GPUs)
policy = tf.keras.mixed_precision.Policy('mixed_float16')
tf.keras.mixed_precision.set_global_policy(policy)

# Reproducible results
tf.random.set_seed(42)
np.random.seed(42)

# Check TensorFlow configuration
print("TensorFlow version:", tf.__version__)
print("Keras version:", tf.keras.__version__)
print("GPU available:", tf.config.list_physical_devices('GPU'))
print("Eager execution:", tf.executing_eagerly())
```