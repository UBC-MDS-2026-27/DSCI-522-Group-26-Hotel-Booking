# Contributing to the Hotel Booking Cancellation Prediction Project

This repository contains an analysis that predicts whether a hotel booking will be cancelled (`is_canceled`) using the [Hotel Booking Demand dataset](https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand) from Kaggle. The analysis compares a majority-class baseline with a logistic regression model, selects the model by average precision using cross-validation, chooses a decision threshold on out-of-fold training predictions, and evaluates once on a held-out test set.

Contributions from outside the core team are welcome, whether that is reporting a bug, suggesting an improvement, fixing a typo, or adding a new model comparison. This document explains how to do each of these.

By participating, you agree to follow this project's Code of Conduct (see `CODE_OF_CONDUCT.md` in the root of this repository).

## Table of contents

1. [Ways to contribute](#ways-to-contribute)
2. [Reporting a bug](#reporting-a-bug)
3. [Suggesting a change or new feature](#suggesting-a-change-or-new-feature)
4. [Submitting your work](#submitting-your-work)
5. [Project conventions](#project-conventions)
6. [AI policy](#ai-policy)
7. [Attribution](#attribution)

## Ways to contribute

Some contributions that would be especially helpful:

- Fixing errors in the analysis, code, or written report.
- Improving documentation, figures, or explanations.
- Adding another model to the comparison.
- Addressing the limitations noted in the report, such as removing duplicate rows before splitting, or using a time-based split instead of a random split.
- Suggesting a decision threshold based on realistic cancellation and false-alarm costs rather than the F1-maximizing assumption.

## Reporting a bug

If something in the analysis does not run, gives a result that differs from the report, or looks wrong:

1. **Search existing issues first** (open and closed) in this repository's **Issues** tab to see whether it has already been reported.
2. **Open a new issue** if it has not. Give it a short, descriptive title.
3. **Include the following details:**
   - What you expected to happen and what actually happened.
   - The steps to reproduce the problem, including which file or code cell you ran.
   - The full error message or traceback, if there is one.
   - Your operating system, Python version, and the versions of key packages (`pandas`, `numpy`, `scikit-learn`, `scipy`, `kagglehub`).
   - Any changes you made to the code before the problem appeared.

Please do not include Kaggle credentials, API tokens, or other private information in an issue.

## Suggesting a change or new feature

1. Search existing issues to make sure the idea has not already been proposed.
2. Open a new issue describing:
   - The change you would like to see and why it would improve the project.
   - How it relates to the project goal (predicting booking cancellations).
   - Any alternatives you considered.
3. Wait for feedback from a maintainer before starting large pieces of work. This avoids duplicated effort and makes it more likely your pull request will be accepted. Small fixes (typos, broken links, minor wording) do not need a prior issue.

## Submitting your work

We use the standard fork-and-pull-request workflow.

1. **Fork** the repository to your own GitHub account, then **clone** your fork:
   ```bash
   git clone https://github.com/<your-username>/<repository-name>.git
   cd <repository-name>
   ```
   (Use your own GitHub username and the name of this repository.)
2. **Create a branch** from `main` for your work. Use a short, descriptive name, for example `fix-threshold-table-typo`:
   ```bash
   git checkout -b fix-threshold-table-typo
   ```
3. **Make your changes.** Keep each pull request focused on one topic. See [Project conventions](#project-conventions) below.
4. **Run the analysis from start to finish** to confirm that it still runs and that any numbers quoted in the written report still match the code output. If your change alters results, update the text and tables that report them.
5. **Commit** your changes with clear messages that say what changed and why:
   ```bash
   git add .
   git commit -m "Fix typo in threshold selection table caption"
   ```
6. **Push** your branch to your fork:
   ```bash
   git push origin fix-threshold-table-typo
   ```
7. **Open a pull request** against the `main` branch of this repository. In the description:
   - Summarize what you changed and why.
   - Link the related issue, if any (for example, "Closes #12").
   - Note whether results or figures changed.
   - Include the AI disclosure described in the [AI policy](#ai-policy), if it applies.
8. **Respond to review.** A maintainer will review your pull request and may ask for changes. Please address comments by pushing additional commits to the same branch. A maintainer will merge the pull request once it is approved.

## Project conventions

- **Do not touch the test set during modelling decisions.** The design of this analysis keeps the held-out test set untouched until the final evaluation. Model selection must use cross-validation on the training set only, and the decision threshold must be chosen from out-of-fold training predictions. Pull requests that use test-set results to make modelling choices will not be accepted.
- **Avoid data leakage.** All preprocessing (imputation, scaling, encoding) must stay inside the scikit-learn pipeline so it is learned from training data only. Do not add columns that are recorded after the booking is made or that reveal the outcome (the report already removes `reservation_status`, `reservation_status_date`, `assigned_room_type`, `booking_changes` and `days_in_waiting_list` for this reason).
- **Keep the analysis reproducible.** Use the fixed random seed (`RANDOM_STATE = 42`) for splits and models. Raw data are downloaded from Kaggle into `data/raw/` and processed data are written to `data/processed/`; do not commit large data files or credentials.
- **Keep text and code in sync.** If you change code that affects a number, table, or figure, update the report text so it matches.
- **Code style.** Follow [PEP 8](https://peps.python.org/pep-0008/), use clear variable names, and add comments where the reasoning is not obvious.
- **Dependencies.** If your change needs a new package, mention it in the pull request and explain why it is needed.

## AI policy

Generative AI tools can be useful, but this project holds contributors fully responsible for what they submit. The rules below apply to all issues, pull requests, comments, code, and documentation.

### What is allowed

You may use generative AI tools as an assistant, for example to:

- Brainstorm ideas or explain unfamiliar concepts or error messages.
- Suggest code, which you then read, test, and edit.
- Improve the wording, grammar, or clarity of text that you wrote.

### What is not allowed

- **AI agents may not act on your behalf in this repository.** Issues and pull requests must be opened, reviewed, and merged by a human, not by an AI agent. Do not use an autonomous agent or bot to open issues, create or push branches, open pull requests, post review comments, or merge changes.
- **Do not submit AI output you do not understand.** If you cannot explain what a piece of code or text does and why it is correct, do not submit it.
- **Do not paste unreviewed AI output** into issues, pull requests, or review comments. Low-effort, bulk, or automatically generated contributions will be closed without review.
- **Do not fabricate.** AI tools can invent results, citations, and links. Every number, reference, and link in your contribution must be checked by you against the actual code output or the original source.
- **Do not share private data** (credentials, API tokens, or non-public information) with AI tools while working on this project.

### What you must disclose

If a generative AI tool contributed substantially to your pull request or issue (for example, it wrote or rewrote code, generated analysis text, or drafted the issue description), say so in the description. State the name of the tool and briefly describe how you used it, for example: "I used Claude to suggest a refactor of the plotting code; I reviewed and tested the result myself." Minor uses such as spell-checking or autocompleting a few characters do not need to be disclosed. If you are unsure, disclose.

### Who is responsible

You are the author and are fully responsible for everything you submit, including its correctness, licensing, and quality, no matter which tools helped produce it. Maintainers may ask you to explain any part of your contribution, and may close a contribution that you cannot explain or that appears to be unreviewed AI output. Reviewers and maintainers are also responsible for their own reviews and decisions; these must not be delegated to an AI agent.

## Attribution

The AI policy section above was written with reference to the AI policies of other open-source projects. We drew on ideas from these sources, and we encourage you to read them:

- [earthaccess AI policy](https://earthaccess.readthedocs.io/en/latest/contributor/ai-policy/)
- [SciPy AI policy](https://scipy.github.io/devdocs/dev/conduct/ai_policy.html)
- [SymPy AI-generated code policy](https://docs.sympy.org/dev/contributing/ai-generated-code-policy.html)

The structure of this document (reporting bugs, suggesting changes, and submitting work through forks and pull requests) follows common open-source practice, as seen in the contributing guides of projects such as [Altair](https://github.com/altair-viz/altair/blob/master/CONTRIBUTING.md) and [dplyr](https://github.com/tidyverse/dplyr/blob/master/.github/CONTRIBUTING.md).

The dataset used in this project is the [Hotel Booking Demand dataset](https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand) on Kaggle; please follow its license and terms of use when contributing.