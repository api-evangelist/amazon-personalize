---
title: "Provide a personalized experience for news readers using Amazon Personalize and Amazon Titan Text Embeddings on Amazon Bedrock"
url: "https://aws.amazon.com/blogs/machine-learning/provide-a-personalized-experience-for-news-readers-using-amazon-personalize-and-amazon-titan-text-embeddings-on-amazon-bedrock/"
date: "Thu, 29 Aug 2024 17:00:57 +0000"
author: "Joydeep Dutta"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-personalize/feed/"
---
<p>News publishers want to provide a personalized and informative experience to their readers, but the short shelf life of news articles can make this quite difficult. In news publishing, articles typically have peak readership within the same day of publication. Additionally, news publishers frequently publish new articles and want to show these articles to interested readers as quickly as possible. This poses challenges for interaction-based recommender system methodologies such as collaborative filtering and the deep learning-based approaches used in <a href="https://aws.amazon.com/personalize/" rel="noopener" target="_blank">Amazon Personalize</a>, a managed service that can learn user preferences from their past behavior and quickly adjust recommendations to account for changing user behavior in near real time.</p> 
<p>News publishers typically don’t have the budget or the staff to experiment with in-house algorithms, and need a fully managed solution. In this post, we demonstrate how to provide high-quality recommendations for articles with short shelf lives by using <a href="https://aws.amazon.com/what-is/embeddings-in-machine-learning/" rel="noopener" target="_blank">text embeddings</a> in <a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a>. Amazon Bedrock a fully managed service that offers a choice of high-performing foundation models (FMs) from leading artificial intelligence (AI) companies like AI21 Labs, Anthropic, Cohere, Meta, Mistral AI, Stability AI, and Amazon through a single API, along with a broad set of capabilities to build generative AI applications with security, privacy, and responsible AI.</p> 
<p>Embeddings are a mathematical representation of a piece of information such as a text or an image. Specifically, they are a vector or ordered list of numbers. This representation helps capture the meaning of the image or text in such a way that you can use it to determine how similar images or text are to each other by taking their <a href="https://en.wikipedia.org/wiki/Metric_space" rel="noopener" target="_blank">distance</a> from each other in the embedding space. For our post, we use the <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html" rel="noopener" target="_blank">Amazon Titan Text Embeddings</a> model.</p> 
<h2>Solution overview</h2> 
<p>By combining the benefits of Amazon Titan Text Embeddings on Amazon Bedrock with the real-time nature of Amazon Personalize, we can recommend articles to interested users in an intelligent way within seconds of the article being published. Although Amazon Personalize can provide articles shortly after they’re published, it generally takes a few hours (and a filter to select items from the correct time frame) to surface items to the right users. For our use case, we want to recommend articles immediately after they’re published.</p> 
<p>The following diagram shows the architecture of the solution and the high-level steps of the workflow. The architecture follows AWS best practices to use managed and serverless services where possible.</p> 
<p><img alt="" class="alignnone wp-image-83613 size-full" height="1080" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/14/ML-16651-1.png" width="1646" /></p> 
<p>The workflow consists of the following steps:</p> 
<ol> 
 <li>A trigger invokes an <a href="http://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> function every time a new article is published, which runs Steps 2–5.</li> 
 <li>A text embedding model hosted on Amazon Bedrock creates an embedding of the text of the article.</li> 
 <li>An <a href="https://aws.amazon.com/sagemaker/" rel="noopener" target="_blank">Amazon SageMaker</a> hosted model assigns the article to a cluster of similar articles.</li> 
 <li>An Amazon Bedrock hosted model can also generate headlines and summaries of the new article if needed.</li> 
 <li>The new articles are added to <a href="https://aws.amazon.com/dynamodb/" rel="noopener" target="_blank">Amazon DynamoDB</a> with information on their type and when they were published, with a Time-To-Live (TTL) representing when the articles are no longer considered breaking news.</li> 
 <li>When users arrive at the website, their requests are processed by <a href="https://aws.amazon.com/api-gateway/" rel="noopener" target="_blank">Amazon API Gateway</a>.</li> 
 <li>API Gateway makes a request to Amazon Personalize to learn what individual articles and article types a reader is most interested in, which can be directly shown to the reader.</li> 
 <li>To recommend breaking news articles, a call is made to DynamoDB to determine what articles have been recently published of each type. This allows newly published articles to be shown to interested readers in seconds.</li> 
 <li>As users read articles, their interactions are streamed using <a href="https://aws.amazon.com/kinesis/data-streams/" rel="noopener" target="_blank">Amazon Kinesis Data Streams</a> to an Amazon Personalize <a href="https://docs.aws.amazon.com/personalize/latest/dg/event-get-tracker.html" rel="noopener" target="_blank">event tracker</a>.</li> 
 <li>The Amazon Personalize event tracker updates the deployed personalization models within 1–2 seconds.</li> 
</ol> 
<h2>Prerequisites</h2> 
<p>To implement the proposed solution, you should have the following:</p> 
<ul> 
 <li>An <a href="https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&amp;client_id=signup" rel="noopener" target="_blank">AWS account</a> and familiarity with Amazon Personalize, SageMaker, DynamoDB, and Amazon Bedrock.</li> 
 <li>The Amazon Titan Text Embeddings V2 model enabled on Amazon Bedrock. You can confirm it’s enabled on the <strong>Model access </strong>page of the Amazon Bedrock console. If Amazon Titan Text Embeddings is enabled, the access status will show as <strong>Access granted</strong>, as shown in the following screenshot. You can enable access to the model by choosing <strong>Manage model access</strong>, selecting <strong>Amazon </strong><u>Titan Text Embeddings V2</u>, and then choosing <strong>Save Changes</strong>.</li> 
</ul> 
<p><img alt="" class="alignnone wp-image-83614 size-full" height="548" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/14/ML-16651-2.png" width="2572" /></p> 
<ul> 
 <li>A <a href="https://docs.aws.amazon.com/sagemaker/latest/dg/sm-domain.html" rel="noopener" target="_blank">SageMaker domain</a>. You can <a href="https://docs.aws.amazon.com/sagemaker/latest/dg/onboard-quick-start.html" rel="noopener" target="_blank">onboard a SageMaker domain</a> by using the <strong>Set up for single user (Quick setup)</strong> option from the SageMaker console.</li> 
 <li>Either an <a href="https://aws.amazon.com/opensearch-service/features/serverless/" rel="noopener" target="_blank">Amazon OpenSearch Service</a> <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/createupdatedomains.html" rel="noopener" target="_blank">domain</a> or an <a href="https://aws.amazon.com/opensearch-service/features/serverless/" rel="noopener" target="_blank">Amazon OpenSearch Serverless</a> <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless-collections.html" rel="noopener" target="_blank">collection</a>.</li> 
</ul> 
<h2>Create embeddings of the text of previously published articles</h2> 
<p>First, you need to load a set of historically published articles so you have a history of user interactions with those articles and then create embeddings for them using <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html" rel="noopener" target="_blank">Amazon Titan Text Embeddings</a>. AWS also has machine learning (ML) services that can perform tasks such as translation, summarization, and the identification of an article’s tags, title, or genre, if required. The following code snippet shows how to generate embeddings using Amazon Titan Text Embeddings:</p> 
<pre><code class="lang-python">def titan_embeddings(text, bedrock_client):
    prompt = f"{text}"
    body = json.dumps({
        "inputText": prompt,
    })
        
    model_id = 'amazon.titan-embed-text-v2:0'
    accept = 'application/json' 
    content_type = 'application/json'
        
    response = bedrock_client.invoke_model(
        body=body, 
        modelId=model_id, 
        accept=accept, 
        contentType=content_type
    )
        
    response_body = json.loads(response['body'].read())
    return response_body.get('embedding')
</code></pre> 
<h2>Train and deploy a clustering model</h2> 
<p>Next, you deploy a clustering model for the historical articles. A clustering model identifies clusters of article embeddings and assigns each cluster an ID. In this case, we use a k-means model hosted on SageMaker, but you can use a different clustering approach if you prefer.</p> 
<p>The following code snippet is an example of how to create a list of the text embeddings using the Python function above and then train a k-means cluster for article embeddings. In this case, the choice of 100 clusters is arbitrary. You should experiment to find a number that is best for your use case. The instance type represents the <a href="https://aws.amazon.com/pm/ec2/?gclid=EAIaIQobChMI1LWulabghQMVDp9aBR2j_w2nEAAYASAAEgLMd_D_BwE&amp;trk=36c6da98-7b20-48fa-8225-4784bced9843&amp;sc_channel=ps&amp;ef_id=EAIaIQobChMI1LWulabghQMVDp9aBR2j_w2nEAAYASAAEgLMd_D_BwE:G:s&amp;s_kwcid=AL!4422!3!536392622533!e!!g!!ec2%20instance!11198711716!118263957108" rel="noopener" target="_blank">Amazon Elastic Compute Cloud</a> (Amazon EC2) compute instance that runs the SageMaker k-means training job. For detailed information on which instance types fit your use case and their performance capabilities, see <a href="https://aws.amazon.com/ec2/instance-types/" rel="noopener" target="_blank">Amazon EC2 Instance types</a>. For information about pricing for these instance types, see <a href="https://aws.amazon.com/ec2/pricing/" rel="noopener" target="_blank">Amazon EC2 Pricing</a>. For information about available SageMaker notebook instance types, see <a href="https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateNotebookInstance.html#sagemaker-CreateNotebookInstance-request-InstanceType" rel="noopener" target="_blank">CreateNotebookInstance</a>. For most experimentation, you should use an ml.t3.medium instance. This is the default instance type for CPU-based SageMaker images, and is available as part of the <a href="https://aws.amazon.com/free/" rel="noopener" target="_blank">AWS Free Tier</a>.</p> 
<pre><code class="lang-python">text_embeddings_list = []
for text in text_list:
    text_embeddings_list.append(titan_embeddings(text, bedrock_client))

num_clusters = 100

kmeans = KMeans(
    role=role,
    instance_count=1,
    instance_type="ml.t3.medium",
    output_path="s3://your_unique_s3bucket_name/",
    k=num_clusters,
    num_trials=num_clusters,
    epochs=10
)

kmeans.fit(kmeans.record_set(np.asarray(text_embeddings_list, dtype=np.float32)))
</code></pre> 
<p>After you finish training and deploying the clustering model, you can assign a cluster ID to each of the historical articles by passing their embeddings through the k-means (or other) clustering model. Also, importantly, you assign clusters to any articles you consider breaking news (article shelf life can vary from a couple of days to a couple of hours depending on the publication).</p> 
<h2>Set up a DynamoDB table</h2> 
<p>The next step of the process is to set up a DynamoDB table to contain the breaking news articles, their identifiers, and their clusters. This DynamoDB table will help you later when you try to query the mapping of the article item ID with the cluster ID.</p> 
<p>The breaking news table has the following attributes:</p> 
<ul> 
 <li><strong>Article cluster ID</strong> – An initial cluster ID</li> 
 <li><strong>Article ID</strong> – The ID of the article (numeric for this example)</li> 
 <li><strong>Article timestamp</strong> – The time when the article was created</li> 
 <li><strong>Article genre</strong> – The genre of article, such as tech, design best practices, and so on</li> 
 <li><strong>Article language </strong>– A two-letter language code of the article</li> 
 <li><strong>Article text</strong> – The actual article text</li> 
</ul> 
<p>The article cluster ID is the&nbsp;partition key&nbsp;and the article timestamp (in Unix Epoch Time) is the&nbsp;sort key&nbsp;for the breaking news table.</p> 
<h2>Update the article interactions dataset with article clusters</h2> 
<p>When you’re <a href="https://docs.aws.amazon.com/personalize/latest/dg/getting-started.html" rel="noopener" target="_blank">creating your Amazon Personalize user personalization campaign</a>, the <a href="https://docs.aws.amazon.com/personalize/latest/dg/interactions-datasets.html" rel="noopener" target="_blank">item interactions dataset</a> represents the user interactions history with your items. For our use case, we train our recommender on the article clusters instead of the individual articles. This will give the model the opportunity to recommend based on the cluster-level interactions and understand user preferences to article types as opposed to individual articles. That way, when a new article is published, we simply have to identify what type of article it is, and we can immediately recommend it to interested users.</p> 
<p>To do so, you need to update the interactions dataset, replacing the individual article ID with the cluster ID of the article and store the item interactions dataset in an <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) bucket, at which point it can be brought into Amazon Personalize.</p> 
<h2>Create an Amazon Personalize user personalization campaign</h2> 
<p>The <a href="https://docs.aws.amazon.com/personalize/latest/dg/user-personalization-recipes.html" rel="noopener" target="_blank">USER_PERSONALIZATION recipe</a> generates a list of recommendations for a specific user subject to the constraints of filters added to it. This is useful for populating home pages of websites and subsections where specific article types, products, or other pieces of content are focused on. Refer to the following <a href="https://github.com/aws-samples/amazon-personalize-samples/tree/master/next_steps/core_use_cases/user_personalization" rel="noopener" target="_blank">Amazon Personalize user personalization sample</a> on GitHub for step-by-step instructions to create a user personalization model.</p> 
<p>The steps in an Amazon Personalize workflow are as follows:</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/data-prep-ds-group.html" rel="noopener" target="_blank">Create a dataset group</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/data-prep.html" rel="noopener" target="_blank">Prepare and import data</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/create-recommenders-custom-solutions.html" rel="noopener" target="_blank">Create recommenders or custom resources</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/getting-recommendations.html" rel="noopener" target="_blank">Get recommendations</a>.</li> 
</ol> 
<p>To create and deploy a user personalization campaign, you first need to create a user personalization solution. A <em>solution</em> is a combination of a dataset group and a recipe, which is basically a set of instructions for Amazon Personalize for how to prepare a model to solve a specific type of business use case. After this, you train a solution version, then deploy it as a campaign.</p> 
<p>This following code snippet shows how to <a href="https://docs.aws.amazon.com/personalize/latest/dg/customizing-solution-config.html" rel="noopener" target="_blank">create a user personalization solution resource:</a></p> 
<pre><code class="lang-python">create_solution_response = personalize.create_solution (
    name = "personalized-articles-solution”,
    datasetGroupArn = dataset_group_arn,
    recipeArn = "arn:aws:personalize:::recipe/aws-user-personalization-v2",
)
solution_arn = create_solution_response['solutionArn']
</code></pre> 
<p>The following code snippet shows how to <a href="https://docs.aws.amazon.com/personalize/latest/dg/creating-a-solution-version.html">create a user personalization solution version resource</a>:</p> 
<pre><code class="lang-python">create_solution_version_response = personalize.create_solution_version(
   solutionArn = solution_arn
)
solution_version_arn = create_solution_version_response['solutionVersionArn']
</code></pre> 
<p>The following code snippet shows how to <a href="https://docs.aws.amazon.com/personalize/latest/dg/campaigns.html">create a user personalization campaign resource</a>:</p> 
<pre><code class="lang-python">create_campaign_response = personalize.create_campaign (
   name = "personalized-articles-campaign”,
   solutionVersionArn = solution_version_arn,
)
campaign_arn = create_campaign_response['campaignArn']
</code></pre> 
<h2>Deliver a curated and hyper-personalized breaking news experience</h2> 
<p>Articles for the breaking news section of the front page can be drawn from the Amazon Personalize campaign you trained on the article clusters in the previous section. This model identifies the types of articles aligned with each user’s preferences and interests.</p> 
<p>The articles of this type can then be obtained by querying DynamoDB for all articles of that type, then selecting the most recent ones of each relevant type. This solution also allows the editorial team a degree of curation over the diversity of articles shown to individual users. This makes sure users can see the breadth of content available on the site and see a diverse array of perspectives while still having a hyper-personalized experience.</p> 
<p>This is accomplished by setting a maximum number of articles that can be shown per type (a value that can be determined experimentally or by the editorial team). The most recently published articles, up to the maximum, can be selected from each cluster until the desired number of articles is obtained.</p> 
<p>The following Python function obtains the most recently published articles (as measured by their timestamp) in the article cluster. In production, the individual articles should have a TTL representing the shelf life of the articles. The following code assumes the article IDs are numeric and increase over time. If you want to use string values for your article IDs and the article’s timestamp as the sort key for this table, you’ll need to adjust the code.</p> 
<p>The following arguments are passed to the function:</p> 
<ul> 
 <li><strong>cluster (str or int) </strong>– A string or integer representing the cluster in question for which we want to obtain the list of interested users</li> 
 <li><strong>dynamo_client </strong>– A Boto3 <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html">DynamoDB client</a></li> 
 <li><strong>table_name (str) </strong>– The table name of the DynamoDB table in which we store the information</li> 
 <li><strong>index_name (str) </strong>– The name of the index</li> 
 <li><strong>max_per_cluster (int) </strong>– The maximum number of items to pull per cluster</li> 
</ul> 
<pre><code class="lang-python">def query_dynamo_db_articles(
	cluster,
	index_name, 
	dynamo_client, 
	table_name, 
	max_per_cluster):

	arguments = {
		"TableName": table_name,
		"IndexName" : index_name,
		"ScanIndexForward": False,
		"KeyConditionExpression": "articleClusterId = :V1",
		"ExpressionAttributeValues": {
		":V1": {"S": str(cluster)}
	},
        "Limit": max_per_cluster
}

return dynamo_client.query(**arguments)
</code></pre> 
<p>Using the preceding function, the following function selects the relevant articles in each cluster recommended by the Amazon Personalize user personalization model that we created earlier and continues iterating through each cluster until it obtains the maximum desired number of articles. Its arguments are as follows:</p> 
<ul> 
 <li><strong>personalize_runtime </strong>– A Boto3 client representing <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/personalize-runtime.html" rel="noopener" target="_blank">Amazon Personalize Runtime</a></li> 
 <li><strong>personalize_campaign </strong>– The campaign ARN generated when you deployed the user personalization campaign</li> 
 <li><strong>user_id (str)</strong> – The user ID of the reader</li> 
 <li><strong>dynamo_client </strong>– A Boto3 DynamoDB client</li> 
 <li><strong>table_name (str) </strong>– The table name of the DynamoDB table storing the information</li> 
 <li><strong>index_name (str) </strong>– The name of the index</li> 
 <li><strong>max_per_cluster (str) </strong>– The maximum number of articles to pull per cluster</li> 
 <li><strong>desired_items (int) </strong>– The total number of articles to return</li> 
</ul> 
<pre><code class="lang-python">def breaking_news_cluster_recommendation(personalize_runtime,
	personalize_campaign, 
	user_id,
	dynamo_client, 
	table_name,
	index_name,
	max_per_cluster,
	desired_items):


	recommendation = personalize_runtime.get_recommendations(
		campaignArn=personalize_campaign, 
		userId=user_id
	) # Returns recommended clusterId list

	item_count = 0
	item_list = []

	for cluster_number in recommendation['itemList']:
		cluster = cluster_number['itemId']
		dynamo_query_response = query_dynamo_db_articles(
			cluster,
			index_name,
			dynamo_client,
			table_name,
			max_per_cluster
		)

		for item in dynamo_query_response['Items']:
			item_list.append(item)
			item_count += 1
			if item_count == desired_items:
				break
			if item_count == desired_items:
				break
				
	return item_list
</code></pre> 
<h2>Keep recommendations up to date for users</h2> 
<p>When users interact with an article, the interactions are sent to an event tracker. However, unlike a typical Amazon Personalize deployment, in this case we send an interaction as if it occurred with the cluster the article is a member of. There are several ways to do this; one is to embed the article’s cluster in its metadata along with the article ID so they can be fed back to the event tracker. Another is to look up the article’s cluster using its ID in some form of lightweight cache (or key-value database).</p> 
<p>Whichever way you choose, after you obtain the article’s cluster, you stream in an interaction with it using the event tracker.</p> 
<p>The following code snippet sets up the event tracker:</p> 
<pre><code class="lang-python">create_event_tracker_response = personalize.create_event_tracker(
    name = event_tracker_name,
    datasetGroupArn=dataset_group_arn
)
</code></pre> 
<p>The following code snippet feeds in new interactions to the event tracker:</p> 
<pre><code class="lang-python">event_tracker_id = create_event_tracker_response['trackingId']

response = personalize_events.put_events(
    trackingId=event_tracker_id,
    userId=sample_user,
    sessionId=session_id, # a unique id for this users session
    eventList=[]# contains a list of up to 10 item-interactions
)
</code></pre> 
<p>These new interactions will cause Amazon Personalize to update its recommendations in real time. Let’s see what this looks like in practice.</p> 
<p>With a sample dataset derived from the <a href="https://www.kaggle.com/datasets/gspmoreira/articles-sharing-reading-from-cit-deskdrop" rel="noopener" target="_blank">CI&amp;T DeskDrop dataset</a>, a user logging in to their homepage would see these articles. (The dataset is a mixture of Portuguese and English articles; the raw text has been translated but the titles have not. The solution described in this post works for multilingual audiences without requiring separate deployments.) All the articles shown are considered breaking news, meaning we haven’t tracked interactions with them in our dataset and they are being recommended using the clustering techniques described earlier.</p> 
<p><img alt="" class="alignnone wp-image-83649 size-full" height="334" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/15/Breakingnews-16652-2.png" width="1571" /></p> 
<p>However, we can interact with the more technical articles, as shown in the following screenshot.</p> 
<p><img alt="" class="alignnone size-full wp-image-83916" height="861" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/16/Screenshot-2024-08-16-at-6.06.14 PM.png" width="2547" /></p> 
<p>When we refresh our recommendations, the page is updated.</p> 
<p><img alt="" class="alignnone wp-image-83657 size-full" height="278" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/15/image017-16652.png" width="1573" /></p> 
<p>Let’s change our behavior and interact with articles more about design best practices and career development.</p> 
<p><img alt="" class="alignnone size-full wp-image-83917" height="889" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/16/Screenshot-2024-08-16-at-6.12.04 PM.png" width="2621" /></p> 
<p>We get the following recommendations.</p> 
<p><img alt="" class="alignnone wp-image-83668 size-large" height="220" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/15/image029-16652-1024x220.png" width="1024" /></p> 
<p>If we limit the number of articles that we can draw per cluster, we can also enforce a bit more diversity in our recommendations.</p> 
<p><img alt="" class="alignnone wp-image-83669 size-large" height="210" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/15/image031-16652-1024x210.png" width="1024" /></p> 
<p>As new articles are added as part of the news publishing process, the articles are saved to an S3 bucket first. A Lambda trigger on the bucket invokes a series of steps:</p> 
<ol> 
 <li>Generate an embedding of the text of the article using the model on Amazon Bedrock.</li> 
 <li>Determine the cluster ID of the article using the k-means clustering model on SageMaker that you trained earlier.</li> 
 <li>Store the relevant information on the article in a DynamoDB table.</li> 
</ol> 
<h2>Clean up</h2> 
<p>To avoid incurring future charges, delete the resources you created while building this solution:</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/sagemaker/latest/dg/gs-studio-delete-domain.html#gs-studio-delete-domain-studio" rel="noopener" target="_blank">Delete the SageMaker resources</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/gs-cleanup.html" rel="noopener" target="_blank">Delete the Amazon Personalize resources</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/solutions/latest/research-service-workbench-on-aws/deleting-the-amazon-dynamodb-tables.html" rel="noopener" target="_blank">Delete the Amazon DynamoDB tables</a>.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>In this post, we described how you can recommend breaking news to a user using AWS AI/ML services. By taking advantage of the power of Amazon Personalize and Amazon Titan Text Embeddings on Amazon Bedrock, you can show articles to interested users within seconds of them being published.</p> 
<p>As always, AWS welcomes your feedback. Leave your thoughts and questions in the comments section. To learn more about the services discussed in this blog, you can sign up for an <a href="https://skillbuilder.aws/" rel="noopener" target="_blank">AWS Skill Builder</a> account, where you can find free digital courses on Amazon Personalize, Amazon Bedrock, Amazon SageMaker and other AWS services.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><img alt="" class="wp-image-83918 size-full alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/16/ebolme-1.jpg" width="100" /><strong>Eric Bolme</strong>&nbsp;is a Specialist Solution Architect with AWS based on the East Coast of the United States. He has 8 years of experience building out a variety of deep learning and other AI use cases and focuses on Personalization and Recommendation use cases with AWS.</p> 
<p style="clear: both;"><img alt="" class="alignleft wp-image-83054 size-full" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/08/12/djoydeep.jpg" width="100" /><strong>Joydeep Dutta</strong> is a Principal Solutions Architect at AWS. Joydeep enjoys working with AWS customers to migrate their workloads to the cloud, optimize for cost, and help with architectural best practices. He is passionate about enterprise architecture to help reduce cost and complexity in the enterprise. He lives in New Jersey and enjoys listening to music and enjoying the outdoors in his spare time.</p>
