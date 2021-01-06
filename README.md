# AzureMLcapstone
This is the repository for Azure Machine learning Nanodegree.
## Project Main Goal<br>
The main goal of the project is to utilize Azure Machine Learning Hyper Drive and Auto Ml capabilities to build a machine learning model and deploy the best model based
based on the performance evaluation metric.<br><br>
The Project architecture can be as below<br>
![Complete Architecture of the Project](architecture.png)

## Business Problem and Dataset <br>
Diabetes is on eof the diseases  suffered by many people across the globe in recent years. Many scientists are trying to find a permanent cure for this but could not be able to do
it. So I wanted to build a machine learning model with the clinical data available and predict whether a patient is having Diabetes or not based on the symptoms.<br>
## Dataset 
I have used the dataset from Kaggle which can be found here . [https://www.kaggle.com/kumargh/pimaindiansdiabetescsv](DiabetesDataset)<br>
## Dataset Description <br>
1. Pregnancy      : Number Of Pregnacies happend <br>
2. Glucose        : Blood Glucose Levels <br>
3. Blood Pressure : Blood Pressure Levels <br>
4. Skin ThickNess : Triceps Skin Thickness<br>
5. Insulin        : Blood Insulin Levels<br>
6. BMI            :Body Mass Index<br>
7. Diabetes       :Diabetes Function<br>
8. Age            : Age Of Patient in Years<br>
9. Outcome        : 1 0r 0 to indicate whether a patient has diabetes or Not<br>
## Machine Learning Model for the Problem statement<br>
The data has lot of outliers, some have irrelavant values. All the preprocessing was done and placed in train.py using clean data function. After that I choose Logistic Regression Model to build  as the Problem is binary problem and choosen accuracy as primary metric which needs to be maximised.<br>
## Dataset into Azure <br>
Since this is an external dataset we need to register the dataset by going to create a new dataset in Azure Machine learning studio, also we can do this by using python sdk <br>
I have tested both ways. Once Dataset is registered we need to convert the dataset to Tabular dataset using Tabular Dataset factory module of Azure Machine learning.<br>

## Azure ML Hyper Drive <br>
Since the model is Logistic regression I have choosen Inverse Regularisation factor (--C)  which penalises the model to prevent over fitting and maximum number of iteration(--Max_iter) as other Hyperparameter to be tuned.<br>
**Sampling Used** I have choosen Random parameter sampling to tune the Hyper parameters to save the computational cost and time of completion.<br>
**Early STopping Technique** I have choosen Bandit policy as it terminates the model building if any of the conditions like slack factor, slack amout, delay interval are not met as per prescribed limits during maximizing the accuracy.<br>
We need to pass the hyperparameter using an entry scriot in my case its train.py file having all the preprocessing steps and Hyperparameters using arg.parser method to be passed to Hyper Drive.<br>
Once we pass the Hyper parameters using train.py  and submit the Job Hyper Drive will create number of jobs based on the parameters given in Hyperdrive configuration using the combinations of Hyper parameters . After running all the 20 models we find the best model and  register it in the portal.<br>
We can see the Hyper Drives running as below.<br>
![HyperDriveRundetails](HyperdriveRunDetails.PNG)
![HyperDriveRuncompletinlogs](hdriveruncompletion.PNG)
Once the Run details are completed we fetch the best run .<br>
We will regsiter the model for future use.<br>
![HdrivemodelRegsitered](hdrivemodelregpy.PNG)<br>
The Registerd model can be seen under models in Azure Machine learning studio ,  a new version of the model is registered everytime we run register. So we have two models registerred.<br>

![ModelRegistered](hdrivemodelregisteredv1&v2.PNG) 
![Hdrivebestparams](Hdrivebestrunmetrics.PNG)
![Hdrivebestrunparams](hdrivebestrunparams.PNG)<br>
This shows the successful completion of Hyperdrive model Running along with best parameters and accuracy.<br>
## Compute Cluster<br>
To run the Jobs we need a target cluster to be created before starting the process whcih can be found in both the notebooks . <br>
## Auto Ml <br>
Hyper Drive run needs lot of preprocessing technqiues for the successful building of a model. Azure has AUTOML capalbilities with less preprocessing techniques reuqired . Here we are going to  build a automl model for our problem. The Dataste is registered and converted to Tabular Dataset using Tabular dataset Factory module.<br>
The AutomL config can be as seen below.<br>
![AutoMLConfig](automlconfig.PNG)<br>

Since the probelm is classification we choose task as classification.<br>
TimeOut is set to 30 minutes sicne the dataset is only 800 rows approximately.<br>
Primary metric is accuracy as we are trying to maximise the accuracy.<br>
label column is the column we are trying to predict here outcome .<br>
Compute target is the target cluster where the computation needs to be done<br>
N_cross_Validations=5 the number of k fold cross validations, since the dataset is small choosen 5 <br>
Iterations: Number of iterations to be run 24 , so this checks 24 automl models<be>
Max_concurernt_iterations: 8 number of parallel runs at a time, choosing this too high impact performance so choosen 8<br>
Once we write the config file and submit the experiment it starts building models whcih can be seen below.<br>
 ![AutoMLRundetails](automlmodelrunstatus.PNG)<br>
  Once the run completes we find the best model and register that.<br>
  ![AutomLBestModel](automlbestrunpy.PNG)<br>
After all the Runs AutomL gave voting ensemble model as best model with accuracy of 78.39 better than HYperdrive model.**VotingEnsemble** model works on taking the majority voting of underlying models and choose the model with highest votes as best model.<br>
 The Fitted Model Parameters are as shown below<br>
  ![Model Parameters](automlfittedmodelparams.PNG)
  ![Model Parameters Continued](automlfittedmodelparams1.PNG)<br>
  Once we have the best model we will register it using model register module.<br>
  ![Model registered](automlbestrunpy.PNG)<br>
  ![models from portal](automlmodelsfromportal.PNG)<br>
  ![AutoMLModel regsitered on portal](automlmodelregportal.PNG)<br>
  The Model Resgister codes are available in Jupyter Notebooks.<br>
  ![AutomLModelRegistercode](automlmodelregister.PNG)<br>
 ## Deployment <br>
  Now both models are tested we choose automl model to be deployed as it gave better accuarcy compared to HyperDrive Model.<br> Before Deploying the model we need to pack all the dependencies into conda environment file whcih are included in the repository. Once we pack the dependencies a docker conatiner is built and pushed to Azure Container isntance. We need to consume the ACI instance using a rest Endpoint. The endpoint deployed will be seen in endpoints section of the Azure Machine learning studio. Before deploying an endpoint we need to define scoring script which defines the entrypoint to the deployment whcih is given in repository.<br>
 We need to define inference config and endpoint config which are in jupyter Notebook of Automl.<br>
 Once the end point is deployed and shwoing healthy its ready to cosnume<br>
 ![Endpointinportal](Endpointdeployed.PNG)
 ![EndpointHealthy](endpointhealthy.PNG)<br>
 This shows the Endpoint is successfully deployed and is healthy.Now we can consume the endpoint using scoring URL genereated after deployment.<br>
 The Endpoint is consumed using endpoint.py where we use requests library for cosnuming the endpoint.<br>
 The sample input to the endpoint is as below<br>
 ![SampleDatatoendpoint](endpointsampledata.PNG)<br>
 Here we are testing two datapoints and we are expecting two outputs. This can be seen in next image<br>
 ![Endpointesting](endpointtesting.PNG)<br>
 In the same way we can test the endpoint on multiple Data points using sample data from the given dataset.<br>
 We convert the sample dataframe into json and pass the json to service endpoint.<br>
 ![Sample DataFrame](endpointtestting1.PNG)<br>
 This shows the endpoint is functioning successfully.
 ## Video Casting Explaining the Process
 [https://youtu.be/lzueNMlqs0w](Youtubelink)<br>
 ## Future Enhancements <BR>
 The model can be converted to ONNX format and deploy on Edge devices.<br>
 Applciation insights can be enabled.<br>
 
 
 
 
  
 
  


