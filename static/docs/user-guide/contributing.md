# Contributing

We welcome contributions to [DVC](https://github.com/iterative/dvc) by the
community. See the
[Contributing to the Documentation](/doc/user-guide/contributing-docs) guide if
you want to fix or update the documentation or this website.

## How to report a problem

Please search [issue tracker](https://github.com/iterative/dvc/issues) before
creating a new issue (problem or an improvement request). Feel free to add
issues related to the project.

For problems with [dvc.org](https://dvc.org/) site please use this
[GitHub repository](https://github.com/iterative/dvc.org/).

If you feel that you can fix or implement it yourself, please read a few
paragraphs below to learn how to submit your changes.

## Submitting changes

- Open a new issue in the
  [issue tracker](https://github.com/iterative/dvc/issues).
- Setup the [development environment](#development-environment) if you need to
  run tests or [run](#running-development-version) the DVC with your changes.
- Fork [DVC](https://github.com/iterative/dvc.git) and prepare necessary
  changes.
- Add tests for your changes to `tests/test_*.py`. You can skip this step if the
  effort to create tests for your change is unreasonable. Changes without tests
  are still going to be considered by us.
- [Run tests](#running-tests) and make sure all of them pass.
- Submit a pull request, referencing any issues it addresses.

We will review your pull request as soon as possible. Thank you for
contributing!

## Development environment

Get the latest development version. Fork and clone the repo:

```dvc
$ git clone git@github.com:<username>/dvc.git
```

Make sure that you have Python 3 installed. Version 3.6 or higher is required to
run style checkers on pre-commit. On Mac OS, we recommend using `brew` to
install Python.

Install DVC in editable mode with `pip install -e ".[all,tests]"`. We
**strongly** recommend initializing a
[virtual environment](https://virtualenv.pypa.io/en/latest/userguide/) before
doing so:

```dvc
$ cd dvc
$ virtualenv --python python3 .env
$ source .env/bin/activate
$ pip install -e ".[all,tests]"
```

> Note that the double quotes (`""`) used to wrap the dependencies to be
> installed are only compulsory when using `zsh`. In `bash`, installation
> commands can work without them: `pip install -e .[all,tests]`.

Install coding style pre-commit hooks with:

```dvc
$ pip install pre-commit
$ pre-commit install
```

That should be it. You should be ready to make changes, run tests do commits! If
you experience any problems, please don't hesitate to ping us in our
[chat](/chat).

## Running tests

The simplest way to run tests:

```dvc
$ cd dvc
$ python -m tests
```

This uses `pytest` to run the full test suite and report the result. At the very
end you should see something like this:

```dvc
$ python -m tests

...

============= 434 passed, 6 skipped, 8 warnings in 131.43 seconds ==============
```

Otherwise, for each failed test you should see the following output, to help you
identify the problem:

```
$ python -m tests

...

[gw2] [ 84%] FAILED tests/unit/test_progress.py::TestProgressAware::test
tests/unit/test_prompt.py::TestConfirm::test_eof
tests/test_updater.py::TestUpdater::test
...
=================================== FAILURES ===================================
____________________________ TestProgressAware.test ____________________________
...
======== 1 failed, 433 passed, 6 skipped, 8 warnings in 137.49 seconds =========
```

You can pass any additional arguments to the script that will override the
default `pytest`'s scope:

To run a single test case:

```dvc
$ python -m tests tests/func/test_metrics.py::TestCachedMetrics
```

To pass additional arguments:

```dvc
$ python -m tests --pdb
```

## Testing remotes

This tests require additional effort to set up, so they are skipped by default.
However, running code tests locally may be necessary when changing core source
code.

Install requirements for whatever remotes you are going to test:

```dvc
$ pip install -e ".[s3]"
$ pip install -e ".[gs]"
$ pip install -e ".[azure]"
$ pip install -e ".[ssh]"
# or
$ pip install -e ".[all]"
```

You will need to update your environment throughout subsequent steps, so we
created a template for you:

```dvc
$ cp tests/remotes_env.sample tests/remotes_env
```

Then uncomment lines you need and fill in needed values, the details are
explained in remote specific subsections. To activate these environment
variables, use:

```dvc
$ source tests/remotes_env
```

If one of your colleagues has already gone through this guide, you could just
ask for their `remotes_env` file and Google Cloud credentials, and skip any env
manipulations below.

<details>

### Click for S3 testing instructions

Install
[aws cli](https://docs.aws.amazon.com/en_us/cli/latest/userguide/cli-chap-install.html)
tools.

To set up AWS access, first get credentials with S3 permissions. Then, set env
vars like this:

```dvc
$ export AWS_ACCESS_KEY_ID="...YOUR-ACCESS-KEY-ID..."
$ export AWS_SECRET_ACCESS_KEY="...YOUR-SECRET-ACCESS-KEY..."
$ export DVC_TEST_AWS_REPO_BUCKET="...TEST-S3-BUCKET..."
```

</details>

<details>

### Click for Google Cloud Storage testing instructions

Go through the [quick start](https://cloud.google.com/sdk/docs/quickstarts) for
your OS. After that you should have `gcloud` command line tool available and
authenticated with your google account.

You then need to create a bucket, a service account and get its credentials. You
can do this via web UI or terminal. Then you need to put your keys to
`scripts/ci/gcp-creds.json` and add these to your env vars:

```dvc
$ export GOOGLE_APPLICATION_CREDENTIALS=".gcp-creds.json"
$ export GCP_CREDS="yes"
$ export DVC_TEST_GCP_REPO_BUCKET="dvc-test-xyz"
```

Here are some command examples to do this:

```dvc
# This name needs to be globally unique
$ export GCP_NAME="dvc-test-xyz"
$ gcloud projects create $GCP_NAME
$ gcloud iam service-accounts create $GCP_NAME --project=$GCP_NAME
$ gcloud iam service-accounts keys create \
    scripts/ci/gcp-creds.json \
    --iam-account=$GCP_NAME@$GCP_NAME.iam.gserviceaccount.com

$ gcloud auth activate-service-account \
    --key-file=scripts/ci/gcp-creds.json
$ gcloud config set project $GCP_NAME
$ gsutil mb gs://$GCP_NAME/
```

I used the same name for project, service account and bucket for simplicity. You
may use different names.

</details>

<details>

### Click for Azure testing instructions

Install [Node.js](https://nodejs.org/en/download/) and then install and run
Azurite:

```dvc
$ npm install -g 'azurite@<3' # Need 2.x version
$ mkdir azurite
$ azurite -s -l azurite -d azurite/debug.log
```

Add this to your env:

```dvc
$ export AZURE_STORAGE_CONTAINER_NAME="dvc-test"
$ export AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;"
```

</details>

<details>

### Click for HDFS testing instructions

Tests currently only work on Linux. First you need to set up passwordless ssh
access to localhost:

```dvc
# Only run next line if you don't yet have keys
$ ssh-keygen -t rsa -P ""
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

Then run an install script:

```dvc
$ ./scripts/ci/install_hadoop.sh
```

To remove hadoop later use:

```dvc
$ ./scripts/ci/remove_hadoop.sh
```

</details>

## Code style guidelines (for Python)

We are using [PEP8](https://www.python.org/dev/peps/pep-0008/?) and checking
that our code is formatted with [black](https://github.com/ambv/black).

For [docstrings](https://www.python.org/dev/peps/pep-0257/#what-is-a-docstring),
we try to adhere by the
[Google Python Style Guide](https://github.com/google/styleguide/blob/gh-pages/pyguide.md#38-comments-and-docstrings).

## Commit message format guidelines

Format:

```
(component): (short description)

(long description)

Fixes #(GitHub issue id).
```

Message types:

- _component_: Name of a component that this patch is affecting. Use `dvc` in a
  general case
- _short description_: Short description of the patch
- _long description_: If needed, longer message describing the patch in more
  details
- _github issue id_: ID of the GitHub issue that this patch is addressing

Example:

```
remote: add support for Amazon S3

Fixes #123
```
