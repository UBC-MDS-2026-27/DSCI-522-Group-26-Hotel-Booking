# Hotel Booking Cancellation Prediction

This project analyzes hotel booking data and develops a baseline model to predict whether a reservation will be cancelled. The goal is to understand cancellation patterns and assess how well booking information can identify bookings at higher risk of cancellation.

## Authors

- Trung Nghia Vo
- Misha Shen
- Nathan Li
- Anushka Seth

## Summary

This project asks whether hotel-booking information available at reservation time can predict whether a booking will be cancelled. We compare a majority-class baseline with logistic regression and evaluate both on a held-out test set. Logistic regression performs much better at ranking and identifying cancellations, and a lower decision threshold catches more cancellations while producing more false alarms. These findings could help hotels direct cancellation-risk follow-up more effectively, but the best threshold depends on hotel-specific costs and must be validated on future and external bookings.

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

## How to run the analysis

1. Install Git, Python 3.14.7, `uv`, and [Quarto](https://quarto.org/docs/get-started/).
2. Clone the repository and enter it:

   ```powershell
   git clone https://github.com/UBC-MDS-2026-27/DSCI-522-Group-26-Hotel-Booking.git
   cd DSCI-522-Group-26-Hotel-Booking
   ```

3. From the repository root, create or update the project environment and install the dependencies:

   ```powershell
   uv sync
   ```

4. Render the main report:

   ```powershell
   uv run quarto render analysis/main_analysis.qmd --to html
   ```

The rendered report is written to `analysis/main_analysis.html`. Rendering executes the Python code chunks, downloads the source dataset with `kagglehub`, and creates `data/raw/hotel_bookings.csv` and `data/processed/hotel_bookings_processed.csv`. An internet connection is required for the initial dataset download.

In an editor such as Positron or Visual Studio Code, select the `.venv` Python environment created by `uv` if you run the notebook or Python code interactively.

## Dependencies

The Python dependencies are listed in [`pyproject.toml`](pyproject.toml). The project uses Python 3.14.7 and `uv`; Quarto is required to render the report. The analysis dependencies include `kagglehub`, `pandas`, `numpy`, `scipy`, `matplotlib`, `seaborn`, `scikit-learn`, and `jupyter`. Quarto and `uv` are external tools rather than Python packages in `pyproject.toml`.

## Dataset source

The data is obtained from the [Hotel Booking Demand dataset on Kaggle](https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand/data) using `kagglehub`.

## Limitations

- The analysis compares only a dummy classifier and logistic regression; other models may perform better.
- The F1-based threshold assumes precision and recall are equally important, rather than using hotel-specific business costs.
- Duplicate records may occur in both the training and test sets, which could make evaluation scores optimistic.
- The dataset covers two hotels and a limited time period. A random split may not represent performance on future bookings or at other hotels.

## Licenses

- The original project code is licensed under the [MIT License](LICENSE.md).
- The original report, figures, tables, and rendered report content are licensed under the [Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License](LICENSE.md) (CC BY-NC-ND 4.0).
- Embedded report code remains under the MIT License.
- Third-party software and data retain their respective licenses.

## AI use statement

Claude Sonnet 5.5 (Anthropic) and ChatGPT (OpenAI) were used to proofread this report and to suggest code syntax for threshold selection and uv environment setup. All outputs were reviewed and verified by the authors, who take full responsibility for the final content. Example prompts used during development include:

- Proofread this section for grammar and clarity. Keep technical terms and meaning unchanged, and list your changes.

- Suggest scikit-learn syntax to select the F1-maximizing classification threshold on a validation set.

- Suggest how to safely change the >= dependency constraints in my pyproject.toml to exact == versions that match my current uv environment, and update the project version.