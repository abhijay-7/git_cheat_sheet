# PyTorch Intermediate Cheatsheet

## Custom Datasets and DataLoaders

### Custom Dataset Class
```python
from torch.utils.data import Dataset, DataLoader
from PIL import Image
import pandas as pd

class CustomImageDataset(Dataset):
    def __init__(self, csv_file, img_dir, transform=None):
        self.data = pd.read_csv(csv_file)
        self.img_dir = img_dir
        self.transform = transform
    
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, idx):
        img_name = os.path.join(self.img_dir, self.data.iloc[idx, 0])
        image = Image.open(img_name)
        label = self.data.iloc[idx, 1]
        
        if self.transform:
            image = self.transform(image)
            
        return image, label
```

### Advanced DataLoader
```python
from torch.utils.data import DataLoader, random_split

# Dataset splitting
dataset = CustomImageDataset(csv_file, img_dir, transform)
train_size = int(0.8 * len(dataset))
val_size = len(dataset) - train_size
train_dataset, val_dataset = random_split(dataset, [train_size, val_size])

# DataLoader with advanced options
train_loader = DataLoader(
    train_dataset,
    batch_size=32,
    shuffle=True,
    num_workers=4,      # Parallel data loading
    pin_memory=True,    # Faster GPU transfer
    drop_last=True      # Drop incomplete batches
)

val_loader = DataLoader(
    val_dataset,
    batch_size=64,
    shuffle=False,
    num_workers=4,
    pin_memory=True
)
```

## Convolutional Neural Networks (CNNs)

### Basic CNN Architecture
```python
class CNN(nn.Module):
    def __init__(self, num_classes=10):
        super(CNN, self).__init__()
        
        self.features = nn.Sequential(
            # First conv block
            nn.Conv2d(3, 64, kernel_size=3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=2, stride=2),
            
            # Second conv block
            nn.Conv2d(64, 128, kernel_size=3, padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=2, stride=2),
            
            # Third conv block
            nn.Conv2d(128, 256, kernel_size=3, padding=1),
            nn.BatchNorm2d(256),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=2, stride=2)
        )
        
        self.classifier = nn.Sequential(
            nn.AdaptiveAvgPool2d((7, 7)),
            nn.Flatten(),
            nn.Linear(256 * 7 * 7, 512),
            nn.ReLU(inplace=True),
            nn.Dropout(0.5),
            nn.Linear(512, num_classes)
        )
    
    def forward(self, x):
        x = self.features(x)
        x = self.classifier(x)
        return x
```

### CNN Layers and Operations
```python
# Convolutional layers
conv2d = nn.Conv2d(in_channels=3, out_channels=64, kernel_size=3, stride=1, padding=1)
conv1d = nn.Conv1d(in_channels=100, out_channels=128, kernel_size=5)

# Pooling layers
maxpool = nn.MaxPool2d(kernel_size=2, stride=2)
avgpool = nn.AvgPool2d(kernel_size=2, stride=2)
adaptive_pool = nn.AdaptiveAvgPool2d((7, 7))

# Normalization layers
batch_norm = nn.BatchNorm2d(64)
instance_norm = nn.InstanceNorm2d(64)
layer_norm = nn.LayerNorm([64, 32, 32])

# Upsampling
upsample = nn.Upsample(scale_factor=2, mode='bilinear', align_corners=False)
conv_transpose = nn.ConvTranspose2d(64, 32, kernel_size=4, stride=2, padding=1)
```

## Recurrent Neural Networks (RNNs)

### LSTM Network
```python
class LSTMModel(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers, num_classes):
        super(LSTMModel, self).__init__()
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        
        self.lstm = nn.LSTM(
            input_size, 
            hidden_size, 
            num_layers, 
            batch_first=True,
            dropout=0.2 if num_layers > 1 else 0
        )
        self.fc = nn.Linear(hidden_size, num_classes)
        
    def forward(self, x):
        # Initialize hidden and cell states
        h0 = torch.zeros(self.num_layers, x.size(0), self.hidden_size).to(x.device)
        c0 = torch.zeros(self.num_layers, x.size(0), self.hidden_size).to(x.device)
        
        # Forward pass through LSTM
        out, (hn, cn) = self.lstm(x, (h0, c0))
        
        # Use the last output
        out = self.fc(out[:, -1, :])
        return out

# Usage
model = LSTMModel(input_size=100, hidden_size=128, num_layers=2, num_classes=10)
```

### GRU and RNN Variants
```python
# GRU
gru = nn.GRU(input_size=100, hidden_size=128, num_layers=2, batch_first=True)

# Bidirectional LSTM
bi_lstm = nn.LSTM(input_size=100, hidden_size=128, num_layers=2, 
                  batch_first=True, bidirectional=True)

# Simple RNN
rnn = nn.RNN(input_size=100, hidden_size=128, num_layers=2, batch_first=True)
```

## Transfer Learning

### Using Pre-trained Models
```python
import torchvision.models as models

# Load pre-trained ResNet
model = models.resnet18(pretrained=True)

# Freeze all parameters
for param in model.parameters():
    param.requires_grad = False

# Replace the final layer for your task
num_features = model.fc.in_features
model.fc = nn.Linear(num_features, num_classes)

# Only final layer parameters will be updated
optimizer = optim.Adam(model.fc.parameters(), lr=0.001)
```

### Fine-tuning
```python
# Load pre-trained model
model = models.resnet18(pretrained=True)

# Modify final layer
model.fc = nn.Linear(model.fc.in_features, num_classes)

# Different learning rates for different parts
optimizer = optim.Adam([
    {'params': model.features.parameters(), 'lr': 0.0001},  # Lower LR for pre-trained
    {'params': model.fc.parameters(), 'lr': 0.001}          # Higher LR for new layer
])
```

## Data Transforms and Augmentation

```python
from torchvision import transforms

# Training transforms (with augmentation)
train_transform = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.RandomHorizontalFlip(p=0.5),
    transforms.RandomRotation(10),
    transforms.ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2, hue=0.1),
    transforms.RandomAffine(degrees=0, translate=(0.1, 0.1)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], 
                        std=[0.229, 0.224, 0.225])
])

# Validation transforms (no augmentation)
val_transform = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], 
                        std=[0.229, 0.224, 0.225])
])

# Custom transforms
class AddNoise(object):
    def __init__(self, noise_factor=0.1):
        self.noise_factor = noise_factor
    
    def __call__(self, tensor):
        noise = torch.randn_like(tensor) * self.noise_factor
        return tensor + noise
```

## Advanced Training Techniques

### Learning Rate Scheduling
```python
from torch.optim.lr_scheduler import *

optimizer = optim.Adam(model.parameters(), lr=0.001)

# Step decay
scheduler = StepLR(optimizer, step_size=30, gamma=0.1)

# Exponential decay
scheduler = ExponentialLR(optimizer, gamma=0.95)

# Reduce on plateau
scheduler = ReduceLROnPlateau(optimizer, mode='min', factor=0.5, patience=5)

# Cosine annealing
scheduler = CosineAnnealingLR(optimizer, T_max=50)

# One cycle
scheduler = OneCycleLR(optimizer, max_lr=0.01, epochs=100, steps_per_epoch=len(train_loader))

# Usage in training loop
for epoch in range(num_epochs):
    train_one_epoch()
    val_loss = validate()
    
    if isinstance(scheduler, ReduceLROnPlateau):
        scheduler.step(val_loss)
    else:
        scheduler.step()
```

## Regularization Techniques

### Dropout
```python
# Standard dropout
dropout = nn.Dropout(p=0.5)

# 2D dropout (for CNNs)
dropout2d = nn.Dropout2d(p=0.25)

# Alpha dropout (for SELU activation)
alpha_dropout = nn.AlphaDropout(p=0.5)

# Feature alpha dropout
feature_dropout = nn.FeatureAlphaDropout(p=0.5)
```

### Weight Decay and L2 Regularization
```python
# L2 regularization through weight decay
optimizer = optim.Adam(model.parameters(), lr=0.001, weight_decay=1e-4)

# Manual L1/L2 regularization
def l1_regularization(model, lambda_l1):
    l1_penalty = 0
    for param in model.parameters():
        l1_penalty += torch.sum(torch.abs(param))
    return lambda_l1 * l1_penalty

def l2_regularization(model, lambda_l2):
    l2_penalty = 0
    for param in model.parameters():
        l2_penalty += torch.sum(param ** 2)
    return lambda_l2 * l2_penalty

# In training loop
loss = criterion(outputs, targets)
loss += l1_regularization(model, 1e-5)
loss += l2_regularization(model, 1e-4)
```

## Model Saving and Loading

### State Dictionary Approach
```python
# Save model
torch.save({
    'epoch': epoch,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),
    'loss': loss,
    'scheduler_state_dict': scheduler.state_dict()
}, 'checkpoint.pth')

# Load model
checkpoint = torch.load('checkpoint.pth')
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
scheduler.load_state_dict(checkpoint['scheduler_state_dict'])
epoch = checkpoint['epoch']
loss = checkpoint['loss']
```

### Model Checkpointing
```python
class ModelCheckpoint:
    def __init__(self, filepath, monitor='val_loss', mode='min'):
        self.filepath = filepath
        self.monitor = monitor
        self.mode = mode
        self.best_score = float('inf') if mode == 'min' else -float('inf')
    
    def __call__(self, score, model, optimizer, epoch):
        if (self.mode == 'min' and score < self.best_score) or \
           (self.mode == 'max' and score > self.best_score):
            self.best_score = score
            torch.save({
                'epoch': epoch,
                'model_state_dict': model.state_dict(),
                'optimizer_state_dict': optimizer.state_dict(),
                'score': score
            }, self.filepath)
            print(f'Saved best model with {self.monitor}: {score:.4f}')

# Usage
checkpoint = ModelCheckpoint('best_model.pth', monitor='val_acc', mode='max')
# In validation loop
checkpoint(val_accuracy, model, optimizer, epoch)
```

## Custom Loss Functions

```python
# Focal Loss for imbalanced datasets
class FocalLoss(nn.Module):
    def __init__(self, alpha=1, gamma=2):
        super(FocalLoss, self).__init__()
        self.alpha = alpha
        self.gamma = gamma
    
    def forward(self, inputs, targets):
        ce_loss = F.cross_entropy(inputs, targets, reduction='none')
        pt = torch.exp(-ce_loss)
        focal_loss = self.alpha * (1 - pt) ** self.gamma * ce_loss
        return focal_loss.mean()

# Dice Loss for segmentation
class DiceLoss(nn.Module):
    def __init__(self, smooth=1):
        super(DiceLoss, self).__init__()
        self.smooth = smooth
    
    def forward(self, inputs, targets):
        inputs = torch.sigmoid(inputs)
        inputs = inputs.view(-1)
        targets = targets.view(-1)
        
        intersection = (inputs * targets).sum()
        dice = (2. * intersection + self.smooth) / (inputs.sum() + targets.sum() + self.smooth)
        return 1 - dice

# Combined Loss
class CombinedLoss(nn.Module):
    def __init__(self, alpha=0.5):
        super(CombinedLoss, self).__init__()
        self.alpha = alpha
        self.ce_loss = nn.CrossEntropyLoss()
        self.focal_loss = FocalLoss()
    
    def forward(self, inputs, targets):
        ce = self.ce_loss(inputs, targets)
        focal = self.focal_loss(inputs, targets)
        return self.alpha * ce + (1 - self.alpha) * focal
```

## Model Evaluation and Metrics

```python
from sklearn.metrics import accuracy_score, precision_recall_fscore_support, confusion_matrix
import numpy as np

def evaluate_model(model, dataloader, device):
    model.eval()
    all_preds = []
    all_targets = []
    total_loss = 0
    
    with torch.no_grad():
        for data, targets in dataloader:
            data, targets = data.to(device), targets.to(device)
            outputs = model(data)
            loss = F.cross_entropy(outputs, targets)
            
            total_loss += loss.item()
            _, predicted = torch.max(outputs, 1)
            
            all_preds.extend(predicted.cpu().numpy())
            all_targets.extend(targets.cpu().numpy())
    
    # Calculate metrics
    accuracy = accuracy_score(all_targets, all_preds)
    precision, recall, f1, _ = precision_recall_fscore_support(all_targets, all_preds, average='weighted')
    
    return {
        'loss': total_loss / len(dataloader),
        'accuracy': accuracy,
        'precision': precision,
        'recall': recall,
        'f1': f1
    }

# Top-k accuracy
def top_k_accuracy(output, target, topk=(1, 5)):
    with torch.no_grad():
        maxk = max(topk)
        batch_size = target.size(0)
        
        _, pred = output.topk(maxk, 1, True, True)
        pred = pred.t()
        correct = pred.eq(target.view(1, -1).expand_as(pred))
        
        res = []
        for k in topk:
            correct_k = correct[:k].reshape(-1).float().sum(0, keepdim=True)
            res.append(correct_k.mul_(100.0 / batch_size))
        return res
```

## Memory Management and Optimization

```python
# Clear GPU cache
torch.cuda.empty_cache()

# Gradient accumulation for large batch sizes
accumulation_steps = 4
optimizer.zero_grad()

for i, (data, targets) in enumerate(train_loader):
    outputs = model(data)
    loss = criterion(outputs, targets) / accumulation_steps
    loss.backward()
    
    if (i + 1) % accumulation_steps == 0:
        optimizer.step()
        optimizer.zero_grad()

# Mixed precision training (basic)
from torch.cuda.amp import autocast, GradScaler

scaler = GradScaler()

for data, targets in train_loader:
    optimizer.zero_grad()
    
    with autocast():
        outputs = model(data)
        loss = criterion(outputs, targets)
    
    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
```

## Debugging and Monitoring

```python
# Register hooks to monitor gradients
def print_grad_norm(model):
    total_norm = 0
    for name, param in model.named_parameters():
        if param.grad is not None:
            param_norm = param.grad.data.norm(2)
            total_norm += param_norm.item() ** 2
            print(f'{name}: {param_norm:.4f}')
    total_norm = total_norm ** (1. / 2)
    print(f'Total norm: {total_norm:.4f}')

# Gradient clipping
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)

# NaN checking
def check_for_nan(model):
    for name, param in model.named_parameters():
        if torch.isnan(param).any():
            print(f'NaN detected in {name}')
            return True
    return False
```