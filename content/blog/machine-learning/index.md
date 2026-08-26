---
title: "A Practical Machine Learning Workflow from Simulation to Forecasting"
date: 2026-08-25
summary: "Explore reproducible data generation, uncertainty simulation, time-series forecasting, recurrent neural networks, and transfer learning with R and Python."
tags:
  - Machine Learning
  - Python
  - R
  - Time Series
  - Deep Learning
  - Statistics
authors:
  - me
featured: false
---

Machine learning projects often begin before model fitting. We need a clean table, a defensible uncertainty model, a time-aware evaluation split, or a carefully controlled image input. This post develops those pieces in sequence: generate and inspect data, quantify uncertainty, forecast a time series, train an LSTM, and fine-tune a convolutional image classifier.

The examples are intentionally generic. Replace the paths, column names, labels, and model parameters with values appropriate for the project, and keep the data split and preprocessing decisions reproducible.

## The workflow

```text
raw table or images
  -> validation and exploratory summaries
  -> simulation and uncertainty checks
  -> time-aware baseline forecast
  -> sequence model
  -> transfer-learning classifier
  -> evaluation, diagnostics, and saved artifacts
```

## 1. Create a reproducible tabular dataset

When testing a workflow, a synthetic dataset makes the data-generating assumptions explicit. Set the seed before sampling and distinguish identifiers, dates, categorical variables, and outcomes:

```r
set.seed(123)
n_rows <- 1000

example_data <- data.frame(
  event_date = sample(
    seq(as.Date("2024-01-01"), as.Date("2024-12-31"), by = "day"),
    n_rows, replace = TRUE
  ),
  record_id = sample(100000:999999, n_rows),
  age = sample(0:100, n_rows, replace = TRUE),
  group = sample(c("group_a", "group_b"), n_rows, replace = TRUE),
  outcome = sample(c(0, 1), n_rows, replace = TRUE)
)

write.csv(example_data, "example_data.csv", row.names = FALSE)
```

Synthetic data is useful for exercising code paths, but it does not validate a model's scientific or clinical performance. For real data, define the target, prediction time, eligible population, and missing-data policy before training.

## 2. Validate and summarize an external table

The first pass over a table should identify missing values, malformed dates, duplicate identifiers, and impossible ranges:

```r
library(dplyr)
library(readr)

data_table <- read_csv("path/to/data.csv", show_col_types = FALSE)
missing_counts <- sapply(data_table, function(column) sum(is.na(column)))
print(missing_counts)

data_table <- data_table |>
  mutate(event_time = as.POSIXct(event_time, tz = "UTC")) |>
  filter(!is.na(event_time), !is.na(target))

summary_table <- data_table |>
  group_by(group) |>
  summarise(
    observations = n(),
    target_total = sum(target, na.rm = TRUE),
    .groups = "drop"
  )
```

Do not fill missing coordinates or measurements with a group-level value without documenting why that value is appropriate. Imputation should be fitted on the training data and applied to validation and test data without using future information.

## 3. Model probability distributions and uncertainty

The probability exercises compare normal, binomial, Poisson, and beta distributions. Generate samples and inspect whether their empirical behavior matches the assumed parameters:

```r
set.seed(0)
n_samples <- 1000
successes <- rbinom(n_samples, size = trial_count, prob = success_probability)
event_counts <- rpois(n_samples, lambda = expected_events)
proportions <- rbeta(n_samples, shape1 = shape_a, shape2 = shape_b)

par(mfrow = c(1, 3))
hist(successes, main = "Binomial", xlab = "successes")
hist(event_counts, main = "Poisson", xlab = "events")
hist(proportions, main = "Beta", xlab = "proportion")
```

The generic parameters must be defined before execution:

```r
trial_count <- 40
success_probability <- 0.5
expected_events <- 5
shape_a <- 2
shape_b <- 5
```

For an estimated mean, a t-based confidence interval is:

```r
sample_mean <- mean(observed_values)
sample_sd <- sd(observed_values)
sample_size <- length(observed_values)
confidence_level <- 0.95
standard_error <- sample_sd / sqrt(sample_size)
t_critical <- qt((1 + confidence_level) / 2, df = sample_size - 1)
margin <- t_critical * standard_error
confidence_interval <- c(sample_mean - margin, sample_mean + margin)
```

Relative half-width is useful when comparing precision across sample sizes:

```r
relative_half_width <- margin / abs(sample_mean)
```

It becomes unstable when the mean is near zero, so report the absolute interval as well.

## 4. Propagate uncertainty with Monte Carlo simulation

When an observed rate is adjusted by uncertain factors, simulate both the observed count and the adjustment factors rather than reporting only a point estimate. A beta distribution is parameterized by a mean and standard deviation as follows:

```r
beta_parameters <- function(mean_value, sd_value) {
  alpha <- (((1 - mean_value) / sd_value^2) - (1 / mean_value)) * mean_value^2
  beta <- ((1 / mean_value) - 1) * alpha
  c(alpha = alpha, beta = beta)
}

set.seed(42)
n_iterations <- 100000
population_size <- 100000
observed_events <- 250
true_rate <- observed_events / population_size
event_draws <- rbinom(n_iterations, population_size, true_rate)

factor_parameters <- beta_parameters(mean_value = 0.75, sd_value = 0.05)
adjustment_draws <- rbeta(
  n_iterations, factor_parameters["alpha"], factor_parameters["beta"]
)

crude_rate <- event_draws / population_size
adjusted_rate <- crude_rate / adjustment_draws
quantile(adjusted_rate, c(0.025, 0.5, 0.975))
```

For several age groups or strata, repeat the simulation with group-specific populations, event counts, and adjustment distributions. Check that beta parameters are positive and that simulated factors remain within the intended interval. A Monte Carlo interval reflects the assumptions supplied to the simulation; it is not automatically a causal or population-level confidence interval.

## 5. Build a time-series baseline

Time-series data must be ordered by time and split chronologically. Randomly shuffling observations can leak future information into training:

```python
import pandas as pd
import matplotlib.pyplot as plt
from statsmodels.tsa.stattools import adfuller

time_series = pd.read_csv("path/to/time_series.csv")
time_series["timestamp"] = pd.to_datetime(time_series["timestamp"])
time_series = time_series.sort_values("timestamp").set_index("timestamp")

series = time_series["target"].astype(float)
print(series.describe())
print("ADF p-value:", adfuller(series.dropna())[1])
series.plot(figsize=(12, 5), title="Observed time series")
plt.show()
```

Autocorrelation and partial autocorrelation plots help identify candidate orders, but they should support rather than replace validation:

```python
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf

plot_acf(series.dropna())
plot_pacf(series.dropna())
plt.show()
```

## 6. Evaluate ARIMA with a chronological holdout

Fit a baseline ARIMA model on the early portion of the series and forecast the held-out tail:

```python
import numpy as np
import statsmodels.api as sm
from sklearn.metrics import mean_absolute_error, mean_squared_error

values = series.dropna().to_numpy()
split_index = int(len(values) * 0.85)
train_values = values[:split_index]
test_values = values[split_index:]
p_order, differencing_order, q_order = 2, 0, 1

arima_model = sm.tsa.arima.ARIMA(
    train_values, order=(p_order, differencing_order, q_order)
).fit()
forecast_result = arima_model.get_forecast(steps=len(test_values))
predicted_values = forecast_result.predicted_mean
interval = forecast_result.conf_int()

mae = mean_absolute_error(test_values, predicted_values)
rmse = np.sqrt(mean_squared_error(test_values, predicted_values))
print({"MAE": mae, "RMSE": rmse})
```

Use a rolling-origin forecast when the deployed model will be updated after each newly observed value:

```python
history = list(train_values)
rolling_predictions = []

for observed_value in test_values:
    model = sm.tsa.arima.ARIMA(
        history, order=(p_order, differencing_order, q_order)
    ).fit()
    next_prediction = model.get_forecast(steps=1).predicted_mean[0]
    rolling_predictions.append(next_prediction)
    history.append(observed_value)
```

Compare fixed-model and rolling-update forecasts using the same test dates and metrics. Include prediction intervals and inspect residuals; a low average error can hide systematic underprediction during peaks.

## 7. Train an LSTM on rolling windows

An LSTM consumes a three-dimensional array with shape `[samples, time_steps, features]`. Fit scaling parameters on training data only, then construct windows for the target:

```python
import numpy as np
from sklearn.preprocessing import StandardScaler
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import LSTM, Dense

feature_frame = pd.read_csv("path/to/features.csv")
feature_frame["timestamp"] = pd.to_datetime(feature_frame["timestamp"])
feature_frame = feature_frame.sort_values("timestamp")

target_name = "target"
feature_names = [name for name in feature_frame.columns
                 if name not in ["timestamp", target_name]]
split_index = int(len(feature_frame) * 0.8)
train_frame = feature_frame.iloc[:split_index].copy()
test_frame = feature_frame.iloc[split_index:].copy()

scaler = StandardScaler()
train_frame[feature_names] = scaler.fit_transform(train_frame[feature_names])
test_frame[feature_names] = scaler.transform(test_frame[feature_names])

window_size = 7
epochs = 40
batch_size = 32
def make_windows(frame, feature_names, target_name, window_size):
    x_values = frame[feature_names].to_numpy()
    y_values = frame[target_name].to_numpy()
    x_windows, y_values_out = [], []
    for index in range(window_size, len(frame)):
        x_windows.append(x_values[index - window_size:index])
        y_values_out.append(y_values[index])
    return np.asarray(x_windows), np.asarray(y_values_out)

x_train, y_train = make_windows(
    train_frame, feature_names, target_name, window_size
)
x_test, y_test = make_windows(
    test_frame, feature_names, target_name, window_size
)

lstm_model = Sequential([
    LSTM(50, return_sequences=True,
         input_shape=(x_train.shape[1], x_train.shape[2])),
    LSTM(50),
    Dense(1)
])
lstm_model.compile(loss="mean_squared_error", optimizer="adam")
lstm_model.fit(x_train, y_train, epochs=epochs, batch_size=batch_size)
```

Evaluate on a future holdout and compare with the ARIMA baseline. Do not scale the target using information from the test period, and do not build a window that crosses the train/test boundary unless that behavior is explicitly part of deployment.

## 8. Classify images with transfer learning

For image classification, a pretrained convolutional backbone can provide useful features when the available labeled dataset is limited. Keep class directories separated by subject or acquisition unit when related images could otherwise appear in both splits:

```python
from glob import glob
from tensorflow.keras.applications import VGG16
from tensorflow.keras.layers import Dense, Flatten
from tensorflow.keras.models import Model
from tensorflow.keras.preprocessing.image import ImageDataGenerator

image_size = (224, 224)
train_dir = "path/to/train_images"
validation_dir = "path/to/validation_images"
batch_size = 32
epochs = 10
class_names = sorted(glob(f"{train_dir}/*"))

backbone = VGG16(
    input_shape=image_size + (3,),
    weights="imagenet",
    include_top=False
)
for layer in backbone.layers:
    layer.trainable = False

features = Flatten()(backbone.output)
outputs = Dense(len(class_names), activation="softmax")(features)
image_model = Model(inputs=backbone.input, outputs=outputs)
image_model.compile(
    loss="categorical_crossentropy",
    optimizer="adam",
    metrics=["accuracy"]
)
```

Apply augmentation only to training images and use the same target size for both generators:

```python
train_generator = ImageDataGenerator(
    rescale=1 / 255,
    shear_range=0.2,
    zoom_range=0.2,
    horizontal_flip=True
).flow_from_directory(
    train_dir, target_size=image_size, batch_size=batch_size,
    class_mode="categorical"
)

validation_generator = ImageDataGenerator(rescale=1 / 255).flow_from_directory(
    validation_dir, target_size=image_size, batch_size=batch_size,
    class_mode="categorical", shuffle=False
)

history = image_model.fit(
    train_generator,
    validation_data=validation_generator,
    epochs=epochs
)
image_model.save("trained_image_model.keras")
```

Accuracy alone can be misleading for imbalanced classes. Report class-wise precision, recall, sensitivity, specificity, calibration, and a confusion matrix. For medical or other high-stakes images, evaluate at the subject level, preserve an untouched test set, and do not interpret a demonstration model as a diagnostic system.

## 9. Run inference consistently

Inference must use the same resize, channel ordering, normalization, and class-index mapping as training:

```python
from tensorflow.keras.models import load_model
from tensorflow.keras.preprocessing import image
from tensorflow.keras.applications.vgg16 import preprocess_input

image_model = load_model("trained_image_model.keras")
input_image = image.load_img("path/to/image.jpg", target_size=image_size)
image_array = image.img_to_array(input_image)
image_array = np.expand_dims(image_array, axis=0)
prediction = image_model.predict(preprocess_input(image_array))
predicted_index = int(np.argmax(prediction[0]))
predicted_label = class_names[predicted_index]
print(predicted_label, float(prediction[0, predicted_index]))
```

Save the class mapping with the model. A probability is a model score unless calibration has been evaluated on representative data.

## Reproducibility and failure checks

- Set and record random seeds, package versions, input snapshots, and model parameters.
- Split time series chronologically and split images by subject or acquisition unit to prevent leakage.
- Fit imputers, scalers, and feature selectors on training data only.
- Compare complex models with simple baselines and report uncertainty intervals where appropriate.
- Check non-finite values, impossible dates, duplicate IDs, class imbalance, and missing labels before fitting.
- Preserve the exact preprocessing and class mapping needed for inference.
- Use run-specific output directories for models, predictions, plots, metrics, and configuration.

The common thread is disciplined evaluation. Simulation makes uncertainty visible, ARIMA provides a transparent temporal baseline, LSTMs model longer feature histories, and transfer learning reuses visual representations. None of those choices removes the need for a correct split, an explicit target, and diagnostics tied to the real decision the model will support.
