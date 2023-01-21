# Operationalizing Machine Learning

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, I continued to work with the [Bank Marketing](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv) dataset. I used Azure to configure a cloud-based machine learning production model, deploy it, and consume it. Both `Azure ML Studio` and `Python SDK` were used.

## Architectural Diagram
In this project, I used AutoML to find and train a best model, then operationalize it by following the workflow below. 

The following diagram provides a visual summary of the workflow (source: Udacity classroom):
![Workflow](/starter_files/images/Workflow.JPG)

As you can see, the steps were:
1. Authentication: Creation of a Service Principal account and association it with a specific workspace
2. Automated ML Experiment:  Creation of an experiment using Automated ML, configuration of a compute cluster and using cluster to run the experiment
3. Deploy the best model: Deploying the Best Model will allow to interact with the HTTP API service and interaction with the model by sending data over POST requests
4. Enable logging: Enabling Application Insights and retrieving logs
5. Swagger Documentation: Consumption of the deployed model using Swagger
6. Consume model endpoints: Consumption of the deployed model via script
7. Create and publish a pipeline: Automated creation and publishing a pipeline with the Python SDK

What was done in each step is described in detail in the next subchapter.

## Key Steps
The results of the steps are described and visualized in the following.

### Authentication
In this step, the Azure Machine Learning Extension was used which allows you to interact with Azure Machine Learning Studio and to create a Service Principal (SP) to access the project workspace. Since I used the lab environment provided by Udacity I had no sufficient privilege to create the SP, so this step was not performed.

### Automated ML Experiment
The [dataset](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv) was added to the Azure ML workspace:

![Dataset](/starter_files/images/Dataset_available.png)

A computing resource was created to run the experiment:

![AutoML Compute Cluster](/starter_files/images/AutoML_compute_cluster.png)

Based on the dataset the classification task was executed via AutoML:

![AutoML Completion](/starter_files/images/AutoML_completed.png)

A few models were automatically generated:

![AutoML Models](/starter_files/images/AutoML_models.png)

The best models based on the `VotingEnsemble` algorithm reaching an AUC of 94.77%:

![AutoML Best Model](/starter_files/images/AutoML_best_model.png)

### Deploy the best model
The best model was deployed including Authentication and by using Azure Container Instance (ACI). The configuration is shown below:

![Deploy Config](/starter_files/images/Deploy_config.png)

### Enable logging
The Workspace information is stored in the [configuration file](/starter_files/config.json) which is needed in the script. By running [logs.py](/starter_files/logs.py), Application Insights was enabled as you can see here:

![Application Insights Enabled](/starter_files/images/ApplicationInsights_enabled_in_AzureML.png)

The logs can be shown in the console:

![Console logs](/starter_files/images/ApplicationInsights_logs_via_script.png)

Also, you can see the logs in the Azure ML Workspace:

![AzureML logs](/starter_files/images/ApplicationInsights_logs_in_AzureML.png)

### Swagger Documentation
The [swagger.json file](/starter_files/swagger/swagger.json) of my custom API needs to be stored in the correct repository.

The purpose of the [swagger.sh script](/starter_files/swagger/swagger.sh) is to start a local webserver of swagger-ui. 

![Output swagger.sh](/starter_files/images/Output_swagger_sh.png)

"swagger-ui" is a tool that understands the [swagger.json file](/starter_files/swagger/swagger.json) file format and visualizes the API definition contained in it. The "swagger-ui" webserver is started by default on port 80, but since on the virtual machine provided by the course lab port 80 is already taken (by another program), I modified the [swagger.sh](/starter_files/swagger/swagger.sh) file to use port 9000.

After having started "swagger-ui" by running the swagger.sh script I need to provide my custom [swagger.json file](/starter_files/swagger/swagger.json) downloaded from Azure to the swagger-ui somehow. This is done by starting a local Python server via using the [serve.py script](/starter_files/swagger/serve.py) in another terminal. This script makes it possible to access the swagger.json file over the HTTP protocol, expected by the "swagger-ui" tool, i.e. on http://localhost:8000/swagger.json. The python webserver does nothing more than serving local files, so in addition to http://localhost:8000/swagger.json it is serving also http://localhost:8000/serve.py for example.
The Swagger inferface looks as follows:

![Swagger Interface](/starter_files/images/Swagger_interface.png)

You can see the `score` method with an example input:

![Swagger score](/starter_files/images/Swagger_score.png)


### Consume model endpoints
The endpoint can be consumed by [script](/starter_files/endpoint2.py) after ensuring the authentication. I passed one data sample and received the following answer:

![Output endpoint2.py](/starter_files/images/Output_endpoint_py.png)

### Create and publish a pipeline
First, a compute instance for the [Jupyter Notebook](/starter_files/aml-pipelines-with-automated-machine-learning-step.ipynb) needs to be created. The notebook was modified and run through the cells.
Also for this step, the [Bankmarketing dataset](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv) was used:

![Dataset](/starter_files/images/Jupyter_dataset_available_resubmit.png)

Via the `Python SDK`, an AutoML pipeline was created and published:

![Pipeline Overview](/starter_files/images/Jupyter_pipeline_created_and_published_resubmit.png)

After publishing, you could see the pipeline endpoint in the Azure ML workspace:

![Pipeline Endpoint](/starter_files/images/Jupyter_pipeline_endpoint_resubmit.png)

The Pipeline Overview shows the pipeline endpoint to be ACTIVE and accessible via REST endpoint:

![Published Pipeline Overview](/starter_files/images/Jupyter_published_pipeline_overview_resubmit.png)

The progress could usually be visualized in the notebook via RunDetails:

![RunDetails](/starter_files/images/Jupyter_RunDetails_resubmit.JPG)

There is a bug with the extension azureml.widgets at the moment. Even after uninstalling, clearing the kernel and installing again, I could not see the output. This is discussed in the [classroom](https://knowledge.udacity.com/questions/947516?utm_campaign=ret_600_auto_ndxxx_knowledge-answer-created_na&utm_source=blueshift&utm_medium=email&utm_content=ret_600_auto_ndxxx_knowledge-answer-created_na&bsft_clkid=740d20b7-b7a4-4f2c-a4e7-72c4ccf533fb&bsft_uid=5c9b5725-f4bc-4a88-94a8-2b0fe40ba6e5&bsft_mid=d0eb1471-65c1-46d3-b39a-f2067ace5511&bsft_eid=22b8f7b6-5eac-66ee-cf9f-0d5b86b9fddc&bsft_txnid=5d771386-1ecb-4eae-874d-b8542df4affa&bsft_mime_type=html&bsft_ek=2023-01-16T18%3A10%3A13Z&bsft_aaid=8d7e276e-4a10-41b2-8868-423fe96dd6b2&bsft_lx=1&bsft_tv=1#948103)

The ML studio is showing the scheduled run:

![Sheduled Run](/starter_files/images/Jupyter_sheduled_run_resubmit.png)


## Screen Recording
A screencast showing the entire process of the implemented ML application, including interactions with the deployed model and published pipeline endpoints is available via the following link:
https://youtu.be/PyJNvO2ogzU


## Future Improvement Suggestions
1. The accuracy of the model (94.77%) is quite good but can be improved by solving the data imbalance in dataset.
2. The Deep Learning option can be tried when training the model which is not used by the  AutoML in the runs.
3. GPU instead of CPU could be used for faster training.
