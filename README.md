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

Install [Git](https://git-scm.com/downloads) and
[Docker Desktop](https://www.docker.com/products/docker-desktop/).
Start Docker Desktop and wait until the Docker engine is running.
Python, uv, Quarto, and the analysis packages are provided inside
the container; they do not need to be installed separately on
your computer.

Run the following commands in Git Bash on Windows or Terminal
on macOS.

### Clone the repository

```bash
git clone https://github.com/UBC-MDS-2026-27/DSCI-522-Group-26-Hotel-Booking.git
cd DSCI-522-Group-26-Hotel-Booking
```

Run all remaining commands from this repository directory.

### Prepare the mounted configuration files

The container mounts your SSH public key and Git configuration.
Check whether these files already exist:

```bash
ls -l ~/.ssh/id_ed25519.pub ~/.gitconfig
```

If you do not already have an SSH key at that location, generate
one:

```bash
ssh-keygen -t ed25519
```

Accept the default file location. Follow the prompts to choose a
passphrase or leave it empty. Do not overwrite an existing key.

Check your Git identity:

```bash
git config --global user.name
git config --global user.email
```

If either value is missing, set it using your own name and email:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### Download the project environment

```bash
docker compose pull
```

Docker Compose downloads the image specified in
[`docker-compose.yml`](docker-compose.yml), pinned to a commit-SHA
tag. Use the configured tag rather than replacing it with `latest`.

### Render the report

```bash
docker compose run --rm analysis-env uv run quarto render analysis/main_analysis.qmd --to html
```

This command executes the analysis inside the container. It
downloads the dataset from Kaggle, saves the original CSV to
`data/raw/hotel_bookings.csv`, writes the processed data to
`data/processed/hotel_bookings_processed.csv`, and generates
`analysis/main_analysis.html`.

An internet connection is required to download the Docker image
and dataset. Open `analysis/main_analysis.html` in a web browser
to read the report. Its figures and styles are embedded in the
HTML file.

The `--rm` option automatically removes the temporary container
when rendering finishes. The report and data remain in your local
project directory because it is mounted into the container.

For editing the project, connecting Positron to the container,
or updating dependencies, see
[`CONTRIBUTING.md`](CONTRIBUTING.md).

## Dependencies

The reproducible environment is defined by
[`Dockerfile`](Dockerfile),
[`docker-compose.yml`](docker-compose.yml),
[`pyproject.toml`](pyproject.toml),
[`uv.lock`](uv.lock), and
[`.python-version`](.python-version).

The Docker image supplies Python, uv, Quarto, and the required
Python packages. Exact direct Python dependency versions are
listed in `pyproject.toml`, and resolved dependency versions are
recorded in `uv.lock`.

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