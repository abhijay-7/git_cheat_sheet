# TensorFlow 2.19 Intermediate Cheatsheet

## Keras Functional API

### Basic Functional Model
```python
from tensorflow.keras import Input, Model
from tensorflow.keras.layers import Dense, Dropout, Concatenate

# Define inputs
inputs = Input(shape=(784,))

# Build the model graph
x = Dense(128, activation='relu')(inputs)
x = Dropout(0.2)(x)
x = Dense(64, activation='relu')(x)
outputs = Dense(10, activation='softmax')(x)

# Create model
model = Model(inputs=inputs, outputs=outputs)
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
```

### Multi-Input and Multi-Output Models
```python
# Multiple inputs
image_input = Input(shape=(224, 224, 3), name='image')
text_input = Input(shape=(100,), name='text')

# Process each input
image_features = Conv2D(32, 3, activation='relu')(image_input)
image_features = GlobalAveragePooling2D()(image_features)
image_features = Dense(64, activation='relu')(image_features)

text_features = Dense(64, activation='relu')(text_input)
text_features = Dense(32, activation='relu')(text_features)

# Combine inputs
combined = Concatenate()([image_features, text_features])
combined = Dense(64, activation='relu')(combined)

# Multiple outputs
classification_output = Dense(10, activation='softmax', name='classification')(combined)
regression_output = Dense(1, name='regression')(combined)

# Create model
model = Model(
    inputs=[image_input, text_input],
    outputs=[classification_output, regression_output]
)

# Compile with different losses for each output
model.compile(
    optimizer='adam',
    loss={
        'classification': 'sparse_categorical_crossentropy',
        'regression': 'mse'
    },
    loss_weights={
        'classification': 1.0,
        'regression': 0.5
    },
    metrics={
        'classification': ['accuracy'],
        'regression': ['mae']
    }
)
```

## Convolutional Neural Networks (CNNs)

### Basic CNN Architecture
```python
from tensorflow.keras.layers import Conv2D, MaxPooling2D, GlobalAveragePooling2D, BatchNormalization

def create_cnn(input_shape=(224, 224, 3), num_classes=10):
    inputs = Input(shape=input_shape)
    
    # First convolutional block
    x = Conv2D(32, (3, 3), activation='relu', padding='same')(inputs)
    x = BatchNormalization()(x)
    x = Conv2D(32, (3, 3), activation='relu', padding='same')(x)
    x = MaxPooling2D((2, 2))(x)
    x = Dropout(0.25)(x)
    
    # Second convolutional block
    x = Conv2D(64, (3, 3), activation='relu', padding='same')(x)
    x = BatchNormalization()(x)
    x = Conv2D(64, (3, 3), activation='relu', padding='same')(x)
    x = MaxPooling2D((2, 2))(x)
    x = Dropout(0.25)(x)
    
    # Third convolutional block
    x = Conv2D(128, (3, 3), activation='relu', padding='same')(x)
    x = BatchNormalization()(x)
    x = Conv2D(128, (3, 3), activation='relu', padding='same')(x)
    x = GlobalAveragePooling2D()(x)
    x = Dropout(0.5)(x)
    
    # Classification head
    outputs = Dense(num_classes, activation='softmax')(x)
    
    model = Model(inputs, outputs)
    return model

# Create and compile model
model = create_cnn(input_shape=(28, 28, 1), num_classes=10)
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
```

### Advanced CNN Layers
```python
from tensorflow.keras.layers import (
    Conv2D, DepthwiseConv2D, SeparableConv2D, Conv2DTranspose,
    MaxPooling2D, AveragePooling2D, GlobalMaxPooling2D,
    UpSampling2D, ZeroPadding2D, Cropping2D
)

# Different convolution types
conv_standard = Conv2D(64, 3, activation='relu', padding='same')
conv_depthwise = DepthwiseConv2D(3, activation='relu', padding='same')
conv_separable = SeparableConv2D(64, 3, activation='relu', padding='same')
conv_transpose = Conv2DTranspose(32, 3, strides=2, padding='same')

# Different pooling layers
max_pool = MaxPooling2D(pool_size=(2, 2), strides=2)
avg_pool = AveragePooling2D(pool_size=(2, 2), strides=2)
global_max_pool = GlobalMaxPooling2D()
global_avg_pool = GlobalAveragePooling2D()

# Upsampling and padding
upsample = UpSampling2D(size=(2, 2))
zero_pad = ZeroPadding2D(padding=1)
crop = Cropping2D(cropping=((1, 1), (1, 1)))
```

## Recurrent Neural Networks (RNNs)

### LSTM and GRU Networks
```python
from tensorflow.keras.layers import LSTM, GRU, SimpleRNN, Bidirectional, TimeDistributed

def create_rnn_model(sequence_length=100, vocab_size=10000, embedding_dim=128):
    inputs = Input(shape=(sequence_length,))
    
    # Embedding layer
    x = Embedding(vocab_size, embedding_dim)(inputs)
    
    # RNN layers
    x = LSTM(128, return_sequences=True, dropout=0.2, recurrent_dropout=0.2)(x)
    x = LSTM(64, dropout=0.2, recurrent_dropout=0.2)(x)
    
    # Dense layers
    x = Dense(64, activation='relu')(x)
    x = Dropout(0.5)(x)
    outputs = Dense(1, activation='sigmoid')(x)
    
    model = Model(inputs, outputs)
    return model

# Bidirectional RNN
def create_bidirectional_rnn():
    inputs = Input(shape=(100, 64))
    
    # Bidirectional LSTM
    x = Bidirectional(LSTM(64, return_sequences=True))(inputs)
    x = Bidirectional(LSTM(32))(x)
    
    outputs = Dense(10, activation='softmax')(x)
    model = Model(inputs, outputs)
    return model

# Many-to-many sequence model
def create_seq2seq_model(input_length=20, output_length=30, features=128):
    # Encoder
    encoder_inputs = Input(shape=(input_length, features))
    encoder = LSTM(256, return_state=True)
    encoder_outputs, state_h, state_c = encoder(encoder_inputs)
    encoder_states = [state_h, state_c]
    
    # Decoder
    decoder_inputs = Input(shape=(output_length, features))
    decoder_lstm = LSTM(256, return_sequences=True, return_state=True)
    decoder_outputs, _, _ = decoder_lstm(decoder_inputs, initial_state=encoder_states)
    decoder_dense = Dense(features, activation='softmax')
    decoder_outputs = decoder_dense(decoder_outputs)
    
    model = Model([encoder_inputs, decoder_inputs], decoder_outputs)
    return model
```

### Text Processing with RNNs
```python
from tensorflow.keras.layers import Embedding
from tensorflow.keras.preprocessing.text import Tokenizer
from tensorflow.keras.preprocessing.sequence import pad_sequences

# Text preprocessing
def preprocess_text(texts, max_features=10000, maxlen=100):
    tokenizer = Tokenizer(num_words=max_features)
    tokenizer.fit_on_texts(texts)
    sequences = tokenizer.texts_to_sequences(texts)
    return pad_sequences(sequences, maxlen=maxlen), tokenizer

# Text classification model
def create_text_classifier(max_features=10000, maxlen=100, embedding_dim=128):
    inputs = Input(shape=(maxlen,))
    
    # Embedding layer
    x = Embedding(max_features, embedding_dim)(inputs)
    
    # RNN layers
    x = LSTM(64, dropout=0.5, recurrent_dropout=0.5)(x)
    
    # Classification head
    outputs = Dense(1, activation='sigmoid')(x)
    
    model = Model(inputs, outputs)
    model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
    return model
```

## Transfer Learning and Fine-tuning

### Using Pre-trained Models
```python
from tensorflow.keras.applications import (
    ResNet50, VGG16, InceptionV3, MobileNetV2, 
    EfficientNetB0, DenseNet121
)
from tensorflow.keras.applications.resnet50 import preprocess_input, decode_predictions

# Load pre-trained model
base_model = ResNet50(
    weights='imagenet',  # Pre-trained weights
    include_top=False,   # Exclude final classification layer
    input_shape=(224, 224, 3)
)

# Freeze base model weights
base_model.trainable = False

# Add custom classification head
inputs = Input(shape=(224, 224, 3))
x = preprocess_input(inputs)  # Preprocess for ResNet50
x = base_model(x, training=False)
x = GlobalAveragePooling2D()(x)
x = Dropout(0.2)(x)
outputs = Dense(10, activation='softmax')(x)

model = Model(inputs, outputs)
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
```

### Fine-tuning Strategy
```python
# Fine-tuning with different learning rates
def fine_tune_model(base_model, num_classes, learning_rate=0.0001):
    # Unfreeze top layers
    base_model.trainable = True
    
    # Fine-tune from this layer onwards
    fine_tune_at = 100
    
    # Freeze all layers before fine_tune_at
    for layer in base_model.layers[:fine_tune_at]:
        layer.trainable = False
    
    # Build model
    inputs = Input(shape=(224, 224, 3))
    x = base_model(inputs, training=False)
    x = GlobalAveragePooling2D()(x)
    x = Dropout(0.2)(x)
    outputs = Dense(num_classes, activation='softmax')(x)
    
    model = Model(inputs, outputs)
    
    # Use lower learning rate for fine-tuning
    model.compile(
        optimizer=tf.keras.optimizers.Adam(learning_rate/10),
        loss='sparse_categorical_crossentropy',
        metrics=['accuracy']
    )
    
    return model
```

## Custom Datasets with tf.data

### Creating Custom Datasets
```python
# From image files
def create_image_dataset(image_paths, labels, image_size=(224, 224)):
    def preprocess_image(image_path, label):
        image = tf.io.read_file(image_path)
        image = tf.image.decode_image(image, channels=3)
        image = tf.image.resize(image, image_size)
        image = tf.cast(image, tf.float32) / 255.0
        return image, label
    
    dataset = tf.data.Dataset.from_tensor_slices((image_paths, labels))
    dataset = dataset.map(preprocess_image, num_parallel_calls=tf.data.AUTOTUNE)
    return dataset

# From directories
train_dataset = tf.keras.utils.image_dataset_from_directory(
    'data/train',
    validation_split=0.2,
    subset="training",
    seed=123,
    image_size=(224, 224),
    batch_size=32
)

validation_dataset = tf.keras.utils.image_dataset_from_directory(
    'data/train',
    validation_split=0.2,
    subset="validation",
    seed=123,
    image_size=(224, 224),
    batch_size=32
)
```

### Advanced Dataset Operations
```python
# Data pipeline optimization
def configure_for_performance(dataset, buffer_size=1000):
    dataset = dataset.cache()  # Cache dataset in memory
    dataset = dataset.shuffle(buffer_size=buffer_size)
    dataset = dataset.batch(32)
    dataset = dataset.prefetch(buffer_size=tf.data.AUTOTUNE)
    return dataset

# Data augmentation pipeline
def augment_image(image, label):
    image = tf.image.random_flip_left_right(image)
    image = tf.image.random_brightness(image, 0.2)
    image = tf.image.random_contrast(image, 0.8, 1.2)
    image = tf.image.random_saturation(image, 0.8, 1.2)
    return image, label

# Apply augmentation
train_dataset = train_dataset.map(
    augment_image, 
    num_parallel_calls=tf.data.AUTOTUNE
)

# Mixed dataset operations
dataset = dataset.filter(lambda x, y: tf.reduce_sum(x) > 0)  # Filter
dataset = dataset.take(1000)  # Take first 1000 samples
dataset = dataset.skip(100)   # Skip first 100 samples
dataset = dataset.repeat(2)   # Repeat dataset 2 times
```

## Data Augmentation and Preprocessing

### Image Augmentation
```python
from tensorflow.keras.layers import (
    RandomFlip, RandomRotation, RandomZoom, RandomCrop,
    RandomBrightness, RandomContrast, RandomTranslation
)

# Create augmentation layers
data_augmentation = tf.keras.Sequential([
    RandomFlip("horizontal_and_vertical"),
    RandomRotation(0.2),
    RandomZoom(0.2),
    RandomBrightness(0.2),
    RandomContrast(0.2),
    RandomTranslation(height_factor=0.1, width_factor=0.1),
])

# Apply to model
def create_model_with_augmentation(input_shape, num_classes):
    inputs = Input(shape=input_shape)
    
    # Apply augmentation only during training
    x = data_augmentation(inputs)
    
    # Model architecture
    x = Conv2D(32, 3, activation='relu')(x)
    x = MaxPooling2D()(x)
    x = Conv2D(64, 3, activation='relu')(x)
    x = GlobalAveragePooling2D()(x)
    outputs = Dense(num_classes, activation='softmax')(x)
    
    return Model(inputs, outputs)

# Manual augmentation functions
def manual_augmentation(image, label):
    # Random rotation
    image = tf.image.rot90(image, k=tf.random.uniform([], 0, 4, dtype=tf.int32))
    
    # Random brightness
    image = tf.image.random_brightness(image, max_delta=0.1)
    
    # Random contrast
    image = tf.image.random_contrast(image, lower=0.9, upper=1.1)
    
    # Random flip
    image = tf.image.random_flip_left_right(image)
    image = tf.image.random_flip_up_down(image)
    
    return image, label
```

### Preprocessing Layers
```python
from tensorflow.keras.layers import (
    Normalization, StringLookup, IntegerLookup,
    TextVectorization, Discretization, CategoryEncoding
)

# Normalization layer
normalizer = Normalization()
normalizer.adapt(train_features)  # Learn statistics

# Text preprocessing
text_vectorizer = TextVectorization(
    max_tokens=10000,
    output_sequence_length=50,
    output_mode='int'
)
text_vectorizer.adapt(text_data)

# Categorical preprocessing
string_lookup = StringLookup(vocabulary=['cat', 'dog', 'bird'], output_mode='int')
integer_lookup = IntegerLookup(vocabulary=[1, 2, 3, 4, 5], output_mode='one_hot')

# Use in model
def create_preprocessing_model():
    # Numerical features
    numerical_input = Input(shape=(10,), name='numerical')
    numerical_normalized = normalizer(numerical_input)
    
    # Text features
    text_input = Input(shape=(), name='text', dtype=tf.string)
    text_vectorized = text_vectorizer(text_input)
    text_embedded = Embedding(10000, 64)(text_vectorized)
    text_features = GlobalAveragePooling1D()(text_embedded)
    
    # Categorical features
    categorical_input = Input(shape=(), name='categorical', dtype=tf.string)
    categorical_encoded = string_lookup(categorical_input)
    categorical_embedded = Embedding(len(string_lookup.get_vocabulary()), 8)(categorical_encoded)
    categorical_features = Flatten()(categorical_embedded)
    
    # Combine all features
    combined = Concatenate()([numerical_normalized, text_features, categorical_features])
    outputs = Dense(1, activation='sigmoid')(combined)
    
    return Model([numerical_input, text_input, categorical_input], outputs)
```

## Regularization and Callbacks

### Regularization Techniques
```python
from tensorflow.keras.layers import Dropout, BatchNormalization
from tensorflow.keras.regularizers import l1, l2, l1_l2

# Dropout layers
dropout_25 = Dropout(0.25)
dropout_50 = Dropout(0.5)

# Batch normalization
batch_norm = BatchNormalization()

# Weight regularization
l1_reg = Dense(64, activation='relu', kernel_regularizer=l1(0.01))
l2_reg = Dense(64, activation='relu', kernel_regularizer=l2(0.01))
l1_l2_reg = Dense(64, activation='relu', kernel_regularizer=l1_l2(l1=0.01, l2=0.01))

# Activity regularization
activity_reg = Dense(64, activation='relu', activity_regularizer=l2(0.01))

# Model with regularization
def create_regularized_model(input_shape, num_classes):
    inputs = Input(shape=input_shape)
    
    x = Dense(128, activation='relu', kernel_regularizer=l2(0.001))(inputs)
    x = BatchNormalization()(x)
    x = Dropout(0.5)(x)
    
    x = Dense(64, activation='relu', kernel_regularizer=l2(0.001))(x)
    x = BatchNormalization()(x)
    x = Dropout(0.3)(x)
    
    outputs = Dense(num_classes, activation='softmax')(x)
    
    return Model(inputs, outputs)
```

### Callbacks
```python
from tensorflow.keras.callbacks import (
    ModelCheckpoint, EarlyStopping, ReduceLROnPlateau,
    LearningRateScheduler, TensorBoard, CSVLogger
)

# Model checkpoint
checkpoint = ModelCheckpoint(
    filepath='best_model.h5',
    monitor='val_accuracy',
    save_best_only=True,
    save_weights_only=False,
    mode='max',
    verbose=1
)

# Early stopping
early_stopping = EarlyStopping(
    monitor='val_loss',
    patience=5,
    restore_best_weights=True,
    verbose=1
)

# Learning rate reduction
reduce_lr = ReduceLROnPlateau(
    monitor='val_loss',
    factor=0.5,
    patience=3,
    min_lr=1e-7,
    verbose=1
)

# Learning rate scheduling
def scheduler(epoch, lr):
    if epoch < 10:
        return lr
    else:
        return lr * tf.math.exp(-0.1)

lr_scheduler = LearningRateScheduler(scheduler)

# TensorBoard logging
tensorboard = TensorBoard(
    log_dir='./logs',
    histogram_freq=1,
    write_graph=True,
    write_images=True
)

# CSV logging
csv_logger = CSVLogger('training.log')

# Custom callback
class CustomCallback(tf.keras.callbacks.Callback):
    def on_epoch_end(self, epoch, logs=None):
        if logs.get('accuracy') > 0.95:
            print(f"\nReached 95% accuracy at epoch {epoch}, stopping training!")
            self.model.stop_training = True

# Use callbacks in training
history = model.fit(
    train_dataset,
    epochs=50,
    validation_data=val_dataset,
    callbacks=[
        checkpoint,
        early_stopping,
        reduce_lr,
        tensorboard,
        csv_logger,
        CustomCallback()
    ]
)
```

## TensorBoard Visualization

```python
# Setup TensorBoard logging
import datetime

log_dir = "logs/fit/" + datetime.datetime.now().strftime("%Y%m%d-%H%M%S")
tensorboard_callback = tf.keras.callbacks.TensorBoard(log_dir=log_dir, histogram_freq=1)

# Custom scalar logging
def log_custom_scalars(epoch, logs):
    with tf.summary.create_file_writer(log_dir).as_default():
        tf.summary.scalar('custom_metric', logs['accuracy'] * logs['val_accuracy'], step=epoch)

class CustomTensorBoard(tf.keras.callbacks.Callback):
    def __init__(self, log_dir):
        self.log_dir = log_dir
        
    def on_epoch_end(self, epoch, logs=None):
        with tf.summary.create_file_writer(self.log_dir).as_default():
            tf.summary.scalar('learning_rate', self.model.optimizer.learning_rate, step=epoch)
            tf.summary.scalar('custom_metric', logs.get('accuracy', 0) ** 2, step=epoch)

# Use custom TensorBoard callback
custom_tb = CustomTensorBoard(log_dir)

# Launch TensorBoard (in terminal)
# tensorboard --logdir logs/fit
```

## Model Subclassing

```python
class CustomModel(tf.keras.Model):
    def __init__(self, num_classes=10):
        super(CustomModel, self).__init__()
        self.conv1 = Conv2D(32, 3, activation='relu')
        self.conv2 = Conv2D(64, 3, activation='relu')
        self.pool = MaxPooling2D()
        self.dropout1 = Dropout(0.25)
        self.flatten = Flatten()
        self.fc1 = Dense(128, activation='relu')
        self.dropout2 = Dropout(0.5)
        self.fc2 = Dense(num_classes, activation='softmax')
        
    def call(self, inputs, training=None):
        x = self.conv1(inputs)
        x = self.pool(x)
        x = self.conv2(x)
        x = self.pool(x)
        x = self.dropout1(x, training=training)
        x = self.flatten(x)
        x = self.fc1(x)
        x = self.dropout2(x, training=training)
        return self.fc2(x)
    
    def get_config(self):
        return {"num_classes": self.fc2.units}

# Create and use custom model
model = CustomModel(num_classes=10)
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])

# Build model by calling it once
model.build(input_shape=(None, 28, 28, 1))
model.summary()
```

## Custom Loss Functions and Metrics

```python
# Custom loss functions
def focal_loss(alpha=0.25, gamma=2.0):
    def focal_loss_fixed(y_true, y_pred):
        epsilon = tf.keras.backend.epsilon()
        y_pred = tf.clip_by_value(y_pred, epsilon, 1. - epsilon)
        
        alpha_t = y_true * alpha + (tf.ones_like(y_true) - y_true) * (1 - alpha)
        p_t = y_true * y_pred + (tf.ones_like(y_true) - y_true) * (tf.ones_like(y_pred) - y_pred)
        
        focal_loss = - alpha_t * tf.pow((tf.ones_like(p_t) - p_t), gamma) * tf.math.log(p_t)
        return tf.reduce_mean(focal_loss)
    return focal_loss_fixed

def dice_loss(y_true, y_pred, smooth=1):
    intersection = tf.reduce_sum(y_true * y_pred)
    union = tf.reduce_sum(y_true) + tf.reduce_sum(y_pred)
    dice = (2. * intersection + smooth) / (union + smooth)
    return 1 - dice

# Custom metrics
class F1Score(tf.keras.metrics.Metric):
    def __init__(self, name='f1_score', **kwargs):
        super(F1Score, self).__init__(name=name, **kwargs)
        self.precision = tf.keras.metrics.Precision()
        self.recall = tf.keras.metrics.Recall()
        
    def update_state(self, y_true, y_pred, sample_weight=None):
        self.precision.update_state(y_true, y_pred, sample_weight)
        self.recall.update_state(y_true, y_pred, sample_weight)
        
    def result(self):
        p = self.precision.result()
        r = self.recall.result()
        return 2 * ((p * r) / (p + r + tf.keras.backend.epsilon()))
    
    def reset_state(self):
        self.precision.reset_state()
        self.recall.reset_state()

# Use custom loss and metrics
model.compile(
    optimizer='adam',
    loss=focal_loss(alpha=0.25, gamma=2.0),
    metrics=['accuracy', F1Score()]
)
```