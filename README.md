# MORF MWE - README
This document maintained by [Alex Tobias](https://github.com/alextobias) (for the [University of Pennsylvania Center for Learning Analytics]((https://www.upenn.edu/learninganalytics/)))

## Introduction (read this first)

This document is to record the steps I took, and hiccups I encountered, in running the MORF MWE. It will help serve as a FAQ/README for new users of MORF, and a resource for the developers of MORF to improve usability.

This guide will walk through my process of setting up the MORF backend locally, then running the MWE. It assumes you have reasonable competency with Python, setting up an environment and installing things with your terminal.

Links:
- [MOOC Replication Framework (MORF) Project Homepage](https://educational-technology-collective.github.io/morf/)
- [MORF MWE](https://github.com/pcla-code/morf-job-mwe)

## Brief Overview

The MORF backend is a Flask application. It's normally hosted on an EC2 instance. Put simply, it accepts jobs, runs them according to the controller script, then emails the results to the submitter.

Jobs are submitted as HTTP requests to the MORF backend. Each 'job' contains a pointer to a Docker file, a controller script which is executed in the docker instance, and other metadata. The MWE is one such instance of a job.

Starting the MORF backend is as simple as running `python3 backend_app.py`.  
Submitting a job can be done via the [MORF API](https://educational-technology-collective.github.io/morf/documentation/#morf-20-api), which exposes a python interface to make job submission simpler.

## 1. Setting up your environment
The OS I used for everything here was **Ubuntu 18.04**.

Since the MORF backend is a Flask application, python is needed. To manage my python environment, I used Miniforge to install conda. That means all packages listed below are from the conda-forge repositories. Conda is not a requirement; if you're experienced with Python then go ahead and manage your packages in whatever way you prefer.

With conda set up, I created a **Python 3.8** environment with the following packages:
```
- Flask
- waitress
- cryptography
- scikit-learn
- pandas
- numpy
- boto3
```

**Note**: if, while trying to run the MORF backend, your interpreter complains about certain packages not being installed, it's probably our mistake that we didn't include it here. Go ahead and install whatever your interpreter complains about until it works.

Aside from python packages, I also needed to install the following utilities/tools:
```
- s3fs-fuse (to 'mount' an s3 bucket as a volume)
- aws-cli (see https://aws.amazon.com/cli/)
```

Lastly, you'll also need to install **Docker**, as the MORF backend runs jobs in a docker container. Instructions for that can be found [here](https://www.docker.com/get-started).

## 2. Setting up directories & file storage

Think about where you'll store or mount the directories containing course data, and where you want any results to be output.

For me, I created a directory `/home/[my_user_name]/morf/backend/` to contain both the course data and output from the MORF backend. I'll be referring to this as `/.../backend/` from this point on.

(This isn't a hard rule, and if you want to change the folders you use then be sure to update the `configs/app.cfg` file accordingly in the morf backend code. Just be sure to use your common sense.)

You'll now need to create the following directories:
- `/.../backend/morf-data/`, which is where you should store your course data.
- `/.../backend/output/`, which is where job output from the MORF backend will go. 
- `/.../backend/submitted_job_files/`, which is where submitted jobs will be kept.


Specifically, for the `morf-data/` directory above, you may want to mount the s3 bucket where the course data is kept. You can do this with `s3fs morf-upenn morf-upenn -o allow_other -d`, though I haven't actually done this myself.

For my own testing purposes (locally, of course), I simply downloaded the relevant course data and placed  it in that `morf-data/` folder. For the MWE, this means the entire `accounting/` course data folder, plus the files `coursera_course_dates.csv`, `labels-test.csv` and `labels-train.csv`.

Now you'll need to point the morf backend to these folders.

In the morf backend directory, navigate to `configs/app.cfg` and change:
- `job_zip_file_storage_path` to point to wherever you created the `submitted_job_files` directory above.
- `morf_data_docker_volume` to point to wherever you created the `output` directory above.
- `job_zip_file_storage_path` to point to wherever you created the `morf-data` directory above.

The reason you need to have created these folders manually is because the morf backend will freak out if the directories don't exist.

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


## Addendum: About MORF
The MOOC Replication Framework (MORF) is being developed to enable the investigation of research questions on MOOCs across multiple data sets. MORF tests whether previously published findings on engagement and completion in MOOCs replicate to new data sets. Currently, MORF has access to over 150 MOOC data sets and is able to test 21 previously published findings on new data sets.
