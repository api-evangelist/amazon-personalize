---
title: "Introducing automatic training for solutions in Amazon Personalize"
url: "https://aws.amazon.com/blogs/machine-learning/introducing-automatic-training-for-solutions-in-amazon-personalize/"
date: "Sat, 20 Apr 2024 00:38:31 +0000"
author: "Ba'Carri Johnson"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-personalize/feed/"
---
<p><a href="https://aws.amazon.com/personalize/">Amazon Personalize</a> is excited to announce automatic training for solutions. Solution training is fundamental to maintain the effectiveness of a model and make sure recommendations align with users’ evolving behaviors and preferences. As data patterns and trends change over time, retraining the solution with the latest relevant data enables the model to learn and adapt, enhancing its predictive accuracy. Automatic training generates a new solution version, mitigating model drift and keeping recommendations relevant and tailored to end-users’ current behaviors while including the newest items. Ultimately, automatic training provides a more personalized and engaging experience that adapts to changing preferences.</p> 
<p>Amazon Personalize accelerates your digital transformation with machine learning (ML), making it effortless to integrate personalized recommendations into existing websites, applications, email marketing systems, and more. Amazon Personalize enables developers to quickly implement a customized personalization engine, without requiring ML expertise. Amazon Personalize provisions the necessary infrastructure and manages the entire ML pipeline, including processing the data, identifying features, using the appropriate algorithms, and training, optimizing, and hosting the customized models based on your data. All your data is encrypted to be private and secure.</p> 
<p>In this post, we guide you through the process of configuring automatic training, so your solutions and recommendations maintain their accuracy and relevance.</p> 
<h2>Solution overview</h2> 
<p>A <em>solution</em> refers to the combination of an Amazon Personalize recipe, customized parameters, and one or more solution versions (trained models). When you create a custom solution, you specify a recipe matching your use case and configure training parameters. For this post, you configure automatic training in the training parameters.</p> 
<h2>Prerequisites</h2> 
<p>To enable automatic training for your solutions, you first need to set up Amazon Personalize resources. Start by <a href="https://docs.aws.amazon.com/personalize/latest/dg/create-domain-dataset-group.html" rel="noopener" target="_blank">creating a dataset group</a>, schemas, and <a href="https://docs.aws.amazon.com/personalize/latest/dg/custom-datasets-and-schemas.html" rel="noopener" target="_blank">datasets</a> representing your items, interactions, and user data. For instructions, refer to <a href="https://docs.aws.amazon.com/personalize/latest/dg/getting-started-console.html" rel="noopener" target="_blank">Getting Started (console)</a> or <a href="https://docs.aws.amazon.com/personalize/latest/dg/getting-started-cli.html" rel="noopener" target="_blank">Getting Started (AWS CLI)</a>.</p> 
<p>After you finish importing your data, you are ready to create a solution.</p> 
<h2>Create a solution</h2> 
<p>To set up automatic training, complete the following steps:</p> 
<ol> 
 <li>On the Amazon Personalize console, create a new solution.</li> 
 <li>Specify a name for your solution, choose the type of solution you want to create, and choose your recipe.</li> 
 <li>Optionally, add any tags. For more information about tagging Amazon Personalize resources, see <a href="https://docs.aws.amazon.com/personalize/latest/dg/tagging-resources.html" rel="noopener" target="_blank">Tagging Amazon Personalize resources</a>.</li> 
 <li>To use automatic training, in the <strong>Automatic training </strong>section, select <strong>Turn on</strong> and specify your training frequency.</li> 
</ol> 
<p>Automatic training is enabled by default to train one time every 7 days. You can configure the training cadence to suit your business needs, ranging from one time every 1–30 days.</p> 
<ol> 
 <li>If your recipe generates item recommendations or user segments, optionally use the <strong>Columns for training</strong> section to choose the columns Amazon Personalize considers when training solution versions.</li> 
 <li>In the <strong>Hyperparameter configuration</strong> <strong>section</strong>, optionally configure any hyperparameter options based on your recipe and business needs.</li> 
 <li>Provide any additional configurations, then choose <strong>Next</strong>.<br /> <img alt="" class="alignnone size-full wp-image-74748" height="910" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/04/19/ML-16686-image001.png" style="margin: 10px 0px 10px 0px;" width="1386" /></li> 
 <li>Review the solution details and confirm that your automatic training is configured as expected.</li> 
 <li>Choose <strong>Create solution</strong>.<br /> <img alt="" class="alignnone size-full wp-image-74749" height="1143" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/04/19/ML-16686-image003.png" style="margin: 10px 0px 10px 0px;" width="1540" /></li> 
</ol> 
<p>Amazon Personalize will automatically create your first solution version. A <em>solution version</em> refers to a trained ML model. When a solution version is created for the solution, Amazon Personalize trains the model backing the solution version based on the recipe and training configuration. It can take up to 1 hour for the solution version creation to start.</p> 
<p>The following is sample code for creating a solution with automatic training using the AWS SDK:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">import boto3 
personalize = boto3.client('personalize')

solution_config = {
    "autoTrainingConfig": {
        "schedulingExpression": "rate(3 days)"
    }
}

recipe = "arn:aws:personalize:::recipe/aws-similar-items"
name = "test_automatic_training"
response = personalize.create_solution(name=name, recipeArn=recipe_arn, datasetGroupArn=dataset_group_arn, 
                            performAutoTraining=True, solutionConfig=solution_config)

print(response['solutionArn'])
solution_arn = response['solutionArn'])</code></pre> 
</div> 
<p>After a solution is created, you can confirm whether automatic training is enabled on the solution details page.&nbsp;Additionally, you can update your automatic training configuration by enabling or disabling automatic training, and adjusting the training frequency as needed.</p> 
<p><img alt="" class="alignnone wp-image-85167 size-full" height="2390" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/30/Solution-details-3-002.png" style="margin: 10px 0px 10px 0px;" width="5100" /></p> 
<p>You can also use the following sample code to confirm via the AWS SDK that automatic training is enabled:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">response = personalize.describe_solution(solutionArn=solution_arn)
print(response)</code></pre> 
</div> 
<p>Your response will contain the fields <code>performAutoTraining</code> and <code>autoTrainingConfig</code>, displaying the values you set in the <code>CreateSolution</code> call.</p> 
<p>Use the following sample code to update automatic training via the AWS SDK:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">solution_config = {
    "autoTrainingConfig": {
        "schedulingExpression": "rate(1 days)"
    }
}

response = personalize.update_solution(solutionArn=solution_arn, 
                            performAutoTraining=True, solutionConfig=solution_config)
print(response['solutionArn'])
solution_arn = response['solutionArn'])</code></pre> 
</div> 
<p>Your response will contain the field&nbsp;<code>latestSolutionUpdate</code>&nbsp;which will contain fields&nbsp;<code>performAutoTraining</code> and&nbsp;<code>autoTrainingConfig</code>, displaying the values you set in the&nbsp;<code>UpdateSolution</code> call.</p> 
<p>On the solution details page, you will also see the solution versions that are created automatically. The <strong>Training type</strong> column specifies whether the solution version was created manually or automatically.</p> 
<p><img alt="" class="alignnone size-full wp-image-74753" height="751" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/04/19/ML-16686-image011.png" style="margin: 10px 0px 10px 0px;" width="1432" /></p> 
<p>You can also use the following sample code to return a list of solution versions for the given solution:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">response = personalize.list_solution_versions(solutionArn=solution_arn)['solutionVersions']
print("List Solution Version response\n")
for val in response:
    print(f"SolutionVersion: {val}")
    print("\n")</code></pre> 
</div> 
<p>Your response will contain the field <code>trainingType</code>, which specifies whether the solution version was created manually or automatically.</p> 
<p>When your solution version is ready, you can <a href="https://docs.aws.amazon.com/personalize/latest/dg/campaigns.html" rel="noopener" target="_blank">create a campaign</a> for your solution version.</p> 
<h2>Create a campaign</h2> 
<p>A <em>campaign</em> deploys a solution version (trained model) to generate real-time recommendations. With Amazon Personalize, you can streamline your workflow and automate the deployment of the latest solution version to campaigns via automatic syncing. To set up auto sync, complete the following steps:</p> 
<ol> 
 <li>On the Amazon Personalize console, create a new campaign.</li> 
 <li>Specify a name for your campaign.</li> 
 <li>Choose the solution you just created.</li> 
 <li>Select <strong>Automatically use the latest solution version</strong>.</li> 
 <li>Set the <a href="https://docs.aws.amazon.com/personalize/latest/dg/campaigns.html#min-tps-auto-scaling" rel="noopener" target="_blank">minimum provisioned transactions per second</a>.<br /> <img alt="" class="alignnone wp-image-74755 size-full" height="1152" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/04/19/ML-16686-image015.png" style="margin: 10px 0px 10px 0px;" width="1448" /></li> 
 <li>Create your campaign.</li> 
</ol> 
<p>The campaign is ready when its status is <code>ACTIVE</code>.</p> 
<p>The following is sample code for creating a campaign with <code>syncWithLatestSolutionVersion</code> set to <code>true</code> using the AWS SDK. You must also append the suffix <code>$LATEST</code> to the <code>solutionArn</code> in <code>solutionVersionArn</code> when you set <code>syncWithLatestSolutionVersion</code> to <code>true</code>.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">campaign_config = {
    "syncWithLatestSolutionVersion": True
}
resource_name = "test_campaign_sync"
solution_version_arn = "arn:aws:personalize:&lt;region&gt;:&lt;accountId&gt;:solution/&lt;solution_name&gt;/$LATEST"
response = personalize.create_campaign(name=resource_name, solutionVersionArn=solution_version_arn, campaignConfig=campaign_config)
campaign_arn = response['campaignArn']
print(campaign_arn)</code></pre> 
</div> 
<p>On the campaign details page, you can see whether the campaign selected has auto sync enabled. When enabled, your campaign will automatically update to use the most recent solution version, whether it was automatically or manually created.</p> 
<p><img alt="" class="alignnone size-full wp-image-74757" height="657" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/04/19/ML-16686-image019.png" style="margin: 10px 0px 10px 0px;" width="1428" /></p> 
<p>Use the following sample code to confirm via the AWS SDK that <code>syncWithLatestSolutionVersion</code> is enabled:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">response = personalize.describe_campaign(campaignArn=campaign_arn)
Print(response)</code></pre> 
</div> 
<p>Your response will contain the field <code>syncWithLatestSolutionVersion</code> under <code>campaignConfig</code>, displaying the value you set in the <code>CreateCampaign</code> call.</p> 
<p>You can enable or disable the option to automatically use the latest solution version on the Amazon Personalize console after a campaign is created by updating your campaign. Similarly, you can enable or disable <code>syncWithLatestSolutionVersion</code> with <code>UpdateCampaign</code> using the AWS SDK.</p> 
<h2>Conclusion</h2> 
<p>With automatic training, you can mitigate model drift and maintain recommendation relevance by streamlining your workflow and automating the deployment of the latest solution version in Amazon Personalize.</p> 
<p>For more information about optimizing your user experience with Amazon Personalize, see the <a href="https://docs.aws.amazon.com/personalize/latest/dg/what-is-personalize.html" rel="noopener" target="_blank">Amazon Personalize Developer Guide</a>.</p> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><img alt="" class="wp-image-64479 size-full alignleft" height="130" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/10/24/BaCarri-Johnson-100.jpg" width="100" /><strong>Ba’Carri Johnson</strong> is a Sr. Technical Product Manager working with AWS AI/ML on the Amazon Personalize team. With a background in computer science and strategy, she is passionate about product innovation. In her spare time, she enjoys traveling and exploring the great outdoors.</p> 
<p style="clear: both;"><img alt="" class="alignleft size-full wp-image-74760" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/04/19/ajvenk.jpeg" width="100" /><strong>Ajay Venkatakrishnan </strong>is a Software Development Engineer on the Amazon Personalize team. In his spare time, he enjoys writing and playing soccer.</p> 
<p style="clear: both;"><img alt="" class="size-full wp-image-66997 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/11/21/Pranesh-Anubhav.jpg" width="100" /><strong>Pranesh Anubhav</strong> is a Senior Software Engineer for Amazon Personalize. He is passionate about designing machine learning systems to serve customers at scale. Outside of his work, he loves playing soccer and is an avid follower of Real Madrid.</p>
