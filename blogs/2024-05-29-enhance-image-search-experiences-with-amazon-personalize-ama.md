---
title: "Enhance image search experiences with Amazon Personalize, Amazon OpenSearch Service, and Amazon Titan Multimodal Embeddings in Amazon Bedrock"
url: "https://aws.amazon.com/blogs/machine-learning/enhance-image-search-experiences-with-amazon-personalize-amazon-opensearch-service-and-amazon-titan-multimodal-embeddings-in-amazon-bedrock/"
date: "Wed, 29 May 2024 16:23:36 +0000"
author: "Maysara Hamdan"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-personalize/feed/"
---
<p>A variety of different techniques have been used for returning images relevant to search queries. Historically, the idea of creating a joint embedding space to facilitate image captioning or text-to-image search has been of interest to machine learning (ML) practitioners and businesses for quite a while. <a href="https://openai.com/research/clip" rel="noopener" target="_blank">Contrastive Language–Image Pre-training (CLIP)</a> and <a href="https://github.com/salesforce/BLIP?tab=readme-ov-file" rel="noopener" target="_blank">Bootstrapping Language-Image Pre-training (BLIP)</a> were the first two open source models that achieved near-human results on the task. More recently, however, there has been a trend to use the same techniques used to train powerful generative models to create multimodal models that map text and images to the same embedding space to achieve state-of-the-art results.</p> 
<p>In this post, we show how to use <a href="https://aws.amazon.com/personalize/" rel="noopener" target="_blank">Amazon Personalize</a> in combination with <a href="https://aws.amazon.com/opensearch-service/" rel="noopener" target="_blank">Amazon OpenSearch Service</a> and <a href="https://aws.amazon.com/bedrock/titan/" rel="noopener" target="_blank">Amazon Titan Multimodal Embeddings</a> from <a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a> to enhance a user’s image search experience by using learned user preferences to further personalize image searches in accordance with a user’s individual style.</p> 
<h2>Solution overview</h2> 
<p>Multimodal models are being used in text-to-image searches across a variety of industries. However, one area where these models fall short is in incorporating individual user preferences into their responses. A user searching for images of a bird, for example, could have many different desired results.</p> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="bird 1" class="aligncenter wp-image-76480 size-full" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image001.png" width="238" /></td> 
   <td><img alt="bird 2" class="aligncenter size-full wp-image-76479" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image002.png" width="238" /></td> 
   <td><img alt="bird 3" class="aligncenter size-full wp-image-76478" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image003.png" width="238" /></td> 
  </tr> 
  <tr> 
   <td><img alt="bird 4" class="aligncenter size-full wp-image-76477" height="241" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image004.png" width="238" /></td> 
   <td><img alt="bird 5" class="aligncenter size-full wp-image-76476" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image005.png" width="238" /></td> 
   <td><img alt="bird 6" class="aligncenter size-full wp-image-76475" height="237" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image006.png" width="238" /></td> 
  </tr> 
 </tbody> 
</table> 
<p>In an ideal world, we can learn a user’s preferences from their previous interactions with images they either viewed, favorited, or downloaded, and use that to return contextually relevant images in line with their recent interactions and style preferences.</p> 
<p>Implementing the proposed solution includes the following high-level steps:</p> 
<ol> 
 <li>Create embeddings for your images.</li> 
 <li>Store embeddings in a data store.</li> 
 <li>Create a cluster for the embeddings.</li> 
 <li>Update the image interactions dataset with the image cluster.</li> 
 <li>Create an Amazon Personalize personalized ranking solution.</li> 
 <li>Serve user search requests.</li> 
</ol> 
<h2>Prerequisites</h2> 
<p>To implement the proposed solution, you should have the following:</p> 
<ul> 
 <li>An <a href="https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&amp;client_id=signup" rel="noopener" target="_blank">AWS account</a> and familiarity with Amazon Personalize, <a href="https://aws.amazon.com/sagemaker/" rel="noopener" target="_blank">Amazon SageMaker</a>, OpenSearch Service, and Amazon Bedrock.</li> 
 <li>The Amazon Titan Multimodal Embeddings model enabled in Amazon Bedrock. You can confirm it’s enabled on the <strong>Model access </strong>page of the Amazon Bedrock console. If Amazon Titan Multimodal Embeddings is enabled, the access status will show as <strong>Access granted</strong>, as shown in the following screenshot. You can enable access to the model by choosing <strong>Manage model access</strong>, selecting <strong>Amazon Titan Multimodal Embeddings G1</strong>, and then choosing <strong>Save Changes.</strong></li> 
</ul> 
<p><img alt="amazon bedrock model access" class="aligncenter size-full wp-image-76474" height="670" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image007.png" style="margin: 10px 0px 10px 0px; border: 1px solid #cccccc;" width="1184" /></p> 
<ul> 
 <li>A <a href="https://docs.aws.amazon.com/sagemaker/latest/dg/sm-domain.html" rel="noopener" target="_blank">SageMaker domain</a>. You can <a href="https://docs.aws.amazon.com/sagemaker/latest/dg/onboard-quick-start.html" rel="noopener" target="_blank">onboard a SageMaker domain</a> by using the <strong>Set up for single users </strong>procedure from the SageMaker console.</li> 
 <li>Either an OpenSearch Service collection or a domain. For <a href="https://aws.amazon.com/opensearch-service/features/serverless/" rel="noopener" target="_blank">Amazon OpenSearch Serverless</a>, you can <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless-collections.html" rel="noopener" target="_blank">create a collection</a>. For OpenSearch Service, you can <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/createupdatedomains.html" rel="noopener" target="_blank">create a domain</a>.</li> 
</ul> 
<h2>Create embeddings for your images</h2> 
<p><a href="https://aws.amazon.com/what-is/embeddings-in-machine-learning/" rel="noopener" target="_blank">Embeddings</a> are a mathematical representation of a piece of information such as a text or an image. Specifically, they are a <em>vector</em> or ordered list of numbers. This representation helps capture the meaning of the image or text in such a way that you can use it to determine how similar images or text are to each other by taking their <a href="https://en.wikipedia.org/wiki/Metric_space" rel="noopener" target="_blank">distance</a> from each other in the embedding space.</p> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="bird" class="aligncenter size-full wp-image-76480" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image001.png" width="238" /></td> 
   <td>→ [-0.020802604, -0.009943095, 0.0012887075, -0….</td> 
  </tr> 
 </tbody> 
</table> 
<p>As a first step, you can use the Amazon Titan Multimodal Embeddings model to generate embeddings for your images. With the Amazon Titan Multimodal Embeddings model, we can use an actual bird image or text like “bird” as an input to generate an embedding. Furthermore, these embeddings will be close to each other when the distance is measured by an appropriate distance metric in a vector database.</p> 
<p>The following code snippet shows how to generate embeddings for an image or a piece of text using Amazon Titan Multimodal Embeddings:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">def generate_embeddings_with_titan(image=None, text=None):
    user_input = {}

    if image is not None:
        user_input["inputImage"] = image
    if text is not None:
        user_input["inputText"] = text

    if not user_input:
        raise ValueError("One user input of an image or a text is required")

    body = json.dumps(user_input)

    response = bedrock_runtime.invoke_model(
        body=body,
        modelId="amazon.titan-embed-image-v1",
        accept="application/json",
        contentType="application/json"
    )

    response_body = json.loads(response.get("body").read())

    embedding_error = response_body.get("message")

    if finish_reason is not None:
        raise EmbedError(f"Embeddings generation error: {embedding_error}")

    return response_body.get("embedding")</code></pre> 
</div> 
<p>It’s expected that the image is base64 encoded in order to create an embedding. For more information, see <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-titan-embed-mm.html" rel="noopener" target="_blank">Amazon Titan Multimodal Embeddings G1</a>. You can create this encoded version of your image for many image file types as follows:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">with open(Image_Filepath+ "/" + image, "rb") as image_file:
     input_image = base64.b64encode(image_file.read()).decode('utf8')</code></pre> 
</div> 
<p>In this case, <code>input_image</code> can be directly fed to the embedding function you generated.</p> 
<h2>Create a cluster for the embeddings</h2> 
<p>As a result of the previous step, a vector representation for each image has been created by the Amazon Titan Multimodal Embeddings model. Because the goal is to create more personalize image search influenced by the user’s previous interactions, you create a cluster out of the image embeddings to group similar images together. This is useful because will force the downstream re-ranker, in this case an Amazon Personalize personalized ranking model, to learn user presences for specific image styles as opposed to their preference for individual images.</p> 
<p>In this post, to create our image clusters, we use an algorithm made available through the fully managed ML service SageMaker, specifically the <a href="https://docs.aws.amazon.com/sagemaker/latest/dg/k-means.html" rel="noopener" target="_blank">K-Means clustering algorithm</a>. You can use any clustering algorithm that you are familiar with. K-Means clustering is a widely used method for clustering where the aim is to partition a set of objects into K clusters in such a way that the sum of the squared distances between the objects and their assigned cluster mean is minimized. The appropriate value of <code>K</code> depends on the data structure and the problem being solved. Make sure to choose the right value of <code>K</code>, because a small value can result in under-clustered data, and a large value can cause over-clustering.</p> 
<p>The following code snippet is an example of how to create and train a K-Means cluster for image embeddings. In this example, the choice of 100 clusters is arbitrary—you should experiment to find a number that is best for your use case. The instance type represents the <a href="https://aws.amazon.com/pm/ec2/?gclid=EAIaIQobChMI1LWulabghQMVDp9aBR2j_w2nEAAYASAAEgLMd_D_BwE&amp;trk=36c6da98-7b20-48fa-8225-4784bced9843&amp;sc_channel=ps&amp;ef_id=EAIaIQobChMI1LWulabghQMVDp9aBR2j_w2nEAAYASAAEgLMd_D_BwE:G:s&amp;s_kwcid=AL!4422!3!536392622533!e!!g!!ec2%20instance!11198711716!118263957108" rel="noopener" target="_blank">Amazon Elastic Compute Cloud</a> (Amazon EC2) compute instance that runs the SageMaker K-Means training job. For detailed information on which instance types fit your use case, and their performance capabilities, see <a href="https://aws.amazon.com/ec2/instance-types/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud instance types</a>. For information about pricing for these instance types, see <a href="https://aws.amazon.com/ec2/pricing/" rel="noopener" target="_blank">Amazon EC2 Pricing</a>. For information about available SageMaker notebook instance types, see <a href="https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_CreateNotebookInstance.html#sagemaker-CreateNotebookInstance-request-InstanceType" rel="noopener" target="_blank">CreateNotebookInstance</a>.</p> 
<p>For most experimentation, you should use an ml.t3.medium instance. This is the default instance type for CPU-based SageMaker images, and is available as part of the <a href="https://aws.amazon.com/free/" rel="noopener" target="_blank">AWS Free Tier</a>.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">num_clusters = 100

kmeans = KMeans(
    role=role,
    instance_count=1,
    instance_type="ml.t3.medium",
    output_path="s3://your_unique_s3bucket_name/",
    k=num_clusters,
    num_trials=num_clusters,
    epochs=10
)

kmeans.fit(kmeans.record_set(np.asarray(image_embeddings_list, dtype=np.float32)))</code></pre> 
</div> 
<h2>Store embeddings and their clusters in a data store</h2> 
<p>As a result of the previous step, a vector representation for each image has been created and assigned to an image cluster by our clustering model. Now, you need to store this vector such that the other vectors that are nearest to it can be returned in a timely manner. This allows you to input a text such as “<code>bird</code>” and retrieve images that prominently feature birds.</p> 
<p><a href="https://aws.amazon.com/what-is/vector-databases/" rel="noopener" target="_blank">Vector databases</a> provide the ability to store and retrieve vectors as high-dimensional points. They add additional capabilities for efficient and fast lookup of nearest neighbors in the N-dimensional space. They are typically powered by nearest neighbor indexes and built with algorithms like the <a href="https://www.pinecone.io/learn/series/faiss/hnsw/" rel="noopener" target="_blank">Hierarchical Navigable Small World (HNSW)</a> and <a href="https://www.pinecone.io/learn/series/faiss/vector-indexes/#Inverted-File-Index" rel="noopener" target="_blank">Inverted File Index (IVF)</a> algorithms. Vector databases provide additional capabilities like data management, fault tolerance, authentication and access control, and a query engine.</p> 
<p>AWS offers many services for your vector database requirements. OpenSearch Service is one example; it makes it straightforward for you to perform interactive log analytics, real-time application monitoring, website search, and more. For information about using OpenSearch Service as a vector database, see <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/knn.html" rel="noopener" target="_blank">k-Nearest Neighbor (k-NN) search in OpenSearch Service</a>.</p> 
<p>For this post, we use OpenSearch Service as a vector database to store the embeddings. To do this, you need to <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/createupdatedomains.html" rel="noopener" target="_blank">create an OpenSearch Service cluster</a> or use OpenSearch Serverless. Regardless which approach you used for the cluster, you need to create a vector index. <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/indexing.html" rel="noopener" target="_blank">Indexing</a> is the method by which search engines organize data for fast retrieval. To use a k-NN vector index for OpenSearch Service, you need to add the <code>index.knn</code> setting and add one or more fields of the <code>knn_vector</code> data type. This lets you search for points in a vector space and find the nearest neighbors for those points by Euclidean distance or cosine similarity, either of which is acceptable for Amazon Titan Multimodal Embeddings.</p> 
<p>The following code snippet shows how to create an OpenSearch Service index with k-NN enabled to serve as a vector datastore for your embeddings:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">def create_index(opensearch_client, index_name, vector_field_name):
    settings = {
      "settings": {
        "index": {
          "knn": True
        }
      },
      "mappings": {
        "properties": {
            vector_field_name: {
              "type": "knn_vector",
              "dimension": 1024,
              "method": {
                "name": "hnsw",
                "space_type": "l2",
                "engine": "faiss",
                "parameters": {
                  "m": 32
                }
              }
            }
        }
      }
    }
    response = opensearch_client.indices.create(index=index_name, body=settings)
    return bool(response['acknowledged'])</code></pre> 
</div> 
<p>The following code snippet shows how to store an image embedding into the open search service index you just created:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">    embedding_vector = {"_index":index_name,
                        "name": image_name, 
                        "type": "Image",
                        "embedding": image_embedding,
			 "cluster": image_cluster }
    #opensearch_client is your Amazon Opensearch cluster client
    opensearch_client.index(
        index=index_name,
        body=embedding_vector,
        id = str(index),
        refresh = True
    )</code></pre> 
</div> 
<h2>Update the image interactions dataset with the image cluster</h2> 
<p>When <a href="https://docs.aws.amazon.com/personalize/latest/dg/getting-started.html" rel="noopener" target="_blank">creating an Amazon Personalize re-ranker</a>, the <a href="https://docs.aws.amazon.com/personalize/latest/dg/interactions-datasets.html" rel="noopener" target="_blank">item interactions dataset</a> represents the user interaction history with your items. Here, the images represent the items and the interactions could consist of a variety of events, such as a user downloading an image, favoriting it, or even viewing a higher resolution version of it. For our use case, we train our recommender on the image clusters instead of the individual images. This gives the model the opportunity to recommend based on the cluster-level interactions and understand the user’s overall stylistic preferences as opposed to preferences for an individual image in the moment.</p> 
<p>To do so, update the interaction dataset including the image cluster instead of the image ID in the dataset, and store the file in an <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) bucket, at which point it can be brought into Amazon Personalize.</p> 
<h2>Create an Amazon Personalize personalized ranking campaign</h2> 
<p>The <a href="https://docs.aws.amazon.com/personalize/latest/dg/native-recipe-search.html" rel="noopener" target="_blank">Personalized-Ranking recipe</a> generates personalized rankings of items. A <em>personalized ranking</em> is a list of recommended items that are re-ranked for a specific user. This is useful if you have a collection of ordered items, such as search results, promotions, or curated lists, and you want to provide a personalized re-ranking for each of your users. Refer to the following <a href="https://github.com/aws-samples/amazon-personalize-samples/blob/master/next_steps/core_use_cases/personalized_ranking/personalize_ranking_example.ipynb" rel="noopener" target="_blank">example</a> available on GitHub for complete step-by-step instructions on how to create an Amazon Personalize recipe. The high-level steps are as follows:</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/data-prep-ds-group.html" rel="noopener" target="_blank">Create a dataset group</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/data-prep.html" rel="noopener" target="_blank">Prepare and import data</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/create-recommenders-custom-solutions.html" rel="noopener" target="_blank">Create recommenders or custom resources</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/getting-recommendations.html" rel="noopener" target="_blank">Get recommendations</a>.</li> 
</ol> 
<p>We create and deploy a personalized ranking campaign. First, you need to create a personalized ranking solution. A <em>solution</em> is a combination of a dataset group and a recipe, which is basically a set of instructions for Amazon Personalize to prepare a model to solve a specific type of business use case. Then you train a solution version and deploy it as a campaign.</p> 
<p>The following code snippet shows how to <a href="https://docs.aws.amazon.com/personalize/latest/dg/customizing-solution-config.html" rel="noopener" target="_blank">create a Personalized-Ranking solution resource: </a></p> 
<div class="hide-language"> 
 <pre><code class="lang-python">personalized_ranking_create_solution_response = personalize_client.create_solution(
    name = "personalized-image-reranker",
    datasetGroupArn = dataset_group_arn,
    recipeArn = personalized_ranking_recipe_arn
)
personalized_ranking_solution_arn = personalized_ranking_create_solution_response['solutionArn']</code></pre> 
</div> 
<p>The following code snippet shows how to <a href="https://docs.aws.amazon.com/personalize/latest/dg/creating-a-solution-version.html" rel="noopener" target="_blank">create a Personalized-Ranking solution version resource: </a></p> 
<div class="hide-language"> 
 <pre><code class="lang-python">personalized_ranking_create_solution_version_response = personalize_client.create_solution_version(
    solutionArn = personalized_ranking_solution_arn
)

personalized_ranking_solution_version_arn = personalized_ranking_create_solution_version_response['solutionVersionArn']</code></pre> 
</div> 
<p>The following code snippet shows how to <a href="https://docs.aws.amazon.com/personalize/latest/dg/campaigns.html" rel="noopener" target="_blank">create a Personalized-Ranking campaign resource</a>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">create_campaign_response = personalize_client.create_campaign(
        name = "personalized-image-reranker-campaign",
        solutionVersionArn = personalized_ranking_solution_version_arn,
        minProvisionedTPS = 1
        )

personalized_ranking_campaign_arn = create_campaign_response['campaignArn']</code></pre> 
</div> 
<h2>Serve user search requests</h2> 
<p>Now our solution flow is ready to serve a user search request and provide personalized ranked results based on the user’s previous interactions. The search query will be processed as shown in the following diagram.</p> 
<p><img alt="personalized image search architecture " class="aligncenter size-full wp-image-76481" height="687" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/Personalize_Image_Search_Blog.png" width="1221" /></p> 
<p>To setup personalized multimodal search, one would execute the following steps:</p> 
<ol> 
 <li>Multimodal embeddings are created for the image dataset.</li> 
 <li>A clustering model is created in SageMaker, and each image is assigned to a cluster.</li> 
 <li>The unique image IDs are replaced with cluster IDs in the image interactions dataset.</li> 
 <li>An Amazon Personalize personalized ranking model is trained on the cluster interaction dataset.</li> 
 <li>Separately, the image embeddings are added to an OpenSearch Service vector index.</li> 
</ol> 
<p>The following workflow would be executed to process a user’s query:</p> 
<ol> 
 <li><a href="https://aws.amazon.com/api-gateway" rel="noopener" target="_blank">Amazon API Gateway</a> calls an <a href="http://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> function when the user enters a query.</li> 
 <li>The Lambda function calls the same multimodal embedding function to generate an embedding of the query.</li> 
 <li>A k-NN search is performed for the query embedding on the vector index.</li> 
 <li>A personalized score for the cluster ID for each retrieved image is obtained from the Amazon Personalize personalized ranking model.</li> 
 <li>The scores from OpenSearch Service and Amazon Personalize are combined through a weighted mean. The images are re-ranked and returned to the user.</li> 
</ol> 
<p>The weights on each score could be tuned based on the available data and desired outcomes and desired degrees of personalization vs. contextual relevance.</p> 
<p><img alt="Personalized image search weighted score" class="aligncenter size-full wp-image-76472" height="685" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image010.png" width="1221" /></p> 
<p>To see what this looks like in practice, let’s explore a few examples. In our example dataset, all users would, in absence of any personalization, receive the following images if they search for “<code>cat</code>”.</p> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="cat 1" class="aligncenter size-full wp-image-76471" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image011.png" width="238" /></td> 
   <td><img alt="cat 2" class="aligncenter size-full wp-image-76470" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image012.png" width="238" /></td> 
   <td><img alt="cat 3" class="aligncenter size-full wp-image-76469" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image013.png" width="238" /></td> 
  </tr> 
  <tr> 
   <td><img alt="cat 4" class="aligncenter size-full wp-image-76468" height="237" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image014.png" width="238" /></td> 
   <td><img alt="cat 5" class="aligncenter size-full wp-image-76467" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image015.png" width="238" /></td> 
   <td><img alt="cat 6" class="aligncenter size-full wp-image-76466" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image016.png" width="238" /></td> 
  </tr> 
 </tbody> 
</table> 
<p>However, a user who has a history of viewing the following images (let’s call them <code>comic-art-user</code>) clearly has a certain style preference that isn’t addressed by the majority of the previous images.</p> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="comic-art-user 1" class="aligncenter size-full wp-image-76465" height="244" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image017.png" width="238" /></td> 
   <td><img alt="comic-art-user 2" class="aligncenter size-full wp-image-76464" height="239" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image018.png" width="238" /></td> 
   <td><img alt="comic-art-user 3" class="aligncenter size-full wp-image-76463" height="237" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image019.png" width="238" /></td> 
  </tr> 
  <tr> 
   <td><img alt="comic-art-user" class="aligncenter size-full wp-image-76462" height="234" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image020.png" width="238" /></td> 
   <td><img alt="comic-art-user 5" class="aligncenter size-full wp-image-76461" height="241" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image021.png" width="238" /></td> 
   <td><img alt="comic-art-user 6" class="aligncenter size-full wp-image-76460" height="235" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image022.png" width="238" /></td> 
  </tr> 
 </tbody> 
</table> 
<p>By combining Amazon Personalize with the vector database capabilities of OpenSearch Service, we are able to return the following results for cats to our user:</p> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="comic-art-user-cat-1" class="aligncenter size-full wp-image-76459" height="241" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image023.png" width="238" /></td> 
   <td><img alt="comic-art-user-cat-2" class="aligncenter size-full wp-image-76458" height="234" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image024.png" width="238" /></td> 
   <td><img alt="comic-art-user-cat-3" class="aligncenter size-full wp-image-76457" height="235" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image025.png" width="238" /></td> 
  </tr> 
  <tr> 
   <td><img alt="comic-art-user-cat-4" class="aligncenter size-full wp-image-76456" height="247" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image026.png" width="238" /></td> 
   <td><img alt="comic-art-user-cat-5" class="aligncenter size-full wp-image-76455" height="239" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image027.png" width="238" /></td> 
   <td><img alt="comic-art-user-cat-6" class="aligncenter size-full wp-image-76454" height="237" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image028.png" width="238" /></td> 
  </tr> 
 </tbody> 
</table> 
<p>In the following example, a user has been viewing or downloading the following images (let’s call them <code>neon-punk-user</code>).</p> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="neon-punk-user-1" class="aligncenter size-full wp-image-76453" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image029.png" width="238" /></td> 
   <td><img alt="neon-punk-user-2" class="aligncenter size-full wp-image-76452" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image030.png" width="238" /></td> 
   <td><img alt="neon-punk-user-3" class="aligncenter size-full wp-image-76451" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image031.png" width="238" /></td> 
  </tr> 
 </tbody> 
</table> 
<p>They would receive the following personalized results instead of the mostly photorealistic cats that all users would receive absent any personalization.</p> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="neon-punk-user-cat-1" class="aligncenter size-full wp-image-76450" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image032.png" width="238" /></td> 
   <td><img alt="neon-punk-user-cat-2" class="aligncenter size-full wp-image-76449" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image033.png" width="238" /></td> 
   <td><img alt="neon-punk-user-cat-3" class="aligncenter size-full wp-image-76448" height="238" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image034.png" width="238" /></td> 
  </tr> 
 </tbody> 
</table> 
<p>Finally, a user viewed or downloaded the following images (let’s call them <code>origami-clay-user</code>).</p> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="origami-clay-user-1" class="aligncenter size-full wp-image-76447" height="237" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image035.png" width="238" /></td> 
   <td><img alt="origami-clay-user-2" class="aligncenter size-full wp-image-76446" height="235" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image036.png" width="238" /></td> 
   <td><img alt="origami-clay-user-3" class="aligncenter size-full wp-image-76445" height="234" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image037.png" width="238" /></td> 
  </tr> 
 </tbody> 
</table> 
<p>They would receive the following images as their personalized search results.</p> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="origami-clay-user-cat-1" class="aligncenter size-full wp-image-76444" height="235" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image038.png" width="238" /></td> 
   <td><img alt="origami-clay-user-2" class="aligncenter size-full wp-image-76443" height="232" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image039.png" width="238" /></td> 
   <td><img alt="origami-clay-user-3" class="aligncenter size-full wp-image-76442" height="237" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/16/ML-16426-image040.png" width="238" /></td> 
  </tr> 
 </tbody> 
</table> 
<p>These examples illustrate how the search results have been influenced by the users’ previous interactions with other images. By combining the power of Amazon Titan Multimodal Embeddings, OpenSearch Service vector indexing, and Amazon Personalize personalization, we are able to deliver each user relevant search results in alignment with their style preferences as opposed to showing all of them the same generic search result.</p> 
<p>Furthermore, because Amazon Personalize is capable of updating based on changes in the user style preference in real time, these search results would update as the user’s style preferences change, for example if they were a designer working for an ad agency who switched mid-browsing session to working on a different project for a different brand.</p> 
<h2>Clean up</h2> 
<p>To avoid incurring future charges, delete the resources created while building this solution:</p> 
<ol> 
 <li>Delete the <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/gsg.html#gsgdeleting" rel="noopener" target="_blank">OpenSearch Service domain</a> or <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/serverless-manage.html#serverless-delete" rel="noopener" target="_blank">OpenSearch Serverless collection</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/sagemaker/latest/dg/gs-studio-delete-domain.html#gs-studio-delete-domain-studio" rel="noopener" target="_blank">Delete the SageMaker resources</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/gs-cleanup.html" rel="noopener" target="_blank">Delete the Amazon Personalize resources</a>.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>By combining the power of Amazon Titan Multimodal Embeddings, OpenSearch Service vector indexing and search capabilities, and Amazon Personalize ML recommendations, you can boost the user experience with more relevant items in their search results by learning from their previous interactions and preferences.</p> 
<p>For more details on Amazon Titan Multimodal Embeddings, refer to <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/titan-multiemb-models.html" rel="noopener" target="_blank">Amazon Titan Multimodal Embeddings G1 model</a>. For more details on OpenSearch Service, refer to <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/gsg.html" rel="noopener" target="_blank">Getting started with Amazon OpenSearch Service</a>. For more details on Amazon Personalize, refer to the <a href="https://docs.aws.amazon.com/personalize/latest/dg/what-is-personalize.html" rel="noopener" target="_blank">Amazon Personalize Developer Guide</a>.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><img alt="" class="size-full wp-image-63665 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/09/28/maysra.jpeg" width="120" /><strong>Maysara Hamdan</strong>&nbsp;is a Partner Solutions Architect based in Atlanta, Georgia. Maysara has over 15 years of experience in building and architecting Software Applications and IoT Connected Products in Telecom and Automotive Industries. In AWS, Maysara helps partners in building their cloud practices and growing their businesses. Maysara is passionate about new technologies and is always looking for ways to help partners innovate and grow.</p> 
<p style="clear: both;"><img alt="" class="size-full wp-image-63755 alignleft" height="120" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/10/02/ebolme-1.jpg" width="120" /><strong>Eric Bolme</strong>&nbsp;is a Specialist Solution Architect with AWS based on the East Coast of the United States. He has 8 years of experience building out a variety of deep learning and other AI use cases and focuses on Personalization and Recommendation use cases with AWS.</p>
