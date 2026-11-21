# DSCI 522: Hotel Booking Cancellation Prediction

This project analyzes hotel booking data and develops a baseline model to predict whether a reservation will be cancelled. The goal is to understand cancellation patterns and assess how well booking information can identify bookings at higher risk of cancellation.

## Project overview

The analysis uses the Hotel Booking Demand dataset, which contains 119,390 bookings from City Hotels and Resort Hotels. The target variable is `is_canceled`:

- `1`: the booking was cancelled
- `0`: the booking was not cancelled

There are 44,224 cancelled bookings (37.0%) and 75,166 bookings that were not cancelled (63.0%). Because the target classes are not evenly balanced, the project emphasizes average precision, recall, and F1 rather than accuracy alone.

## Analysis

The main report is [`analysis/main_analysis.qmd`](analysis/main_analysis.qmd). It downloads the source data, prepares the predictors, explores the training data, compares models, selects a decision threshold, and evaluates the selected model on a held-out test set.

The exploratory visualizations use the training partition only and show:

- The training-set cancellation balance.
- Cancellation rates across booking lead-time groups.
- Cancellation rates by deposit type.
- Cancellation rates by hotel type and arrival month.

The report also visualizes cross-validation model scores and the precision-recall trade-off across candidate decision thresholds.

### Data preparation and modeling

The analysis removes `reservation_status` and `reservation_status_date`, which reveal the booking outcome, and `assigned_room_type`, `booking_changes`, and `days_in_waiting_list`, which are not available at booking time. The remaining predictors are prepared using median imputation and standardization for numeric features, and missing-value handling and one-hot encoding for categorical features. Missing `agent` and `company` IDs are represented as `-1` and treated as categorical values.

The data is split into training and test partitions using an 80/20 stratified split with random seed 42. A majority-class dummy classifier is compared with logistic regression using five-fold cross-validation on the training data. Average precision is used for model selection. The decision threshold is selected by maximizing F1 on out-of-fold training predictions; the test set is held out for final evaluation.

The report also writes a preprocessed CSV for inspection. Model evaluation uses the preprocessing pipeline within cross-validation and training, rather than the exported CSV.

### Reported results

The current analysis reports logistic regression cross-validation average precision of 0.868, ROC-AUC of 0.908, and F1 of 0.749 at the default 0.5 threshold. The threshold selected on training predictions is 0.35. On the held-out test set, this threshold gives recall of 0.817 and F1 of 0.769; at 0.5, test recall is 0.697 and F1 is 0.748. Lowering the threshold catches more cancellations but also produces more false alarms.

These metrics are the results currently documented in the report and may change if the analysis or data changes.

## Repository contents

```text
.
├── analysis/
│   ├── main_analysis.qmd       # Analysis source
│   └── main_analysis.html      # Rendered report
├── data/
│   ├── raw/
│   │   └── hotel_bookings.csv  # Downloaded source data
│   └── processed/
│       └── hotel_bookings_processed.csv
├── pyproject.toml              # Python dependencies and project settings
├── uv.lock                     # Locked Python dependency versions
└── README.md
```

The raw and processed CSV files are created by the analysis when it is rendered.

## Setup and rendering

The project uses Python 3.14.7 and `uv` for Python dependency management. Install [Quarto](https://quarto.org/docs/get-started/) separately; it is the document renderer.

From the repository root, synchronize the Python environment and render the HTML report:

```powershell
uv sync
uv run quarto render analysis/main_analysis.qmd --to html
```

The rendered report is written to `analysis/main_analysis.html`. Rendering executes the Python code chunks, downloads the dataset from Kaggle using `kagglehub`, and creates the raw and processed CSV files. An internet connection is required for the initial dataset download.

To render the PDF version as well, you also need a LaTeX distribution, which `uv` does not install. The simplest option is TinyTeX via Quarto:

```powershell
quarto install tinytex
uv run quarto render analysis/main_analysis.qmd --to pdf
```

In Positron, select the project's `.venv` Python environment when running or rendering the report.

## Dataset source

The data is obtained from the [Hotel Booking Demand dataset on Kaggle](https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand/data) using `kagglehub`.

## Limitations

- The analysis compares only a dummy classifier and logistic regression; other models may perform better.
- The F1-based threshold assumes precision and recall are equally important, rather than using hotel-specific business costs.
- Duplicate records may occur in both the training and test sets, which could make evaluation scores optimistic.
- The dataset covers two hotels and a limited time period. A random split may not represent performance on future bookings or at other hotels.

## Authors

Trung Nghia Vo, Misha Shen, Nathan Li, and Anushka Seth.
