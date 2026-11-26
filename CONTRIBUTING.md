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

Team members use branches in the shared project repository, not
personal forks. Each change must go through a pull request reviewed
and approved by at least one other team member before merging.

### Initial setup

Follow [How to run the analysis](README.md#how-to-run-the-analysis)
in the README first. It covers installing the prerequisites,
cloning the repository, preparing the SSH key and Git configuration,
and downloading the Docker image.

Install [Positron](https://positron.posit.co/download.html).
Remote SSH is included in Positron.

Check your Git identity on your computer before starting the container:

```bash
git config --global user.name
git config --global user.email
```

If either value is missing, follow the Git configuration instructions
in the README. The container mounts your Git configuration so that
commits are attributed to you.

### Create a task branch

From the repository root on your computer, start with a clean
working tree and update main:

```bash
git switch main
git pull --ff-only origin main
git switch -c describe-your-change
```

Replace `describe-your-change` with a meaningful branch name, such
as `improve-report-captions`. Keep each branch focused on one task.

### Work in the container using Positron

We develop in the same Docker environment used to run the analysis.
From the repository root on your computer, start the container:

```bash
docker compose up -d
docker compose ps
```

Confirm that the `analysis-env` service is running.

In Positron, open the Command Palette with `Ctrl+Shift+P` on
Windows or `Cmd+Shift+P` on macOS. Run
**Remote SSH: Show Remote Menu**, choose **Connect to Host**,
and enter:

```text
root@localhost:2222
```

If prompted for the remote platform, choose Linux. For the expected
local container, confirm the host fingerprint when prompted.
The first connection may take several minutes while Positron
installs its remote server.

In the new remote window, select **Open Folder** and open:

```text
/workspace
```

This folder is mounted from your local project directory.
Edits inside `/workspace` also change the files on your computer.

Select the project Python interpreter from the interpreter menu.
Use **New Console Session** and choose the interpreter at:

```text
/opt/venv/bin/python
```

If it is not listed, use **Python: Select Interpreter** from the
Command Palette, select **Enter interpreter path**, and enter
that path. Do not select `/usr/bin/python3` or `/workspace/.venv`.

Check the interpreter in the Python Console:

```python
import sys
print(sys.executable)
```

The output should be `/opt/venv/bin/python`.

Edit and run the project in this remote window. To render the
HTML report, run the following in its terminal from `/workspace`:

```bash
uv run quarto render analysis/main_analysis.qmd --to html
```

Alternatively, from the repository root in a terminal on your
computer, render without opening an SSH session:

```bash
docker compose run --rm analysis-env uv run quarto render analysis/main_analysis.qmd --to html
```

After finishing, close the remote Positron window and stop the
container from a terminal on your computer:

```bash
docker compose down
```

### Troubleshoot the connection

If Docker cannot connect to its engine, start Docker Desktop and
wait until the engine is running.

If SSH asks for a password, check that
`~/.ssh/id_ed25519.pub` is a file and that its path matches the
mount in `docker-compose.yml`. The key used by your SSH client
must correspond to this public key.

If `~/.ssh/id_ed25519.pub` or `~/.gitconfig` is a directory,
Docker may have created it because the source file was missing.
Stop the container with `docker compose down`. Inspect the
directory before removing it, then create the correct file using
the README instructions and start the container again.

If a host identification warning appears after intentionally
recreating the local container, remove the old local host entry:

```bash
ssh-keygen -R "[localhost]:2222"
```

Reconnect and confirm the new fingerprint for the expected
local container.

If port 2222 is already occupied, choose an unused host port in
`docker-compose.yml`, such as `"2223:22"`, and connect to
`root@localhost:2223`.

If GitHub authentication is requested when pushing, complete
the sign-in process yourself. Never put credentials in project
files or issues.

### Add or update dependencies

Make dependency changes on a task branch inside the container.
Use `uv add` with an exact version specification. For example,
replace the package name and version in the following command
with the dependency you actually need:

```bash
uv add "package-name==X.Y.Z"
```

Commit the changed `pyproject.toml` and `uv.lock` together.
Include `.python-version` if the Python version also changes.
Explain the reason for the dependency change in the pull request.

After pushing, wait for the **Publish image to Docker Hub**
GitHub Actions workflow to finish successfully. It publishes
the updated image and commits its new SHA tag to
`docker-compose.yml` on the same branch.

Save and commit your work, then close the remote Positron window.
In the repository root on your computer, retrieve the automated
commit and recreate the container using the new image:

```bash
git pull --ff-only
docker compose pull
docker compose down
docker compose up -d
```

Reconnect Positron and verify the updated environment. Submit
the branch for review.

After the pull request is merged and the workflow on main
finishes, every team member should update their clean local main
branch and download the image again:

```bash
git switch main
git pull --ff-only origin main
docker compose pull
```

Recreate any running container before continuing development
with the new image.

### Update the published report

Edit the source report at `analysis/main_analysis.qmd`, rather
than editing generated HTML directly.

Keep the HTML settings `embed-resources: true` and
`format-links: false` so the published copy includes its figures
and does not contain broken links to other output formats.

After rendering the HTML report, copy it to the publication
directory from the repository root:

```bash
mkdir -p docs
cp analysis/main_analysis.html docs/index.html
touch docs/.nojekyll
```

Open `docs/index.html` in a browser and check the text, tables,
and figures. Commit the report source, rendered report, and
publication copy together when they change.

GitHub Pages must be configured to deploy from the `main` branch
and `/docs` folder. Once that setup is in place, the published
copy updates after the reviewed changes are merged into main.

### Commit, push, and request review

Review your changes and stage only the intended files. For a
report update, for example:

```bash
git diff
git add analysis/main_analysis.qmd analysis/main_analysis.html docs/index.html docs/.nojekyll
git diff --cached --stat
git commit -m "Update report and published HTML"
git push -u origin HEAD
```

Use file paths and a commit message appropriate to your actual
task. Do not commit `.venv`, credentials, caches, or unrelated
generated files.

Open a pull request into main yourself. Describe what changed,
link the related issue, explain how you checked the work, and
include any disclosure required by the AI policy below.

For analysis changes, render the report in the container and
confirm that reported results match the output. For documentation
changes, check links and verify the affected instructions.

Address review comments by adding commits to the same branch.
At least one other team member must review and approve the pull
request before it is merged. Issues, pull requests, reviews,
and merging are handled by people.

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

The container development instructions were adapted from the
[DSCI 522 Milestone 2 guide](https://ubc-mds.github.io/DSCI_522_dsci-workflows_students/assignments/milestone2.html)
and the

[Breast Cancer Predictor contributing guide, version 1.0.0](https://github.com/UBC-MDS/DSCI_522_project-template-breast-cancer-predictor/blob/1.0.0/CONTRIBUTING.md),
with paths and commands adapted for this project.