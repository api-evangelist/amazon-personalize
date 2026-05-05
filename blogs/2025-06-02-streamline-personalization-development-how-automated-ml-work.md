---
title: "Streamline personalization development: How automated ML workflows accelerate Amazon Personalize implementation"
url: "https://aws.amazon.com/blogs/machine-learning/streamline-personalization-development-how-automated-ml-workflows-accelerate-amazon-personalize-implementation/"
date: "Mon, 02 Jun 2025 17:34:51 +0000"
author: "Reagan Rosario"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-personalize/feed/"
---
<p>Crafting unique, customized experiences that resonate with customers is a potent strategy for boosting engagement and fostering brand loyalty. However, creating dynamic personalized content is challenging and time-consuming because of the need for real-time data processing, complex algorithms for customer segmentation, and continuous optimization to adapt to shifting behaviors and preferences—all while providing scalability and accuracy. Despite these challenges, the potential rewards make personalization a worthwhile pursuit for many businesses. <a href="https://aws.amazon.com/personalize/" rel="noopener" target="_blank">Amazon Personalize</a> is a fully managed machine learning (ML) service that uses your data to generate product and content recommendations for your users. Amazon Personalize helps accelerate time-to-value with custom models that are trained on data you provide, such as your users, catalog items, and the interactions between users and items to generate personalized content and product recommendations. You can choose from various <a href="https://docs.aws.amazon.com/personalize/latest/dg/working-with-predefined-recipes.html" rel="noopener" target="_blank">recipes</a>—algorithms for specific use-cases—to find the ones that fit your needs, such as recommending items that a user is mostly likely to engage with next given their past interactions or next best action that a user is most likely to take.</p> 
<p>To maintain a personalized user experience, it’s crucial to implement <a href="https://aws.amazon.com/what-is/mlops/" rel="noopener" target="_blank">machine learning operations</a> (MLOps) practices, including continuous integration, deployment, and training of your ML models. MLOps facilitates seamless integration across various ML tools and frameworks, streamlining the development process. A robust machine learning solution for maintaining personalized experiences typically includes automated pipeline construction, as well as automated configuration, training, retraining, and deployment of personalization models. While services like Amazon Personalize offer a ready-to-use recommendation engine, establishing a comprehensive MLOps lifecycle for a personalization solution remains a complex undertaking. This process involves intricate steps to make sure that models remain accurate and relevant as user behaviors and preferences evolve over time.</p> 
<p>This blog post presents an MLOps solution that uses <a href="https://aws.amazon.com/cdk/" rel="noopener" target="_blank">AWS Cloud Development Kit (AWS CDK)</a> and services like <a href="https://aws.amazon.com/step-functions/" rel="noopener" target="_blank">AWS Step Functions</a>, <a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">Amazon EventBridge</a> and Amazon Personalize to automate provisioning resources for data preparation, model training, deployment, and monitoring for Amazon Personalize.</p> 
<h2>Features and benefits</h2> 
<p>Deploying this solution offers improved scalability and traceability and allows you to quickly set up a production-ready environment to seamlessly deliver tailored recommendations to users using Amazon Personalize. This solution:</p> 
<ul> 
 <li>Streamlines the creation and management of Amazon Personalize resources.</li> 
 <li>Provides greater flexibility in resource management and selective feature activation.</li> 
 <li>Enhances readability and comprehensibility of complex workflows.</li> 
 <li>Enables event-driven architecture by publishing key Amazon Personalize events, allowing real-time monitoring, and enabling automated responses and integrations with other systems.</li> 
 <li>Includes automated creation of Amazon Personalize resources, including recommenders, solutions, and solution versions.</li> 
 <li>Facilitates end-to-end workflow automation for dataset import, model training, and deployment in Amazon Personalize.</li> 
 <li>Improves organization and modularity of complex processes through nested step functions.</li> 
 <li>Provides flexible activation of specific solution components using AWS CDK.</li> 
</ul> 
<h2>Solution overview</h2> 
<p>This solution uses AWS CDK <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/aws-cdk-layers/layer-3.html" rel="noopener" target="_blank">layer 3 constructs</a>. Constructs are the basic building blocks of AWS CDK applications. A construct is a component within your application that represents one or more <a href="https://aws.amazon.com/cloudformation" rel="noopener" target="_blank">AWS CloudFormation</a> resources and their configuration.</p> 
<p><img alt="Solution overview" class="aligncenter wp-image-105795 size-full" height="666" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture1.png" style="margin: 10px 0px 10px 0px;" width="1430" /></p> 
<p>The solution architecture is shown in the preceding figure and includes:</p> 
<ol> 
 <li>An <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> bucket is used to store interactions, users, and items datasets. In this step, you need to configure your bucket permissions so that Amazon Personalize and <a href="https://aws.amazon.com/glue/" rel="noopener" target="_blank">AWS Glue</a> can access the datasets and input files.</li> 
 <li>AWS Glue is used to preprocess the interactions, users, and item datasets. This step helps ensure that the datasets comply with the training data requirements of Amazon Personalize. For more information, see <a href="https://docs.aws.amazon.com/personalize/latest/dg/preparing-training-data.html" rel="noopener" target="_blank">Preparing training data for Amazon Personalize</a>.</li> 
 <li>EventBridge is used to schedule regular updates, by triggering the workflow and for publishing events related to resource provisioning. Because Step Functions workflow orchestrates the workflow based on the input configuration file, you use that configuration when setting up the scheduled start of Step Functions.</li> 
 <li><a href="https://aws.amazon.com/step-functions/" rel="noopener" target="_blank">Step Functions</a> workflow manages all resource provisioning of the <a href="https://aws.amazon.com/personalize/" rel="noopener" target="_blank">Amazon Personalize</a> dataset group (including datasets, schemas, event tracker, filters, solutions, campaigns, and batch inference jobs). Step Functions provides monitoring across the solution through event logs. You can also visually track the stages of your workflow in the Step Functions console. You can adjust the input configuration file to better fit your use case, by defining schemas, recipes, and inference options. The solution workflow will have the following steps: 
  <ol> 
   <li>A preprocessing job that runs an AWS Glue job, if provided. This step facilitates any preprocessing of the data that might be required.</li> 
   <li>Create a dataset group, which is a container for Amazon Personalize resources.</li> 
   <li>Create a dataset import job for the datasets based on the defined S3 bucket.</li> 
   <li>Create filters that define any filtering that you want to apply on top of the recommendations.</li> 
   <li>Create an event tracker for ingesting real-time events, such as user interactions, which in turn influence the recommendations provided.</li> 
   <li>Create solutions and recommenders for creating <a href="https://docs.aws.amazon.com/personalize/latest/dg/create-custom-resources.html" rel="noopener" target="_blank">custom resources</a> and <a href="https://docs.aws.amazon.com/personalize/latest/dg/creating-recommenders.html" rel="noopener" target="_blank">domain recommenders</a>.</li> 
   <li>Create a campaign; or a batch inference or segment job for generating inferences for real-time, batch, and segmentation use cases respectively.</li> 
  </ol> </li> 
 <li>If you have a batch inference use case, then recommendations that match your inputs will be output into the S3 bucket that you defined in the input configuration file.</li> 
 <li>An <a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">Amazon EventBridge</a> event bus, where resource status notification updates are posted throughout the AWS Step Functions workflow.</li> 
</ol> 
<h2>Prerequisites</h2> 
<p>Before you deploy the AWS CDK stack, make sure that you have the following prerequisites in place:</p> 
<ol> 
 <li>Install and configure <a href="https://aws.amazon.com/cli/" rel="noopener" target="_blank">AWS Command Line Interface (AWS CLI).</a></li> 
 <li>Install Python 3.12 or newer</li> 
 <li>Install <a href="https://nodejs.org/en/" rel="noopener" target="_blank">Node.js</a> 20.16.0 or newer.</li> 
 <li>Install <a href="https://aws.amazon.com/cdk/" rel="noopener" target="_blank">AWS CDK</a> 2.88.0 or newer.</li> 
 <li>Docker 27.5.1 or newer (required for <a href="https://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> function bundling).</li> 
</ol> 
<p>Newer versions of AWS CLI, Python, Node.js, and the AWS CDK are generally compatible, this solution has been tested with the versions listed.</p> 
<h2>Deploy the solution</h2> 
<p>With the prerequisites in place, use the following steps to deploy the solution:</p> 
<ol> 
 <li>Clone the <a href="https://github.com/aws-samples/automate-mlops-personalize-cdk-pipeline" rel="noopener" target="_blank">repository</a> to a new folder on your desktop using the following command:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">git clone https://github.com/aws-samples/automate-mlops-personalize-cdk-pipeline.git</code></pre> 
</div> 
<ol start="2"> 
 <li>Create a Python virtual environment for development:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">python3 -m venv .venv
source .venv/bin/activate 
pip install -r requirements.txt</code></pre> 
</div> 
<ol start="3"> 
 <li>Define an Amazon Personalize MLOps pipeline instance <code>PersonalizeMlOpsPipeline</code> (see <a href="https://github.com/aws-samples/automate-mlops-personalize-cdk-pipeline/blob/main/personalize/infrastructure/stacks/personalize_pipeline_stack.py" rel="noopener" target="_blank">personalize_pipeline_stack.py</a> for the complete example, which also includes different inference options). In this walkthrough, you create a custom solution with an associated campaign and batch inference job:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-python">PersonalizeMlOpsPipeline(
    self, 'PersonalizePipelineSolution',
    pre_processing_config={
        "job_class": PreprocessingGlueJobFlow
    },
    enable_filters=True,
    enable_event_tracker=True,
    recommendation_config=[
        {
            "type": "solutions",
            "inference_options": ["campaigns", "batchInferenceJobs"]
        },
        {
            "type": "recommenders"
        }
    ]

)
</code></pre> 
</div> 
<p>Where:</p> 
<ul> 
 <li><strong>‘PersonalizePipelineSolution</strong>‘ – The name of the pipeline solution stack</li> 
 <li><strong>pre_processing_config</strong> – Configuration for the pre-processing job to transform raw data into a format usable by Amazon Personalize. For using AWS Glue jobs for preprocessing specify the AWS Glue job class (<code>PreprocessingGlueJobFlow</code>) as a value to the parameter <code>job_class</code>. Currently, only AWS Glue jobs are supported. You can pass the name of the AWS Glue job that you need to run as a part of the input config. This doesn’t deploy the actual AWS Glue job responsible for pre-processing the files; the actual AWS Glue must be created outside of this solution and the name passed as an input to the state machine. A sample AWS Glue job is supplied in the accompanying repo, which shows how pre-processing can be done.</li> 
 <li><strong>enable_filters</strong> – A Boolean value to enable dataset filters for pre-processing. When set to true, the pipeline will create the state machines needed to create filters. Supported options are true or false. If you specify this value as false, the corresponding state machine is not deployed.</li> 
 <li><strong>enable_event_tracker</strong> – A Boolean value to enable the Amazon Personalize event tracker. When set to true, the pipeline will create the state machines needed to create an event tracker. Supported options are true or false. If you specify this value as false, the corresponding state machine is not deployed.</li> 
 <li><strong>recommendation_config</strong> – Configuration options for recommendations. The two types currently supported are <code>solutions</code> and <code>recommenders</code>. Within the solutions type, you can have multiple options such as <code>campaigns</code>, <code>batchInferenceJobs</code>, and <code>batchSegmentJobs</code>. Based on the selected options, the corresponding state machine and components are created. In the earlier example, we used <code>campaigns</code> and <code>batchInferenceJobs</code> as the option, which means that only the campaigns and batch inference job state machines will be deployed with the AWS CDK.</li> 
</ul> 
<p>After the infrastructure is deployed you can also enable and disable certain options through the state machine input configuration file. You can use this AWS CDK code to control what components are deployed in your AWS environment and with the input config, you can select what components run.</p> 
<p><strong>Preprocessing:</strong> As an optional step, you can use an existing AWS Glue job for preprocessing your data before feeding it into Amazon Personalize, which uses this data to generate recommendations for your end users. While this post demonstrates the process using the <a href="https://grouplens.org/datasets/movielens/" rel="noopener" target="_blank">Movie Lens</a> dataset, you can adapt it for your own datasets or custom processing needs. To do so, navigate to the <code>glue_job</code> folder and modify the <code>movie_script.py</code> file accordingly, or create an entirely new AWS Glue job tailored to your specific requirements. This preprocessing step, though optional, can be crucial in making sure that your data is optimally formatted for Amazon Personalize to generate accurate recommendations.</p> 
<ol> 
 <li>Make sure that the AWS Glue job is configured to write its output to an S3 bucket. This bucket should then be specified as an input source in the Step Functions input configuration file.</li> 
 <li>Verify that the AWS Glue service has the necessary permissions to access the S3 bucket mentioned in your script.</li> 
 <li>In the input configuration, you’ll need to provide the name of the AWS Glue job that will be executed by the main state machine workflow. It’s crucial that this specified AWS Glue job runs without any errors, because any failures could potentially cause the entire state machine execution to fail.</li> 
</ol> 
<p>Package and deploy the solution with AWS CDK, allowing for the most flexibility in development:</p> 
<p>Before you can deploy the pipeline using AWS CDK, you need to set up AWS credentials on your local machine. You can refer <a href="https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html" rel="noopener" target="_blank">Set up AWS temporary credentials</a> for more details.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash"># bootstrap CDK (required once - deploys a CDK bootstrap CloudFormation stack for assets)
cdk bootstrap
# build the solution
cdk synth
# build and deploy the solution
cdk deploy</code></pre> 
</div> 
<h2>Run the pipeline</h2> 
<p>Before initiating the pipeline, create the resources that follow and document the resource names for future reference.</p> 
<ol> 
 <li>Set up an S3 bucket for dataset storage. If you plan to use the preprocessing step, this should be the same bucket as the output destination.</li> 
 <li>Update the S3 bucket policy to grant Amazon Personalize the necessary access permissions. See <a href="https://docs.aws.amazon.com/personalize/latest/dg/granting-personalize-s3-access.html" rel="noopener" target="_blank">Giving Amazon Personalize access to Amazon S3 resources</a> for policy examples.</li> 
 <li>Create an <a href="https://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management (IAM)</a> role to be used by the state machine for accessing Amazon Personalize resources.</li> 
</ol> 
<p>You can find detailed instructions and policy examples in the <a href="https://github.com/aws-samples/automate-mlops-personalize-cdk-pipeline?tab=readme-ov-file#5-test-the-pipeline" rel="noopener" target="_blank">GitHub repository</a>.</p> 
<p>After you’ve set up these resources, you can create the input configuration file for the Step Functions state machine. If you configure the optional AWS Glue job it will create the input files that are required as an input to the pipeline, refer <a href="https://github.com/aws-samples/automate-mlops-personalize-cdk-pipeline/tree/main?tab=readme-ov-file#optional-configure-the-glue-job-to-create-the-output-files" rel="noopener" target="_blank">Configure the Glue Job to create the output files</a> for more details.</p> 
<h3>Create input configuration</h3> 
<p>This input file is crucial because it contains all the essential information needed to create and manage your Amazon Personalize resources, this input configuration json acts as input to the Step Functions state machine. The file can contain the following top level objects</p> 
<ul> 
 <li><code>datasetGroup</code></li> 
 <li><code>datasets</code></li> 
 <li><code>eventTracker</code></li> 
 <li><code>filters</code></li> 
 <li><code>solutions</code> (can contain <code>campaigns</code>, <code>batchInferenceJobs</code> and <code>batchSegmentJobs</code>)</li> 
 <li><code>recommenders</code></li> 
</ul> 
<p>Customize the configuration file according to your specific requirements and include or exclude sections based on the Amazon Personalize artifacts that you want to create. For the dataset import jobs in the datasets section, replace <code>AWS_ACCOUNT_ID</code>, <code>S3_BUCKET_NAME</code> and <code>IAM_ROLE_ARN</code> with the appropriate values. The following is a snippet of the input configuration file. For a complete sample, see <a href="https://github.com/aws-samples/automate-mlops-personalize-cdk-pipeline/blob/main/personalize/sample_configs/input_media.json" rel="noopener" target="_blank">input_media.json</a>.</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">"datasetImportJob": {
    "serviceConfig": {
        "dataSource": {
            "dataLocation": "s3://S3_BUCKET_NAME/users.csv"
        },
        "roleArn": "arn:aws:iam::AWS_ACCOUNT_ID:role/IAM_ROLE_NAME",
        "tags": [
            {
                "tagKey": "string",
                "tagValue": "string"
            }
        ],
        "importMode": "FULL",
        "jobName": "UsersDatasetImportJob"
    },
    "createNewJob": true
}
}</code></pre> 
</div> 
<p>Likewise, if you’re using batch inference or batch segment jobs, remember to also update the <code>BUCKET_NAME</code> and <code>IAM ROLE ARN</code> in those sections. It’s important to verify that you have the required input files for batch inference stored in your S3 bucket. Adjust the file paths in your configuration to accurately reflect the location of these files within your bucket structure. This helps ensure that Amazon Personalize can access the correct data when executing these batch processes.</p> 
<p>Adjust the AWS Glue Job name in the configuration file if you have configured it as a part of the AWS CDK stack.</p> 
<p>See the <a href="https://github.com/aws-samples/automate-mlops-personalize-cdk-pipeline/blob/main/README.md#property-details" rel="noopener" target="_blank">property table </a>for a deep dive into each property and identify whether it’s optional or required.</p> 
<h3>Execute the pipeline</h3> 
<p>You can run the pipeline using the main state machine by the name <code>PersonalizePipelineSolution</code> from the <a href="https://console.aws.amazon.com/states/home#/statemachines" rel="noopener" target="_blank">Step Functions Console</a> or set up a schedule in EventBridge (find the step-by step process in the <strong><em>Schedule the workflow for continued maintenance of the solution</em></strong> section of this post).</p> 
<ol> 
 <li>In the AWS Management Console for Step Functions, navigate to <strong>State machines</strong> and select the <strong>PersonalizePipelineSolution</strong>.</li> 
</ol> 
<p><img alt="Personalize Pipeline Solution" class="aligncenter wp-image-105794 size-full" height="391" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture2.png" style="margin: 10px 0px 10px 0px;" width="1430" /></p> 
<ol start="2"> 
 <li>Choose <strong>Start Execution</strong> and enter the configuration file that you created for your use case based on the steps in the <strong><em>Create input configuration </em></strong>section<strong><em>.</em></strong></li> 
</ol> 
<p><img alt="Start Execution" class="aligncenter wp-image-105793 size-full" height="482" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture3.png" style="margin: 10px 0px 10px 0px;" width="1430" /></p> 
<ol start="3"> 
 <li>Choose <strong>Start execution</strong> and monitor the State Machine execution. In the Step Functions console, you will find a visual representation of the workflow and can track at what stage the execution is. Event logs will give you insight into the progress of the stages and information if there are any errors. The following figure is an example of a completed workflow:</li> 
</ol> 
<p><img alt="Graph view of AWS Step Functions" class="alignnone wp-image-105792 size-full" height="821" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture4.png" style="margin: 10px 0px 10px 0px;" width="1430" /></p> 
<ol start="4"> 
 <li>After the workflow finishes, you can view the resources in the Amazon Personalize console. For batch inference jobs specifically, you can locate the corresponding step under the <strong>Inference tasks</strong> section of the graph, and within the <strong>Custom Resources</strong> area of the Amazon Personalize console.</li> 
</ol> 
<h2>Get recommendations (real-time inference)</h2> 
<p>After your pipeline has completed its run successfully, you can obtain recommendations. In the example configuration, we chose to deploy campaigns as the inference option. As a result, you’ll have access to a campaign that can provide real-time recommendations.</p> 
<p>We use the <a href="https://console.aws.amazon.com/personalize/home" rel="noopener" target="_blank">Amazon Personalize console </a>to get recommendations. Choose <strong>Dataset groups</strong> and select your dataset group name. Choose <strong>Campaigns</strong> and select your campaign name<em>. </em>Enter a userid and items Ids of your choice to test personalized ranking, you can get the userid and item Ids from the input file in the Amazon S3 bucket you configured.</p> 
<p><img alt="Test Campaign Results" class="aligncenter wp-image-105791 size-full" height="770" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture5.png" style="margin: 10px 0px 10px 0px;" width="1430" /></p> 
<p><strong>&nbsp;</strong></p> 
<h2>Get recommendations (batch inference)</h2> 
<p>If you have configured batch inference to run, start by verifying that the batch inference step has successfully completed in the Step Functions workflow. Then, use the Amazon S3 console to navigate to the destination S3 bucket for your batch inference job. If you don’t see an output file there, verify that you’ve provided the correct path for the input file in your input configuration.</p> 
<p><img alt="Media Inference Output - S3 Bucket" class="aligncenter wp-image-105790 size-full" height="513" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture6.png" style="margin: 10px 0px 10px 0px;" width="1430" /></p> 
<p><strong>&nbsp;</strong></p> 
<h2>Schedule the workflow for continued maintenance of the solution</h2> 
<p>While Amazon Personalize offers <a href="https://aws.amazon.com/blogs/machine-learning/introducing-automatic-training-for-solutions-in-amazon-personalize/" rel="noopener" target="_blank">automatic training</a> for solutions through its console or SDK, allowing users to set retraining frequencies such as every three days, this MLOps workflow provides an enhanced approach. By using EventBridge schedules you gain more precise control over the timing of retraining processes. Using this method, you can specify exact dates and times for retraining executions. To implement this advanced scheduling, you can configure an EventBridge schedule to trigger the Step Functions execution, giving you finer granularity in managing your machine learning model updates.</p> 
<ol> 
 <li>Navigate to the <a href="https://console.aws.amazon.com/events/" rel="noopener" target="_blank">Amazon EventBridge Console</a> and select <strong>EventBridge Schedule </strong>and then choose <strong>Create schedule</strong>.</li> 
</ol> 
<p><img alt="Amazon EventBridge" class="aligncenter wp-image-105789 size-full" height="467" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture7.png" style="margin: 10px 0px 10px 0px;" width="1277" /></p> 
<ol start="2"> 
 <li>You can establish a recurring schedule for executing your entire workflow. A key benefit of this solution is the enhanced control it offers over the specific date and time you want your workflow to run. This allows for precise timing of your processes, which you can use to align the workflow execution with your operational needs or optimal data processing windows.</li> 
</ol> 
<p><img alt="Specify schedule detail" class="alignnone wp-image-105788 size-full" height="681" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture8.png" style="margin: 10px 0px 10px 0px;" width="1246" /></p> 
<ol start="3"> 
 <li>Select AWS Step Functions (as shown below) as your target.</li> 
</ol> 
<p><img alt="Select Target - Event Bridge" class="aligncenter wp-image-105787 size-full" height="626" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture9.png" style="margin: 10px 0px 10px 0px;" width="1430" /></p> 
<ol start="4"> 
 <li>Insert the input configuration file that you prepared previously as the input and click Next.</li> 
</ol> 
<p><img alt="Start Execution" class="aligncenter wp-image-105786 size-full" height="738" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture10.png" style="margin: 10px 0px 10px 0px;" width="936" /></p> 
<p>An additional step you can take is to set up a dead-letter queue with <a href="https://aws.amazon.com/sqs" rel="noopener" target="_blank">Amazon Simple Query Service (Amazon SQS)</a> to handle failed Step Functions executions.</p> 
<h2>Monitoring and notification</h2> 
<p>To maintain the reliability, availability, and performance of Step Functions and your solution, set up monitoring and logging. You can set up an EventBridge rule to receive notifications about events that are of interest, such as batch inference being ready in the S3 bucket. Here is how you can set that up:</p> 
<ol> 
 <li>Navigate to <a href="https://us-west-2.console.aws.amazon.com/sns/" rel="noopener" target="_blank">Amazon Simple Notification Service (Amazon SNS)</a> console and create an SNS topic that will be the target for your event.</li> 
</ol> 
<p><img alt="Create Topic" class="aligncenter wp-image-105785 size-full" height="495" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture11.png" style="margin: 10px 0px 10px 0px;" width="1430" /></p> 
<ol start="2"> 
 <li>Amazon SNS supports subscription for different endpoint types such as HTTP/HTTPS, email, Lambda, SMS, and so on. For this example, use an email endpoint.</li> 
</ol> 
<p><img alt="Create subscription" class="aligncenter wp-image-105784 size-full" height="421" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture12.png" style="margin: 10px 0px 10px 0px;" width="1430" /></p> 
<ol start="3"> 
 <li>After you create the topic and the subscription, navigate to the EventBridge console and select <strong>Create Rule</strong>. Define the details associated with the event such as the name, description, and the event bus.</li> 
</ol> 
<p><img alt="Define rule detail" class="aligncenter wp-image-105783 size-full" height="583" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture13.png" style="margin: 10px 0px 10px 0px;" width="1430" /></p> 
<ol start="4"> 
 <li>To set up the event rule, you’ll use the pattern form. You use this form to define the specific events that will trigger notifications. For the batch segment job completion step, you should configure the source and detail-type fields as follows:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
"source": ["solutions.aws.personalize"],
"detail-type": ["Personalize BatchSegmentJob status change"]
}</code></pre> 
</div> 
<p><img alt="Event pattern" class="aligncenter wp-image-105782 size-full" height="843" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture14.png" style="margin: 10px 0px 10px 0px;" width="1430" /></p> 
<ol start="5"> 
 <li>Select the SNS topic as your target and proceed.</li> 
</ol> 
<p><img alt="Select target(s)" class="aligncenter wp-image-105781 size-full" height="430" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/03/ML-17195-Picture15.png" style="margin: 10px 0px 10px 0px;" width="936" /></p> 
<p>With this procedure, you have set up an EventBridge rule to receive notifications on your email when an object is created in your batch inference bucket. You can also set up logic based on your use case to trigger any downstream processes such as creation of email campaigns with the results of your inference by choosing different targets such as Lambda.</p> 
<p>Additionally, you can use Step Functions and Amazon Personalize monitoring through <a href="https://aws.amazon.com/cloudwatch" rel="noopener" target="_blank">Amazon CloudWatch</a> metrics. See <a href="https://docs.aws.amazon.com/step-functions/latest/dg/monitoring-logging.html" rel="noopener" target="_blank">Logging and Monitoring AWS Step Functions</a> and <a href="https://docs.aws.amazon.com/personalize/latest/dg/personalize-monitoring.html" rel="noopener" target="_blank">Monitoring Amazon Personalize</a> for more information.</p> 
<h2>Handling schema updates</h2> 
<p>Schema updates are available in Amazon Personalize for adding columns to the existing schema. Note that deleting columns from existing schemas isn’t currently supported. To update the schema, make sure that you’re modifying the schema in the input configuration passed to Step Functions. See <a href="https://docs.aws.amazon.com/personalize/latest/dg/updating-dataset-schema.html" rel="noopener" target="_blank">Replacing a dataset’s schema to add new columns</a> for more information.</p> 
<h2>Clean up</h2> 
<p>To avoid incurring additional costs, delete the resources you created during this solution walkthrough. You can clean up the solution by deleting the CloudFormation stack you deployed as part of the setup.</p> 
<p><strong>Using the console</strong></p> 
<ol> 
 <li>Sign in to the <a href="https://console.aws.amazon.com/cloudformation/home" rel="noopener" target="_blank">AWS CloudFormation console</a>.</li> 
 <li>On the <strong>Stacks</strong> page, select this solution’s installation stack.</li> 
 <li>Choose <strong>Delete</strong>.</li> 
</ol> 
<p><strong>Using the AWS CLI</strong></p> 
<div class="hide-language"> 
 <pre><code class="lang-json">$ aws cloudformation delete-stack —stack-name &lt;installation-stack-name&gt;</code><strong style="font-family: Georgia, 'Times New Roman', 'Bitstream Charter', Times, serif;">&nbsp;</strong></pre> 
</div> 
<h2>Conclusion</h2> 
<p>This MLOps solution for Amazon Personalize offers a powerful, automated approach to creating and maintaining personalized user experiences at scale. By using AWS services like AWS CDK, Step Functions, and EventBridge, the solution streamlines the entire process from data preparation through model deployment and monitoring. The flexibility of this solution means that you can customize it to fit various use cases, and integration with EventBridge keeps models up to date. Delivering exceptional personalized experiences is critical for business growth, and this solution provides an efficient way to harness the power of Amazon Personalize to improve user engagement, customer loyalty, and business results. We encourage you to explore and adapt this solution to enhance your personalization efforts and stay ahead in the competitive digital landscape.</p> 
<p>To learn more about the capabilities discussed in this post, check out <a href="https://aws.amazon.com/personalize/features/" rel="noopener" target="_blank">Amazon Personalize features</a> and the <a href="https://docs.aws.amazon.com/personalize/latest/dg/what-is-personalize.html" rel="noopener" target="_blank">Amazon Personalize Developer Guide</a>.</p> 
<h2>Additional resources:</h2> 
<ul> 
 <li><a href="https://aws.amazon.com/blogs/machine-learning/how-vistaprint-delivers-personalized-product-recommendations-with-amazon-personalize/" rel="noopener" target="_blank">How VistaPrint delivers personalized product recommendations with Amazon Personalize</a></li> 
 <li><a href="https://aws.amazon.com/blogs/machine-learning/amazon-personalize-launches-new-recipes-supporting-larger-item-catalogs-with-lower-latency/" rel="noopener" target="_blank">Amazon Personalize launches new recipes supporting larger item catalogs with lower latency</a></li> 
 <li><a href="https://aws.amazon.com/blogs/machine-learning/introducing-automatic-training-for-solutions-in-amazon-personalize/" rel="noopener" target="_blank">Introducing automatic training for solutions in Amazon Personalize</a></li> 
 <li><a href="https://aws.amazon.com/blogs/machine-learning/build-a-news-recommender-application-with-amazon-personalize/" rel="noopener" target="_blank">Build a news recommender application with Amazon Personalize</a></li> 
 <li><a href="https://aws.amazon.com/blogs/machine-learning/unlock-personalized-experiences-powered-by-ai-using-amazon-personalize-and-amazon-opensearch-service/" rel="noopener" target="_blank">Unlock personalized experiences powered by AI using Amazon Personalize and Amazon OpenSearch Service</a></li> 
</ul> 
<p style="clear: both;"></p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><img alt="" class="size-full wp-image-78224 alignleft" height="139" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/07/reagan-rosario-aws-author-bio-1.jpeg" width="100" /><strong>Reagan Rosario</strong> brings over a decade of technical expertise to his role as a Sr. Specialist Solutions Architect in Generative AI at AWS. Reagan transforms enterprise systems through strategic implementation of AI-powered cloud solutions, automated workflows, and innovative architecture design. His specialty lies in guiding organizations through digital evolution—preserving core business value while implementing cutting-edge generative AI capabilities that dramatically enhance operations and create new possibilities.</p> 
<p style="clear: both;"><img alt="" class="alignleft wp-image-105841 size-thumbnail" height="125" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/05/05/nensi-100x125.png" width="100" /><strong>Nensi Hakobjanyan</strong> is a Solutions Architect at Amazon Web Services, where she supports enterprise Retail and CPG customers in designing and implementing cloud solutions. In addition to her deep expertise in cloud architecture, Nensi brings extensive experience in Machine Learning and Artificial Intelligence, helping organizations unlock the full potential of data-driven innovation. She is passionate about helping customers through digital transformation and building scalable, future-ready solutions in the cloud.</p>
