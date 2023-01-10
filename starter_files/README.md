# Operationalizing Machine Learning

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, I continued to work with the [Bank Marketing](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv) dataset. I used Azure to configure a cloud-based machine learning production model, deploy it, and consume it. Both `Azure ML Studio` and `Python SDK` were used.

## Architectural Diagram
In this project, we use AutoML to train a best model, then operationalize it by following the workflow below. 

The following diagram provides a visual summary of the workflow:
![Workflow](/starter_files/images/Workflow.JPG)

As you can see, the steps were:
1. Authentication: Creation of a Service Principal account and associate it with a specific workspace
2. Automated ML Experiment:  Creation of an experiment using Automated ML, configuration of a compute cluster and using cluster to run the experiment
3. Deploy the best model: Deploying the Best Model will allow to interact with the HTTP API service and interact with the model by sending data over POST requests
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
The Workspace information is stored in the [configuration file](/starter_files/config.json) which is needed in the script. By running [logs.py](/starter_files/logs.py) Application Insights was enabled as you can see here:

![Application Insights Enabled](/starter_files/images/ApplicationInsights_enabled_in_AzureML.png)

The logs can be shown in the console:

![Console logs](/starter_files/images/ApplicationInsights_logs_via_script.png)

Also you can see the logs in AzureML itself:

![AzureML logs](/starter_files/images/ApplicationInsights_logs_in_AzureML.png)

### Swagger Documentation
The [swagger.json file](/starter_files/swagger/swagger.json) of our custom API needs to be stored in the correct repository.

The purpose of the [swagger.sh script](/starter_files/swagger/swagger.sh) is to start a local webserver of swagger-ui. 
![Output swagger.sh](/starter_files/images/Output_swagger_sh.png)

"swagger-ui" is a tool that understands the [swagger.json file](/starter_files/swagger/swagger.json) file format and visualizes the API definition contained in it. The "swagger-ui" webserver is started by default on port 80, but since on the virtual machine provided by the course lab port 80 is already taken (by another program), I modified the [swagger.sh](/starter_files/swagger/swagger.sh) file to use port 9000.

After having started "swagger-ui" by running the swagger.sh script I need to provide my custom, downloaded from azure, [swagger.json file](/starter_files/swagger/swagger.json) to the swagger-ui somehow. This is done by starting a local (again, on the same machine) python server using the [serve.py script](/starter_files/swagger/serve.py) in another terminal. This script makes it possible to access the swagger.json file over the HTTP protocol, expected by the "swagger-ui" tool, i.e. on http://localhost:8000/swagger.json. The python webserver does nothing more than serving local files, so in addition to http://localhost:8000/swagger.json it is serving also http://localhost:8000/serve.py for example.
The Swagger inferface looks as follows:

![Swagger Interface](/starter_files/images/Swagger_interface.png)

You can see the `score` method with an example input:

![Swagger score](/starter_files/images/Swagger_score.png)


### Consume model endpoints
The endpoint can be consumed by [script](/starter_files/endpoint2.py) after ensuring the authentication. I passed one data sample and received the following answer:

![Output endpoint2.py](/starter_files/images/Output_endpoint_py.png)

### Create and publish a pipeline
First, a compute instance for the [Jupyter Notebook](/starter_files/aml-pipelines-with-automated-machine-learning-step.ipynb) needs to be created. The notebook was modified and run through the cells.
Also for that step, the [Bankmarketing dataset](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv) was used:

![Dataset](/starter_files/images/Dataset_available.png)

Via the Python SDK, an AutoML pipeline was created and published:

![Pipeline Overview](/starter_files/images/Jupyter_pipeline_created_and_published.png)

After publishing, you could see the pipeline endpoint in the Azure ML workspace:

![Pipeline Endpoint](/starter_files/images/Jupyter_pipeline_endpoint.png)

The Pipeline Overview shows the pipeline endpoint to be ACTIVE and accessible via REST endpoint:

![Published Pipeline Overview](/starter_files/images/Jupyter_published_pipeline_overview.png)

The progress could be visualized in the notebook via RunDetails:

![RunDetails](/starter_files/images/Jupyter_RunDetails.JPG)


## Screen Recording
*TODO* Provide a link to a screen recording of the project in action. Remember that the screencast should demonstrate:

## Standout Suggestions
Some suggestions to the Udacity mentors from my side:
1. The enpoint requests via script needs to hand over the data input in a very specific way. That was not updated in the classroom. Refer to my file [endpoint2.py](/starter_files/endpoint2.py)
2. The RunDetails widget seems to have problems when trying to visualize in the Jupyter Notebook.