# MORF - Writing a Job
This document maintained by [Alex Tobias](https://github.com/alextobias) (for the [University of Pennsylvania Center for Learning Analytics]((https://www.upenn.edu/learninganalytics/)))

## Document Purpose (read this first)
Are you a researcher interested in applying machine learning to large educational data sets from massive open online courses (MOOCs)? If so, you are in the right place.

This document is to walk a user through the steps needed to submit a job to MORF. It goes over what a job is, what the components are, and what is neccessary to tell MORF to construct a model and run a job.

To do this, it walks the user through the simple minimum working example (MWE), which can be found here: [MORF MWE](https://github.com/pcla-code/morf-job-mwe)

## Overview
MORF is a platform where researchers can construct and evaluate predictive models from raw MOOC platform data. 

The MORF backend is a Flask application. It's normally hosted on an EC2 instance. Put simply, it accepts jobs, runs them according to the controller script, then emails the results to the submitter.

Jobs are submitted as HTTP requests to the MORF backend. Each 'job' contains a pointer to a Docker file, a controller script which is executed in the docker instance, and other metadata. The MWE is one such instance of a job.

If you would like to run an instance of the MORF backend yourself, [this document](Running_the_MORF_backend.md) contains what you need to know. 

The main steps to executing a job on MORF are (this is straight from the documentation):

- Write code to extract features from raw platform data and build predictive models on the results of that feature extraction.
- Write a high-level “controller script” using the MORF Python API.
- Build a Docker image containing your code and any software dependencies with the appropriate control flow.
- Create a configuration file containing identifiers and links to your controller script and docker image.
- Upload the the controller script, docker image, and configuration file to public locations (you need to provide either HTTPS or S3 URLs to these files).
- Submit your job to the MORF web API. You will receive notifications and results via email as your job is queued, initiated, and completed.

This document will walk you through each of these steps in the context of the MORF MWE (minimum working example).

## Platform Data

When a job is submitted to MORF, it is able to extract data that the MORF backend has access too.

MORF's data currently consists of several datasets from Coursera. The data policy can be found [here](https://docs.google.com/document/d/1oZNkaPDRG0wgJTcIS7viqE-KhENr4skhOQ3vc9s_xSw/edit#heading=h.gjdgxs).

A very thorough explanation from Coursera on the way data is exported and structured can be found here: [https://spark-public.s3.amazonaws.com/mooc/data_exports.pdf](https://spark-public.s3.amazonaws.com/mooc/data_exports.pdf) (If the link is broken, search for "Coursera Data Export Procedures")

The following is another excellent resource on the database schema: [https://wiki.illinois.edu/wiki/display/coursera/Coursera+Session-Based+Courses+SQL+and+Eventing+Data+Tables+Documentation](https://wiki.illinois.edu/wiki/display/coursera/Coursera+Session-Based+Courses+SQL+and+Eventing+Data+Tables+Documentation)

Here are some key things that you should know about the way data is stored and accessed in MORF:

### Directory Structure and files
The following is an example of how a course data directory is structured in our backend, which is identical to how it's given in Coursera's "Data Exports" document above.

- `accounting` (course name)
    - `001` (session)
        - `accounting-001_clickstream_export.gz` (holds the clickstream data in JSON format)
        - `An Introduction to Financial Accounting (accounting-001)_Demographics_individual_responses.csv` 
            - This holds self-reported demographic information about users such as location, gender, etc.
        - `An Introduction to Financial Accounting (accounting-001)_Demographics_summary.html` 
            - This is a browser-viewable HTML visualization of demographic survey responses - you will likely not need to use this.
        - `An Introduction to Financial Accounting (accounting-001)_SQL_anonymized_forum.sql.gz`
            - Contains all data related to the course forum's threads, posts and comments. 
        - `An Introduction to Financial Accounting (accounting-001)_SQL_anonymized_general.sql.gz` (almost everything else in the course - see notes in following section)
            - This database will contain almost everything else in the course, for example: access groups, announcements, grades, lecture metadata, quiz metadata, and users
        - `An Introduction to Financial Accounting (accounting-001)_SQL_hash_mapping.sql.gz` 
            - A single table linking anonymized user ids across tables in the other datasets)
        - `An Introduction to Financial Accounting (accounting-001)_SQL_unanonymizable.sql.gz` 
            - contains misc. unanonymizable data, not very useful)

The clickstream data is stored in JSON format. 

The other files (with the exception of the HTML file) are flat SQL files, so they  contain not only the data itself, but also the SQL commands needed to set up the tables, import the data, etc. Running them in MySQL will automatically set everything up for you. 

*Because the data is stored in these flat JSON/SQL files, the way you'll need to interact with the data job looks like this:*
- Use the dockerfile to specify that MySQL should be installed in the docker container that will be created for running your job.
- In your job script, import the SQL files into the MySQL instance that runs inside your docker container.
- In your job script, run SQL queries to obtain the data needed for your feature extraction.

We will go into more information on this, later in the document. 
By doing this, the MORF backend itself doesn't need to run a MySQL instance - each job has its own MySQL instance in its docker container.

