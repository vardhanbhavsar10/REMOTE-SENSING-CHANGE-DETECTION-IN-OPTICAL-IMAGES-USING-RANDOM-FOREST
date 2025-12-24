This script builds an end-to-end Random Forest–based change detection pipeline for multiple pairs of grayscale satellite/remote sensing images.

Step 1: Load and normalize image pairs  
- Each pair of images (before/after) is read in grayscale using OpenCV (`cv2.imread`).  
- Images are checked to ensure they have the same shape.  
- Pixel values are converted to `float` and normalized to the \([0, 1]\) range to standardize inputs for later processing.

Step 2: Extract pixel-wise neighborhood features  
- For each image pair, the code pads both images and iterates over every pixel.  
- Around each pixel, a local window (default \(5 \times 5\)) is extracted from both the before and after images.  
- For each pixel, a 3D feature vector is computed:  
  - Absolute intensity difference between before and after.  
  - Absolute difference in local mean between the two windows.  
  - Absolute difference in local standard deviation between the two windows.  
- These feature vectors and their pixel coordinates are stored for later prediction.

Step 3: Simulate training labels (ground truth)  
- Because real labels are not provided, the script simulates ground truth.  
- It randomly samples a subset of pixels (default 1000) from the full feature set.  
- For each sampled pixel, a “change” (1) or “no-change” (0) label is assigned based on the intensity difference and a threshold, with added Gaussian noise to mimic realistic variation.  
- This produces a training set: sampled features + corresponding binary labels.

Step 4: Train and tune the Random Forest model  
- The sampled features and labels are split into training and test sets using `train_test_split` (stratified).  
- A `RandomForestClassifier` is wrapped in `GridSearchCV` with a hyperparameter grid:  
  - `n_estimators` (number of trees),  
  - `max_depth`,  
  - `min_samples_split`,  
  - `min_samples_leaf`,  
  - `max_features`.  
- 3-fold cross-validation is used with accuracy as the scoring metric.  
- After fitting, the script:  
  - Retrieves the best estimator.  
  - Evaluates it on the held-out test set.  
  - Prints best hyperparameters, accuracy, and a classification report.

Step 5: Predict full-scene change map  
- The best Random Forest model is applied to the full feature matrix for the entire image pair (all pixels).  
- For each pixel’s feature vector, the model predicts “change” or “no-change”.  
- Predictions are mapped back to their original coordinates to reconstruct a 2D change map:  
  - Changed pixels → 255 (white).  
  - Unchanged pixels → 0 (black).

Step 6: Enhance original images for visualization  
- A `linear_stretch` function improves contrast of grayscale images:  
  - Computes the 2nd and 98th percentile of pixel values.  
  - Linearly rescales values between these percentiles to \([0, 1]\).  
  - Clips values outside the range, enhancing contrast while avoiding outliers.

Step 7: Visualize results for all pairs  
- The script iterates over all specified image pairs.  
- For each pair, it:  
  - Loads images and extracts features.  
  - Simulates labels and trains the Random Forest.  
  - Predicts the change map.  
  - Enhances before/after images with linear stretching.  
- Using Matplotlib, it creates a multi-row figure:  
  - Column 1: Original “Before” image.  
  - Column 2: Original “After” image.  
  - Column 3: Enhanced “Before” image.  
  - Column 4: Enhanced “After” image.  
  - Column 5: Predicted binary change map.  
- `plt.tight_layout()` and `plt.show()` render the full grid, allowing easy visual comparison of raw images and detected changes for each region.
