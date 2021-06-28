# MORF MWE - README
This document maintained by [Alex Tobias](https://github.com/alextobias) (for the [University of Pennsylvania Center for Learning Analytics]((https://www.upenn.edu/learninganalytics/)))

## Introduction (read this first)

This document is to record the steps I took, and hiccups I encountered, in running the MORF MWE. It will help serve as a FAQ/README for new users of MORF, and a resource for the developers of MORF to improve usability.

This document will walk through my process of setting up the MORF backend locally, then running the MWE.

Links:
- [MOOC Replication Framework (MORF) Project Homepage](https://educational-technology-collective.github.io/morf/)
- [MORF MWE](https://github.com/pcla-code/morf-job-mwe)

## Brief Overview
The MORF backend is a Flask application. It's normally hosted on an EC2 instance. Put simply, it accepts jobs, runs them according to the controller script, then emails the results to the submitter.

Jobs are submitted as HTTP requests to the MORF backend. Each 'job' contains a pointer to a Docker file, a controller script which is executed in the docker instance, and other metadata. The MWE is one such instance of a job.

Starting the MORF backend is as simple as running `python3 backend_app.py`.  
Submitting a job can be done via the MORF API, which exposes a python interface to make job submission simpler.

## 1. Setting up your environment
The OS I used for everything here was Ubuntu 18.04.

Since the MORF backend is a Flask application, python is needed. To manage my python environment, I used Miniforge to install conda. That means all packages listed below are from the conda-forge repositories. If you're experienced with Python then go ahead and manage your packages in whatever way you prefer.

With conda set up, I created a Python 3.8 environment with the following packages:
```
- TODO: list missing python-specific packages
- Flask
- waitress
- cryptography
- scikit-learn
- pandas
- numpy
```

Aside from python packages, I also needed to install the following (which I also installed via conda)

```
- TODO: list missing other packages
- boto3 (for AWS mailing)
- s3fs-fuse (to 'mount' an s3 bucket as a volume)
```

## 2. Setting up directories
Think about where you'll store or mount the directories containing course data, and where you want any results to be output.

// todo: explain how the input/output directories are structured, and what folders need to be created

## 3. Setting up authentication and API keys
With all the above packages installed, you'll have to manually modify the config files for things like secrets and private keys.

// TODO: add more details on secret key generation/key pair generation/api key stuff  
// also make note about amazon SES for email

## 4. Running the backend
Now you should be ready to run the backend. Navigate to the directory where your backend code is stored.  
Run `python3 backend_app.py`.

You may want to considering add a dummy 'hello world' GET route just to make sure that the server is functioning and accepting HTTP requests. 

## 5. Submitting the MWE
The MWE can be found at https://github.com/pcla-code/morf-job-mwe.

This submission part of the process is what the MORF API is intended for. If you know how to manage your own python environment, you can probably go ahead and install the library yourself (https://pypi.org/project/morf-api/).

For my own purposes, I wanted to keep everything in a conda environment, and the morf-api package is not available in conda-forge. My workaround is as follows:

I accessed the morf-job-api source from https://github.com/pcla-code/morf-job-api. There, in `morf-job-api/morfjobapi/morf_job_utils.py`, there's a function `submit_job`, which contains the logic for submitting the specified job to the specified morf api endpoint. I then imported this function into a jupyter notebook which I used for job submission (but you don't have to use a notebook, the point is that this function is what you should look at).

Remember that you'll also need to specify your generated API key, as described above (TODO)


## About MORF
The MOOC Replication Framework (MORF) is being developed to enable the investigation of research questions on MOOCs across multiple data sets. MORF tests whether previously published findings on engagement and completion in MOOCs replicate to new data sets. Currently, MORF has access to over 150 MOOC data sets and is able to test 21 previously published findings on new data sets.
