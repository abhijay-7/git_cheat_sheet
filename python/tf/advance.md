# TensorFlow 2.19 Advanced Cheatsheet

## Custom Training Loops with GradientTape

### Basic Custom Training Loop
```python
import tensorflow as tf
from tensorflow.keras import optimizers, losses, metrics

class CustomTrainer:
    def __init__(self, model, optimizer, loss_fn):
        self.model = model
        self.optimizer = optimizer
        self.loss_fn = loss_fn
        self.train_loss = metrics.Mean()
        self.train_accuracy = metrics.SparseCategoricalAccuracy()
        self.val_loss = metrics.Mean()
        self.val_accuracy = metrics.SparseCategoricalAccuracy()
    
    @tf.function
    def train_step(self, x, y):
        with tf.GradientTape() as tape:
            predictions = self.model(x, training=True)
            loss = self.loss_fn(y, predictions)
        
        gradients = tape.gradient(loss, self.model.trainable_variables)
        self.optimizer.apply_gradients(zip(gradients, self.model.trainable_variables))
        
        self.train_loss(loss)
        self.train_accuracy(y, predictions)
        
        return loss
    
    @tf.function
    def val_step(self, x, y):
        predictions = self.model(x, training=False)
        loss = self.loss_fn(y, predictions)
        
        self.val_loss(loss)
        self.val_accuracy(y, predictions)
        
        return loss
    
    def train(self, train_dataset, val_dataset, epochs):
        for epoch in range(epochs):
            # Reset metrics
            self.train_loss.reset_states()
            self.train_accuracy.reset_states()
            self.val_loss.reset_states()
            self.val_accuracy.reset_states()
            
            # Training
            for x_batch, y_batch in train_dataset:
                self.train_step(x_batch, y_batch)
            
            # Validation
            for x_batch, y_batch in val_dataset:
                self.val_step(x_batch, y_batch)
            
            print(f'Epoch {epoch + 1}/{epochs}')
            print(f'Loss: {self.train_loss.result():.4f}, '
                  f'Accuracy: {self.train_accuracy.result():.4f}, '
                  f'Val Loss: {self.val_loss.result():.4f}, '
                  f'Val Accuracy: {self.val_accuracy.result():.4f}')

# Usage
model = tf.keras.Sequential([...])
optimizer = optimizers.Adam(learning_rate=0.001)
loss_fn = losses.SparseCategoricalCrossentropy()

trainer = CustomTrainer(model, optimizer, loss_fn)
trainer.train(train_dataset, val_dataset, epochs=10)
```

### Advanced Custom Training with Multiple Losses
```python
class MultiTaskTrainer:
    def __init__(self, model, optimizers, loss_functions, loss_weights):
        self.model = model
        self.optimizers = optimizers
        self.loss_functions = loss_functions
        self.loss_weights = loss_weights
        
    @tf.function
    def train_step(self, inputs, targets):
        with tf.GradientTape() as tape:
            predictions = self.model(inputs, training=True)
            
            # Calculate multiple losses
            total_loss = 0
            losses = {}
            for name, (loss_fn, weight, pred, target) in zip(
                self.loss_functions.keys(),
                [(fn, w, predictions[i], targets[i]) 
                 for i, (fn, w) in enumerate(zip(self.loss_functions.values(), self.loss_weights))]
            ):
                loss = loss_fn(target, pred)
                losses[name] = loss
                total_loss += weight * loss
        
        # Get gradients and apply updates
        gradients = tape.gradient(total_loss, self.model.trainable_variables)
        
        # Apply gradients with different optimizers for different parts
        self.optimizers['main'].apply_gradients(
            zip(gradients, self.model.trainable_variables)
        )
        
        return total_loss, losses

# Gradient accumulation for large batch sizes
class GradientAccumulationTrainer:
    def __init__(self, model, optimizer, loss_fn, accumulate_steps=4):
        self.model = model
        self.optimizer = optimizer
        self.loss_fn = loss_fn
        self.accumulate_steps = accumulate_steps
        
    @tf.function
    def train_step_with_accumulation(self, dataset_iter):
        accumulated_gradients = [
            tf.Variable(tf.zeros_like(var), trainable=False)
            for var in self.model.trainable_variables
        ]
        
        total_loss = 0
        for _ in range(self.accumulate_steps):
            x, y = next(dataset_iter)
            
            with tf.GradientTape() as tape:
                predictions = self.model(x, training=True)
                loss = self.loss_fn(y, predictions) / self.accumulate_steps
            
            gradients = tape.gradient(loss, self.model.trainable_variables)
            
            # Accumulate gradients
            for i, grad in enumerate(gradients):
                accumulated_gradients[i].assign_add(grad)
            
            total_loss += loss
        
        # Apply accumulated gradients
        self.optimizer.apply_gradients(
            zip(accumulated_gradients, self.model.trainable_variables)
        )
        
        # Reset accumulated gradients
        for grad in accumulated_gradients:
            grad.assign(tf.zeros_like(grad))
        
        return total_loss
```

## Distributed Training Strategies

### Multi-GPU Training
```python
# MirroredStrategy for single-machine multi-GPU
strategy = tf.distribute.MirroredStrategy()
print(f'Number of devices: {strategy.num_replicas_in_sync}')

with strategy.scope():
    # Create model within strategy scope
    model = tf.keras.Sequential([
        tf.keras.layers.Dense(128, activation='relu'),
        tf.keras.layers.Dense(10, activation='softmax')
    ])
    
    # Compile model
    model.compile(
        optimizer='adam',
        loss='sparse_categorical_crossentropy',
        metrics=['accuracy']
    )

# Distributed dataset
def make_datasets_unbatched():
    BUFFER_SIZE = 10000
    BATCH_SIZE_PER_REPLICA = 64
    GLOBAL_BATCH_SIZE = BATCH_SIZE_PER_REPLICA * strategy.num_replicas_in_sync
    
    # Create dataset
    train_dataset = tf.data.Dataset.from_tensor_slices((x_train, y_train))
    test_dataset = tf.data.Dataset.from_tensor_slices((x_test, y_test))
    
    train_dataset = train_dataset.shuffle(BUFFER_SIZE).batch(GLOBAL_BATCH_SIZE)
    test_dataset = test_dataset.batch(GLOBAL_BATCH_SIZE)
    
    return train_dataset, test_dataset

train_dist_dataset = strategy.experimental_distribute_dataset(train_dataset)
test_dist_dataset = strategy.experimental_distribute_dataset(test_dataset)

# Custom distributed training loop
with strategy.scope():
    def compute_loss(labels, predictions):
        per_example_loss = tf.keras.losses.sparse_categorical_crossentropy(labels, predictions)
        return tf.nn.compute_average_loss(per_example_loss, global_batch_size=GLOBAL_BATCH_SIZE)
    
    def train_step(inputs):
        features, labels = inputs
        
        with tf.GradientTape() as tape:
            predictions = model(features, training=True)
            loss = compute_loss(labels, predictions)
        
        gradients = tape.gradient(loss, model.trainable_variables)
        optimizer.apply_gradients(zip(gradients, model.trainable_variables))
        
        return loss
    
    @tf.function
    def distributed_train_step(dataset_inputs):
        per_replica_losses = strategy.run(train_step, args=(dataset_inputs,))
        return strategy.reduce(tf.distribute.ReduceOp.SUM, per_replica_losses, axis=None)
```

### Multi-Worker Distributed Training
```python
import json
import os

# Setup cluster configuration
os.environ['TF_CONFIG'] = json.dumps({
    'cluster': {
        'worker': ["localhost:12345", "localhost:23456"]
    },
    'task': {'type': 'worker', 'index': 0}
})

# MultiWorkerMirroredStrategy
strategy = tf.distribute.MultiWorkerMirroredStrategy()

with strategy.scope():
    # Multi-worker compatible model
    multi_worker_model = tf.keras.Sequential([
        tf.keras.layers.Dense(128, activation='relu'),
        tf.keras.layers.Dense(128, activation='relu'),
        tf.keras.layers.Dense(10, activation='softmax')
    ])
    
    multi_worker_model.compile(
        optimizer=tf.keras.optimizers.SGD(learning_rate=0.001),
        loss='sparse_categorical_crossentropy',
        metrics=['accuracy']
    )

# Fault tolerance with ModelCheckpoint
callbacks = [
    tf.keras.callbacks.ModelCheckpoint(
        filepath='/tmp/keras-ckpt-{epoch}',
        save_weights_only=True
    )
]

multi_worker_model.fit(
    train_dataset,
    epochs=3,
    steps_per_epoch=70,
    callbacks=callbacks
)
```

## Mixed Precision Training

### Automatic Mixed Precision
```python
# Enable mixed precision
policy = tf.keras.mixed_precision.Policy('mixed_float16')
tf.keras.mixed_precision.set_global_policy(policy)

print(f'Compute dtype: {policy.compute_dtype}')
print(f'Variable dtype: {policy.variable_dtype}')

# Model with mixed precision
def create_mixed_precision_model():
    inputs = tf.keras.Input(shape=(224, 224, 3))
    
    x = tf.keras.layers.Conv2D(32, 3, activation='relu')(inputs)
    x = tf.keras.layers.Conv2D(32, 3, activation='relu')(x)
    x = tf.keras.layers.GlobalAveragePooling2D()(x)
    x = tf.keras.layers.Dense(64, activation='relu')(x)
    
    # Ensure output layer uses float32
    outputs = tf.keras.layers.Dense(10, activation='softmax', dtype='float32')(x)
    
    return tf.keras.Model(inputs, outputs)

model = create_mixed_precision_model()

# Custom training with loss scaling
optimizer = tf.keras.optimizers.Adam()
optimizer = tf.keras.mixed_precision.LossScaleOptimizer(optimizer)

@tf.function
def train_step_mixed_precision(x, y):
    with tf.GradientTape() as tape:
        predictions = model(x, training=True)
        loss = tf.keras.losses.sparse_categorical_crossentropy(y, predictions)
        
        # Scale loss to prevent underflow
        scaled_loss = optimizer.get_scaled_loss(loss)
    
    scaled_gradients = tape.gradient(scaled_loss, model.trainable_variables)
    gradients = optimizer.get_unscaled_gradients(scaled_gradients)
    
    # Optional: Clip gradients
    gradients = [tf.clip_by_norm(g, 1.0) for g in gradients]
    
    optimizer.apply_gradients(zip(gradients, model.trainable_variables))
    
    return loss

# Compile with mixed precision
model.compile(
    optimizer=optimizer,
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)
```

## Advanced Model Optimization

### Model Quantization
```python
# Post-training quantization
def quantize_model(model_path, representative_dataset):
    # Convert to TensorFlow Lite with quantization
    converter = tf.lite.TFLiteConverter.from_saved_model(model_path)
    
    # Enable quantization
    converter.optimizations = [tf.lite.Optimize.DEFAULT]
    
    # Set representative dataset for full integer quantization
    def representative_data_gen():
        for input_value in representative_dataset.take(100):
            yield [input_value[0]]
    
    converter.representative_dataset = representative_data_gen
    converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
    converter.inference_input_type = tf.int8
    converter.inference_output_type = tf.int8
    
    quantized_model = converter.convert()
    return quantized_model

# Quantization-aware training
def create_qat_model():
    import tensorflow_model_optimization as tfmot
    
    # Define base model
    base_model = tf.keras.Sequential([
        tf.keras.layers.Conv2D(32, 5, activation='relu'),
        tf.keras.layers.MaxPooling2D((2, 2)),
        tf.keras.layers.Conv2D(64, 5, activation='relu'),
        tf.keras.layers.MaxPooling2D((2, 2)),
        tf.keras.layers.Flatten(),
        tf.keras.layers.Dense(1024, activation='relu'),
        tf.keras.layers.Dense(10, activation='softmax')
    ])
    
    # Apply quantization-aware training
    qat_model = tfmot.quantization.keras.quantize_model(base_model)
    
    return qat_model

# Model pruning
def create_pruned_model():
    import tensorflow_model_optimization as tfmot
    
    base_model = create_base_model()
    
    # Define pruning parameters
    pruning_params = {
        'pruning_schedule': tfmot.sparsity.keras.PolynomialDecay(
            initial_sparsity=0.50,
            final_sparsity=0.80,
            begin_step=0,
            end_step=1000
        )
    }
    
    # Apply pruning
    pruned_model = tfmot.sparsity.keras.prune_low_magnitude(base_model, **pruning_params)
    
    return pruned_model
```

### TensorFlow Serving and Deployment

```python
# Save model for TensorFlow Serving
def save_for_serving(model, model_name, version=1):
    export_path = f"./saved_models/{model_name}/{version}"
    
    # Add serving signatures
    @tf.function
    def serve_function(input_tensor):
        predictions = model(input_tensor)
        return {
            'predictions': predictions,
            'probabilities': tf.nn.softmax(predictions)
        }
    
    # Define input signature
    input_signature = [tf.TensorSpec(shape=[None, 224, 224, 3], dtype=tf.float32)]
    concrete_function = serve_function.get_concrete_function(input_signature[0])
    
    # Save with signatures
    tf.saved_model.save(
        model,
        export_path,
        signatures={
            'serving_default': concrete_function,
            'predict': concrete_function
        }
    )
    
    return export_path

# Load and serve model
def load_and_serve(model_path):
    # Load saved model
    loaded_model = tf.saved_model.load(model_path)
    
    # Get serving function
    serving_fn = loaded_model.signatures['serving_default']
    
    # Make prediction
    def predict(input_data):
        return serving_fn(tf.constant(input_data, dtype=tf.float32))
    
    return predict

# TensorFlow Lite conversion
def convert_to_tflite(model_path, optimization=True):
    # Create converter
    converter = tf.lite.TFLiteConverter.from_saved_model(model_path)
    
    if optimization:
        converter.optimizations = [tf.lite.Optimize.DEFAULT]
    
    # Convert model
    tflite_model = converter.convert()
    
    # Save TFLite model
    with open('model.tflite', 'wb') as f:
        f.write(tflite_model)
    
    return tflite_model

# TensorFlow Lite inference
def tflite_inference(tflite_model_path, input_data):
    # Load TFLite model
    interpreter = tf.lite.Interpreter(model_path=tflite_model_path)
    interpreter.allocate_tensors()
    
    # Get input and output tensors
    input_details = interpreter.get_input_details()
    output_details = interpreter.get_output_details()
    
    # Set input tensor
    interpreter.set_tensor(input_details[0]['index'], input_data)
    
    # Run inference
    interpreter.invoke()
    
    # Get output
    output_data = interpreter.get_tensor(output_details[0]['index'])
    return output_data
```

## Graph Mode and tf.function

### Optimizing with tf.function
```python
# Basic tf.function usage
@tf.function
def optimized_train_step(x, y, model, optimizer, loss_fn):
    with tf.GradientTape() as tape:
        predictions = model(x, training=True)
        loss = loss_fn(y, predictions)
    
    gradients = tape.gradient(loss, model.trainable_variables)
    optimizer.apply_gradients(zip(gradients, model.trainable_variables))
    
    return loss

# tf.function with input signatures
@tf.function(input_signature=[
    tf.TensorSpec(shape=[None, 28, 28, 1], dtype=tf.float32),
    tf.TensorSpec(shape=[None], dtype=tf.int32)
])
def signature_train_step(x, y):
    # Training logic here
    pass

# Conditional execution in tf.function
@tf.function
def conditional_train_step(x, y, training=True):
    if training:
        with tf.GradientTape() as tape:
            predictions = model(x, training=True)
            loss = loss_fn(y, predictions)
        
        gradients = tape.gradient(loss, model.trainable_variables)
        optimizer.apply_gradients(zip(gradients, model.trainable_variables))
        return loss
    else:
        predictions = model(x, training=False)
        return loss_fn(y, predictions)

# AutoGraph conversion
@tf.function
def autograph_example(x):
    # Python control flow is converted to TensorFlow ops
    for i in tf.range(10):
        x = x + 1
        if x > 5:
            x = x * 2
    return x

# Debugging tf.function
@tf.function
def debug_function(x):
    # Use tf.print for debugging in graph mode
    tf.print("Input shape:", tf.shape(x))
    
    # Use tf.debugging.assert_* for assertions
    tf.debugging.assert_rank(x, 2, message="Input must be 2D")
    
    result = tf.reduce_sum(x)
    tf.print("Result:", result)
    
    return result
```

## Advanced Computer Vision

### Object Detection with Custom Training
```python
# Simple object detection model
class ObjectDetectionModel(tf.keras.Model):
    def __init__(self, num_classes, num_anchors=9):
        super(ObjectDetectionModel, self).__init__()
        self.num_classes = num_classes
        self.num_anchors = num_anchors
        
        # Backbone
        self.backbone = tf.keras.applications.ResNet50(
            include_top=False,
            weights='imagenet',
            input_shape=(224, 224, 3)
        )
        
        # Detection head
        self.conv1 = tf.keras.layers.Conv2D(256, 3, padding='same', activation='relu')
        self.conv2 = tf.keras.layers.Conv2D(256, 3, padding='same', activation='relu')
        
        # Classification head
        self.cls_head = tf.keras.layers.Conv2D(
            num_anchors * num_classes, 3, padding='same'
        )
        
        # Regression head
        self.reg_head = tf.keras.layers.Conv2D(
            num_anchors * 4, 3, padding='same'
        )
    
    def call(self, inputs, training=None):
        # Extract features
        features = self.backbone(inputs, training=training)
        
        # Detection head
        x = self.conv1(features)
        x = self.conv2(x)
        
        # Predictions
        cls_pred = self.cls_head(x)
        reg_pred = self.reg_head(x)
        
        # Reshape predictions
        batch_size = tf.shape(inputs)[0]
        cls_pred = tf.reshape(cls_pred, [batch_size, -1, self.num_classes])
        reg_pred = tf.reshape(reg_pred, [batch_size, -1, 4])
        
        return cls_pred, reg_pred

# Object detection loss
def detection_loss(y_true_cls, y_true_reg, y_pred_cls, y_pred_reg, alpha=1.0):
    # Classification loss (focal loss)
    cls_loss = tf.keras.losses.sparse_categorical_crossentropy(
        y_true_cls, y_pred_cls, from_logits=True
    )
    
    # Regression loss (smooth L1)
    reg_loss = tf.keras.losses.huber(y_true_reg, y_pred_reg)
    
    # Combine losses
    total_loss = cls_loss + alpha * reg_loss
    return total_loss

# Image segmentation model (U-Net)
def create_unet(input_shape=(256, 256, 3), num_classes=1):
    inputs = tf.keras.Input(shape=input_shape)
    
    # Encoder
    c1 = tf.keras.layers.Conv2D(64, 3, activation='relu', padding='same')(inputs)
    c1 = tf.keras.layers.Conv2D(64, 3, activation='relu', padding='same')(c1)
    p1 = tf.keras.layers.MaxPooling2D((2, 2))(c1)
    
    c2 = tf.keras.layers.Conv2D(128, 3, activation='relu', padding='same')(p1)
    c2 = tf.keras.layers.Conv2D(128, 3, activation='relu', padding='same')(c2)
    p2 = tf.keras.layers.MaxPooling2D((2, 2))(c2)
    
    c3 = tf.keras.layers.Conv2D(256, 3, activation='relu', padding='same')(p2)
    c3 = tf.keras.layers.Conv2D(256, 3, activation='relu', padding='same')(c3)
    p3 = tf.keras.layers.MaxPooling2D((2, 2))(c3)
    
    # Bottleneck
    c4 = tf.keras.layers.Conv2D(512, 3, activation='relu', padding='same')(p3)
    c4 = tf.keras.layers.Conv2D(512, 3, activation='relu', padding='same')(c4)
    
    # Decoder
    u3 = tf.keras.layers.Conv2DTranspose(256, 2, strides=(2, 2), padding='same')(c4)
    u3 = tf.keras.layers.Concatenate()([u3, c3])
    c5 = tf.keras.layers.Conv2D(256, 3, activation='relu', padding='same')(u3)
    c5 = tf.keras.layers.Conv2D(256, 3, activation='relu', padding='same')(c5)
    
    u2 = tf.keras.layers.Conv2DTranspose(128, 2, strides=(2, 2), padding='same')(c5)
    u2 = tf.keras.layers.Concatenate()([u2, c2])
    c6 = tf.keras.layers.Conv2D(128, 3, activation='relu', padding='same')(u2)
    c6 = tf.keras.layers.Conv2D(128, 3, activation='relu', padding='same')(c6)
    
    u1 = tf.keras.layers.Conv2DTranspose(64, 2, strides=(2, 2), padding='same')(c6)
    u1 = tf.keras.layers.Concatenate()([u1, c1])
    c7 = tf.keras.layers.Conv2D(64, 3, activation='relu', padding='same')(u1)
    c7 = tf.keras.layers.Conv2D(64, 3, activation='relu', padding='same')(c7)
    
    outputs = tf.keras.layers.Conv2D(num_classes, 1, activation='sigmoid')(c7)
    
    return tf.keras.Model(inputs, outputs)
```

## Generative Models

### Variational Autoencoder (VAE)
```python
class VAE(tf.keras.Model):
    def __init__(self, latent_dim=2):
        super(VAE, self).__init__()
        self.latent_dim = latent_dim
        
        # Encoder
        self.encoder = tf.keras.Sequential([
            tf.keras.layers.Input(shape=(28, 28, 1)),
            tf.keras.layers.Flatten(),
            tf.keras.layers.Dense(512, activation='relu'),
            tf.keras.layers.Dense(256, activation='relu'),
        ])
        
        self.mean_layer = tf.keras.layers.Dense(latent_dim)
        self.logvar_layer = tf.keras.layers.Dense(latent_dim)
        
        # Decoder
        self.decoder = tf.keras.Sequential([
            tf.keras.layers.Input(shape=(latent_dim,)),
            tf.keras.layers.Dense(256, activation='relu'),
            tf.keras.layers.Dense(512, activation='relu'),
            tf.keras.layers.Dense(28 * 28 * 1, activation='sigmoid'),
            tf.keras.layers.Reshape((28, 28, 1))
        ])
    
    def encode(self, x):
        h = self.encoder(x)
        mean = self.mean_layer(h)
        logvar = self.logvar_layer(h)
        return mean, logvar
    
    def reparameterize(self, mean, logvar):
        eps = tf.random.normal(shape=tf.shape(mean))
        return mean + eps * tf.exp(logvar * 0.5)
    
    def decode(self, z):
        return self.decoder(z)
    
    def call(self, x):
        mean, logvar = self.encode(x)
        z = self.reparameterize(mean, logvar)
        return self.decode(z), mean, logvar

# VAE loss function
def vae_loss(x, x_reconstructed, mean, logvar):
    # Reconstruction loss
    reconstruction_loss = tf.keras.losses.binary_crossentropy(x, x_reconstructed)
    reconstruction_loss = tf.reduce_sum(reconstruction_loss, axis=[1, 2])
    
    # KL divergence loss
    kl_loss = -0.5 * tf.reduce_sum(1 + logvar - tf.square(mean) - tf.exp(logvar), axis=1)
    
    return tf.reduce_mean(reconstruction_loss + kl_loss)

# Training VAE
@tf.function
def train_vae_step(model, optimizer, x):
    with tf.GradientTape() as tape:
        x_reconstructed, mean, logvar = model(x)
        loss = vae_loss(x, x_reconstructed, mean, logvar)
    
    gradients = tape.gradient(loss, model.trainable_variables)
    optimizer.apply_gradients(zip(gradients, model.trainable_variables))
    
    return loss

# Simple GAN
class SimpleGAN:
    def __init__(self, latent_dim=100):
        self.latent_dim = latent_dim
        self.generator = self.build_generator()
        self.discriminator = self.build_discriminator()
        
        self.gen_optimizer = tf.keras.optimizers.Adam(0.0002, beta_1=0.5)
        self.disc_optimizer = tf.keras.optimizers.Adam(0.0002, beta_1=0.5)
        
        self.cross_entropy = tf.keras.losses.BinaryCrossentropy(from_logits=True)
    
    def build_generator(self):
        model = tf.keras.Sequential([
            tf.keras.layers.Dense(256, activation='relu', input_shape=(self.latent_dim,)),
            tf.keras.layers.Dense(512, activation='relu'),
            tf.keras.layers.Dense(1024, activation='relu'),
            tf.keras.layers.Dense(28 * 28 * 1, activation='tanh'),
            tf.keras.layers.Reshape((28, 28, 1))
        ])
        return model
    
    def build_discriminator(self):
        model = tf.keras.Sequential([
            tf.keras.layers.Flatten(input_shape=(28, 28, 1)),
            tf.keras.layers.Dense(512, activation='relu'),
            tf.keras.layers.Dense(256, activation='relu'),
            tf.keras.layers.Dense(1)
        ])
        return model
    
    def discriminator_loss(self, real_output, fake_output):
        real_loss = self.cross_entropy(tf.ones_like(real_output), real_output)
        fake_loss = self.cross_entropy(tf.zeros_like(fake_output), fake_output)
        return real_loss + fake_loss
    
    def generator_loss(self, fake_output):
        return self.cross_entropy(tf.ones_like(fake_output), fake_output)
    
    @tf.function
    def train_step(self, real_images, batch_size):
        noise = tf.random.normal([batch_size, self.latent_dim])
        
        with tf.GradientTape() as gen_tape, tf.GradientTape() as disc_tape:
            generated_images = self.generator(noise, training=True)
            
            real_output = self.discriminator(real_images, training=True)
            fake_output = self.discriminator(generated_images, training=True)
            
            gen_loss = self.generator_loss(fake_output)
            disc_loss = self.discriminator_loss(real_output, fake_output)
        
        gen_gradients = gen_tape.gradient(gen_loss, self.generator.trainable_variables)
        disc_gradients = disc_tape.gradient(disc_loss, self.discriminator.trainable_variables)
        
        self.gen_optimizer.apply_gradients(zip(gen_gradients, self.generator.trainable_variables))
        self.disc_optimizer.apply_gradients(zip(disc_gradients, self.discriminator.trainable_variables))
        
        return gen_loss, disc_loss
```

## Performance Profiling and Debugging

### TensorFlow Profiler
```python
# Enable profiling
tf.profiler.experimental.start('logs/profile')

# Training code
for epoch in range(epochs):
    for batch in dataset:
        train_step(batch)

# Stop profiling
tf.profiler.experimental.stop()

# Profile specific function
@tf.function
def profiled_train_step(x, y):
    with tf.profiler.experimental.Trace('train_step', step_num=1):
        # Training logic
        pass

# Memory profiling
def profile_memory_usage():
    # Get current memory info
    memory_info = tf.config.experimental.get_memory_info('GPU:0')
    print(f"Current memory usage: {memory_info['current'] / (1024**3):.2f} GB")
    print(f"Peak memory usage: {memory_info['peak'] / (1024**3):.2f} GB")

# Custom profiling context
class ProfilerContext:
    def __init__(self, name):
        self.name = name
    
    def __enter__(self):
        self.start_time = tf.timestamp()
        return self
    
    def __exit__(self, *args):
        end_time = tf.timestamp()
        tf.print(f"{self.name} took: {end_time - self.start_time} seconds")

# Usage
with ProfilerContext("Training Step"):
    train_step(batch_x, batch_y)

# Debug utilities
def debug_model_outputs(model, input_data):
    """Debug intermediate layer outputs"""
    layer_outputs = []
    
    for i, layer in enumerate(model.layers):
        intermediate_model = tf.keras.Model(
            inputs=model.input,
            outputs=layer.output
        )
        output = intermediate_model(input_data)
        layer_outputs.append(output)
        
        print(f"Layer {i} ({layer.name}): {output.shape}")
        print(f"Min: {tf.reduce_min(output):.4f}, Max: {tf.reduce_max(output):.4f}")
        print(f"Mean: {tf.reduce_mean(output):.4f}, Std: {tf.math.reduce_std(output):.4f}")
        
        # Check for NaN or Inf
        if tf.reduce_any(tf.math.is_nan(output)):
            print(f"WARNING: NaN detected in layer {i}")
        if tf.reduce_any(tf.math.is_inf(output)):
            print(f"WARNING: Inf detected in layer {i}")
        
        print("-" * 50)
    
    return layer_outputs

# Gradient debugging
def debug_gradients(model, loss_fn, x, y):
    """Debug gradient flow through the model"""
    with tf.GradientTape() as tape:
        predictions = model(x, training=True)
        loss = loss_fn(y, predictions)
    
    gradients = tape.gradient(loss, model.trainable_variables)
    
    for i, (layer, grad) in enumerate(zip(model.layers, gradients)):
        if grad is not None:
            grad_norm = tf.norm(grad)
            print(f"Layer {i} ({layer.name}) - Gradient norm: {grad_norm:.6f}")
            
            if tf.reduce_any(tf.math.is_nan(grad)):
                print(f"WARNING: NaN gradients in layer {i}")
            if grad_norm < 1e-7:
                print(f"WARNING: Very small gradients in layer {i} (vanishing)")
            if grad_norm > 100:
                print(f"WARNING: Very large gradients in layer {i} (exploding)")
```