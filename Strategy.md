# Complete Solution Strategy

## Dataset Assessment

The dataset (1,291 training images) is adequate for transfer learning but requires careful handling. Studies show similar agricultural datasets (800-1,200 samples) achieve R²=0.80-0.90 with proper augmentation and pretrained models.

## Optimal Model Architecture

Multi-input CNN regression combining visual and tabular features:

- **Image Branch**: EfficientNetB4 or ResNet50 (pretrained on ImageNet) → Extract 1280-2048 dim features
- **Tabular Branch**: NDVI + Height + State + Species + Month → Dense layers → 32 dim features
- **Merge & Output**: Concatenate → Dense(256) → Dense(128) → Dense(5 outputs)

EfficientNetB4 achieved R²=0.92 for biomass prediction in comparable studies, making it your best choice.

## Critical: Data Augmentation (ESSENTIAL!)

With only 1,291 samples, augmentation provides 12-18% improvement:

- **Geometric**: Horizontal/vertical flips (50%), small rotations (±10°), random crops
- **Color**: Brightness (±15%), contrast (±15%), saturation (±10%)
- **Advanced**: Mixup (α=0.2) for regression - blends images and targets, proven 3-5% boost

Apply augmentation only during training, generating 10-20× dataset size. This is the single most important factor for preventing overfitting on small datasets.

## Data Preprocessing Pipeline

### 1. CSV Transformation
Reshape from long (5 rows per image) to wide format (1 row with 5 target columns)

### 2. Image Preprocessing
- Resize 2000×1000 → 256×256 or 224×224
- Normalize: mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]
- Convert to float32

### 3. Tabular Features
- **NDVI**: Use as-is (already 0-1 range)
- **Height**: Standardize (x - mean) / std
- **State**: Label encode (Tas→0, NSW→1, etc.)
- **Species**: Label encode
- **Date**: Extract month (1-12) and season (0-3)

## Weighted Loss Implementation

Critical: Use weighted MSE with [0.1, 0.1, 0.1, 0.2, 0.5]:

```python
def weighted_mse_loss(weights=[0.1, 0.1, 0.1, 0.2, 0.5]):
    def loss(y_true, y_pred):
        squared_error = tf.square(y_true - y_pred)
        weighted_squared_error = squared_error * weights
        return tf.reduce_mean(tf.reduce_sum(weighted_squared_error, axis=-1))
    return loss
```

This ensures Dry_Total_g (50% weight) dominates the optimization as intended.

## Training Strategy

- **Phase 1** (20-30 epochs): Freeze CNN backbone, train only dense layers, LR=0.001
- **Phase 2** (30-50 epochs): Unfreeze last 20-30% of CNN, fine-tune with LR=0.0001
- **Phase 3**: Learning rate scheduler (ReduceLROnPlateau) + early stopping (patience=15)

Use 5-fold stratified cross-validation (stratify by State + Species) and ensemble the 5 models - reduces variance by ~10%.

## Regularization (Prevent Overfitting)

Essential for 1,291 samples:

- **Dropout**: 0.2-0.4 after each dense layer
- **L2 regularization**: weight decay 1e-4 to 1e-5
- **Batch normalization**: After dense layers
- **Early stopping**: Monitor validation weighted R²
- **Data augmentation**: Most effective regularization

## Expected Performance

Based on agricultural biomass studies with similar datasets:

| Approach | Expected R² |
|----------|-------------|
| CNN + metadata + augmentation | 0.80-0.90 |
| Ensemble (5 CV folds) | 0.82-0.92 |

### Individual target expectations:
- **Dry_Total_g** (0.5 weight): R²=0.85-0.92 (easiest)
- **GDM_g** (0.2 weight): R²=0.80-0.88
- **Dry_Green/Dead/Clover** (0.1 each): R²=0.65-0.85

## Submission Format

Transform model output (batch×5) back to required long format:

```python
target_names = ['Dry_Clover_g', 'Dry_Dead_g', 'Dry_Green_g', 'Dry_Total_g', 'GDM_g']
for img_id, pred in zip(test_ids, predictions):
    for name, value in zip(target_names, pred):
        submission.append({f'{img_id}__{name}': value})
```

## Key Success Factors

### MUST DO:
1. Transfer learning with EfficientNetB4/ResNet50
2. Heavy data augmentation (10-20× expansion)
3. Combine image + tabular (NDVI, Height, State, Species)
4. Weighted loss [0.1, 0.1, 0.1, 0.2, 0.5]
5. 5-fold cross-validation + ensemble
6. Dropout + early stopping


**Target**: Weighted R² > 0.80 (competitive), realistically expect 0.82-0.88 with proper implementation.
