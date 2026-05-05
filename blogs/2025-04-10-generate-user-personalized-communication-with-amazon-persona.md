---
title: "Generate user-personalized communication with Amazon Personalize and Amazon Bedrock"
url: "https://aws.amazon.com/blogs/machine-learning/generate-user-personalized-communication-with-amazon-personalize-and-amazon-bedrock/"
date: "Thu, 10 Apr 2025 16:43:53 +0000"
author: "Anna Grüebler"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-personalize/feed/"
---
<p>Today, businesses are using AI and generative models to improve productivity in their teams and provide better experiences to their customers. Personalized outbound communication can be a powerful tool to increase user engagement and conversion.</p> 
<p>For instance, as a marketing manager for a video-on-demand company, you might want to send personalized email messages tailored to each individual user—taking into account their demographic information, such as gender and age, and their viewing preferences. You want the messaging and movie recommendations to be both engaging and applicable to the customer. To achieve this, you can use <a href="https://aws.amazon.com/personalize/" rel="noopener" target="_blank">Amazon Personalize</a> to generate user-personalized recommendations and <a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a> to generate the text of the email.</p> 
<p>Amazon Personalize enables your business to improve customer engagement by creating personalized product and content recommendations in websites, applications, and targeted marketing campaigns. You can get started without any prior machine learning (ML) experience, and Amazon Personalize allows you to use APIs to build sophisticated personalization capabilities. Using this service, all your data is encrypted to be private and secure, and is only used to create recommendations for your users.</p> 
<p>Amazon Bedrock is a fully managed service that offers a choice of high-performing foundation models (FMs) from leading AI companies and Amazon through a single API, along with a broad set of capabilities to build generative AI applications with security, privacy, and responsible AI. Using Amazon Bedrock, you can experiment with and evaluate top FMs for your use case, customize the model using fine tuning, or restrict the model output using Retrieval Augmented Generaion (RAG), and build agents that execute tasks using your enterprise systems and data sources.</p> 
<p>In this post, we demonstrate how to use Amazon Personalize and Amazon Bedrock to generate personalized outreach emails for individual users using a video-on-demand use case. This concept can be applied to other domains, such as compelling customer experiences for ecommerce and digital marketing use cases.</p> 
<h2>Solution overview</h2> 
<p>The following diagram shows how you can use Amazon Personalize and Amazon Bedrock to generate user-personalized outreach messages for each user.</p> 
<p><img alt="Workflow Diagram: 1. Import your user, item, and interaction data into Amazon Personalize. 2. Train an Amazon Personalize “Top pics for you” recommender. 3. Get the top recommended movies for each user. 4. Use a prompt template, the recommended movies, and the user demographics to generate the model prompt. 5. Use Amazon Bedrock LLMs to generate personalized outbound communication with the prompt. 6. Share the personalize outbound communication with each of your users." class="alignnone size-full wp-image-100859" height="920" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/03/04/ML-16267-step-flow-1.png" width="2164" /></p> 
<p>The workflow consists of the following steps:</p> 
<ol> 
 <li>Import your user, item, and interaction data into Amazon Personalize. The user and item datasets are not required for Amazon Personalize to generate recommendations, but providing good item and user metadata provides the best results in your trained models.</li> 
 <li>Train an Amazon Personalize “Top picks for you” recommender. Amazon Personalize recommenders are domain-specific resources that generate recommendations. When you create an Amazon Personalize recommender, Amazon Personalize trains the models backing the recommender with the best configurations for the use case. In our example, we use the “<a href="https://docs.aws.amazon.com/personalize/latest/dg/VIDEO_ON_DEMAND-use-cases.html#top-picks-use-case" rel="noopener" target="_blank">Top picks for you</a>” recommender. This recommender generates personalized content recommendations for a user that you specify. With this use case, Amazon Personalize automatically filters videos the user watched.</li> 
 <li>After the model is trained, you can get the top recommended movies for each user by querying the recommender with each user ID through the <a href="https://docs.aws.amazon.com/personalize/latest/dg/API_Operations_Amazon_Personalize_Runtime.html" rel="noopener" target="_blank">Amazon Personalize Runtime API</a>.</li> 
 <li>Combine a predefined prompt template with the top recommendations and user demographic information to generate an enhanced prompt.</li> 
 <li>Use the enhanced prompt in Amazon Bedrock through its API to generate your personalized outbound communication.</li> 
 <li>Amazon Bedrock returns the personalized outbound communication that you can email to your users.</li> 
</ol> 
<p>We go deeper into each of these steps in the following sections. A code sample for this use case is available on <a href="https://github.com/aws-samples/amazon-personalize-samples/tree/master/next_steps/generative_ai/user_personalized_marketing_messaging_with_amazon_personalize_and_gen_ai" rel="noopener" target="_blank">AWS Samples on GitHub</a>.</p> 
<h2>Prerequisites</h2> 
<p>To generate personalized recommendations, you must first set up Amazon Personalize resources. You start by creating your dataset group, loading your data, and then training a recommender. For full instructions, see <a href="https://docs.aws.amazon.com/personalize/latest/dg/getting-started.html" rel="noopener" target="_blank">Getting started tutorials</a>.</p> 
<ol> 
 <li> 
  <ol> 
   <li>Create a dataset group.</li> 
   <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/data-prep-ds-group.html" rel="noopener" target="_blank">Create</a> an <code>Interactions</code> dataset using the following <a href="https://docs.aws.amazon.com/personalize/latest/dg/how-it-works-dataset-schema.html" rel="noopener" target="_blank">schema</a>: 
    <div class="hide-language"> 
     <pre><code class="lang-json">{
    "type": "record"
    "name": "Interactions",
    "namespace": "com.amazonaws.personalize.schema",
    "fields": [
        {
            "name": "USER_ID",
            "type": "string"
        },
        {
            "name": "ITEM_ID",
            "type": "string"
        },
        {
            "name": "TIMESTAMP",
            "type": "long"
        },
        {
            "name": "EVENT_TYPE",
            "type": "string"
        }
    ],
    "version": "1.0"
}</code></pre> 
    </div> <p>Interaction data consists of information about the user interactions with the content in your application. This usually comes from analytics tools or a customer data platform (CDP). The best interaction data to use in Amazon Personalize includes the sequential order of user behavior and the content the user watched or clicked on. For this example, we use the <code>ml-latest-small</code> dataset from the <a href="https://grouplens.org/datasets/movielens/" rel="noopener" target="_blank">MovieLens dataset</a> to simulate user-item interactions.</p></li> 
   <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/bulk-data-import-step.html" rel="noopener" target="_blank">Import the interaction data</a> to Amazon Personalize from <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a>&nbsp;(Amazon S3). For this example, we convert the data to the appropriate format following the steps in the notebook <a href="https://github.com/aws-samples/amazon-personalize-samples/blob/master/next_steps/generative_ai/user_personalized_marketing_messaging_with_amazon_personalize_and_gen_ai/01_Introduction_and_Data_Preparation.ipynb" rel="noopener" target="_blank">01_Introduction_and_Data_Preparation</a>.</li> 
   <li>Item data consists of information about the content that is being interacted with, which generally comes from a content management system (CMS) in video-on-demand use cases. This can be information like the title, description, or movie genre. To provide additional metadata, and also provide a consistent experience for our users, we use a subset of the IMDb Essential Metadata for Movies/TV/OTT dataset. IMDb has multiple datasets available in AWS Data Exchange. For this post, we have extracted and prepared a subset of data for use with the following information from the IMDb Essential Metadata for Movies/TV/OTT (Bulk data) dataset.With this data, create an <code>Items</code> dataset using the following schema: 
    <div class="hide-language"> 
     <pre><code class="lang-json">items_schema = {
    "type": "record",
    "name": "Items",
    "namespace": "com.amazonaws.personalize.schema",
    "fields": [
        {
            "name": "ITEM_ID",
            "type": "string"
        },
        {
            "name": "TITLE",
            "type": "string"
        },
        {
            "name": "YEAR",
            "type": "int"
        },
        {
            "name": "IMDB_RATING",
            "type": "int"
        },
        {
            "name": "IMDB_NUMBEROFVOTES",
            "type": "int"
        },
        {
            "name": "PLOT",
            "type": "string",
            "textual": True
        },
        {
            "name": "US_MATURITY_RATING_STRING",
            "type": "string"
        },
        {
            "name": "US_MATURITY_RATING",
            "type": "int"
        },
        {
            "name": "GENRES",
            "type": "string",
            "categorical": True
        },
        {
            "name": "CREATION_TIMESTAMP",
            "type": "long"
        },
        {
            "name": "PROMOTION",
            "type": "string"
        }
    ],
    "version": "1.0
}</code></pre> 
    </div> </li> 
   <li>Import the item data to Amazon Personalize from Amazon S3. For this example, we convert the data to the appropriate format following the steps in the notebook <a href="https://github.com/aws-samples/amazon-personalize-samples/blob/master/next_steps/generative_ai/user_personalized_marketing_messaging_with_amazon_personalize_and_gen_ai/01_Introduction_and_Data_Preparation.ipynb" rel="noopener" target="_blank">01_Introduction_and_Data_Preparation</a>.<br /> For more information on formatting and importing your interactions and items data from Amazon S3, see <a href="https://docs.aws.amazon.com/personalize/latest/dg/bulk-data-import.html" rel="noopener" target="_blank">Importing bulk records</a>.</li> 
   <li>Create a recommender. In this example, we create a “Top picks for you” recommender.</li> 
  </ol> </li> 
</ol> 
<h2>Get personalized recommendations using Amazon Personalize</h2> 
<p>Now that we have trained the “<a href="https://docs.aws.amazon.com/personalize/latest/dg/VIDEO_ON_DEMAND-use-cases.html#top-picks-use-case" rel="noopener" target="_blank">Top picks for you</a>” recommender, we can generate recommendations for our users. For more details and ways to use Amazon Personalize to get recommendations, see <a href="https://docs.aws.amazon.com/personalize/latest/dg/getting-recommendations.html" rel="noopener" target="_blank">Getting recommendations from Amazon Personalize</a>.We include the item metadata in the response so we can use this information in our outbound communication in the next step.You can use the following code to get recommended movies for each user:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">get_recommendations_response = personalize_runtime.get_recommendations(
    recommenderArn = workshop_recommender_top_picks_arn,
    userId = str(user_id),
    numResults = number_of_movies_to_recommend,
    metadataColumns = {
        "ITEMS": [
            'TITLE', 'PLOT', 'GENRES']
        }
)</code></pre> 
</div> 
<p>In the items dataset, we can specify the metadata columns we want the recommender to return. In this case, we request the <code>Title</code>, <code>Plot</code>, and <code>Genres</code> of the recommended movie. You can request metadata columns only if this feature has been enabled when the <a href="https://docs.aws.amazon.com/personalize/latest/dg/creating-recommenders.html#create-recommender-return-metadata" rel="noopener" target="_blank">recommender was created</a>.</p> 
<p>For an example <code>user_Id</code>, the following movies are recommended:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">Title: There's Something About Mary
Genres: Comedy and Romance
Plot: A man gets a chance to meet up with his dream girl from high school, even though his date with her back then was a complete disaster.

Title: Shakespeare in Love
Genres: Comedy and Drama and History and Romance
Plot: The world's greatest ever playwright, William Shakespeare, is young, out of ideas and short of cash, but meets his ideal woman and is inspired to write one of his most famous plays.

Title: The Birdcage
Genres: Comedy
Plot: A gay cabaret owner and his drag queen companion agree to put up a false straight front so that their son can introduce them to his fiancée's right-wing moralistic parents.
</code></pre> 
</div> 
<h2>Get the user’s favorite movie genre</h2> 
<p>To provide a better personalized outbound communication experience, we determine the user’s favorite movie genre based on the genres of all the movies they have interacted with in the past. There are a number of ways to do this, such as counting the number of interactions per genre for our user. In this example, our sample user’s favorite genre is <code>Comedy</code>.</p> 
<h2>Generate personalized marketing emails with recommended movies</h2> 
<p>To generate personalized marketing emails, we use <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html" rel="noopener" target="_blank">Amazon Bedrock</a>. Amazon Bedrock users must request access to models before they are available for use. Amazon Bedrock is a fully managed service that makes base models from Amazon and third-party model providers accessible through an API.</p> 
<p>To request access, select choose <strong>Model access</strong> in the navigation pane on the Amazon Bedrock console. For more information, see <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html" rel="noopener" target="_blank">Access Amazon Bedrock foundation models</a>.</p> 
<p>In this example, we use <a href="https://aws.amazon.com/bedrock/claude/" rel="noopener" target="_blank">Anthropic’s Claude 3.7</a> on Amazon Bedrock and have defined the following configuration parameters:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python"># The LLM we will be using
model_id = 'us.anthropic.claude-3-7-sonnet-20250219-v1:0'

# The maximum number of tokens to use in the generated response
max_tokens_to_sample = 1000</code></pre> 
</div> 
<p>Let’s generate a simple outreach email using the recommended movies and the following prompt template:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">prompt_template = f'''Write a marketing email advertising several movies available in a video-on-demand streaming platform next week, given the movie and user information below. The movies to recommend and their information is contained in the &lt;movie&gt; tag. Put the email between &lt;email&gt; tags.

&lt;movie&gt;
{movie_list}
&lt;/movie&gt;

Assistant: Email body:
&lt;email&gt;
'''</code></pre> 
</div> 
<p>Using the recommended movies, the full prompt is as follows:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">"Write a marketing email advertising several movies available in a video-on-demand streaming platform next week, given the movie and user information below. The movies to recommend and their information is contained in the &lt;movie&gt; tag. Put the email between &lt;email&gt; tags.
\n
\n
&lt;movie&gt;
\n
[
{
'title': \"There's Something About Mary\",
'genres': 'Comedy and Romance',
'plot': 'A man gets a chance to meet up with his dream girl from high school, even though his date with her back then was a complete disaster.'
},
{
'title': 'Shakespeare in Love',
'genres': 'Comedy and Drama and History and Romance',
'plot': \"The world's greatest ever playwright, William Shakespeare, is young, out of ideas and short of cash, but meets his ideal woman and is inspired to write one of his most famous plays.\"
},
{
'title': 'The Birdcage',
'genres': 'Comedy',
'plot': \"A gay cabaret owner and his drag queen companion agree to put up a false straight front so that their son can introduce them to his fianc\u00e9e's right-wing moralistic parents.\"
}
]
\n
&lt;/movie&gt;
\n
\n
Assistant: Email body:
\n
&lt;email&gt;.
"</code></pre> 
</div> 
<p>We then use an Amazon Bedrock API call to generate the personalized email. For more information, see <a href="https://docs.aws.amazon.com/bedrock/latest/APIReference/API_InvokeModel.html" rel="noopener" target="_blank">Amazon Bedrock API Reference</a>.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">request_body = json.dumps({
    "max_tokens": max_tokens_to_sample,
    "messages": [{"role": "user", "content": prompt}],
    "anthropic_version": "bedrock-2023-05-31"
})

personalized_email_response = bedrock_client.invoke_model(
    body = request_body,
    modelId = identifier_of_the_model
)</code></pre> 
</div> 
<p>Amazon Bedrock returns a personalized email for the user:</p> 
<blockquote>
 <p>Subject: Your Weekend Movie Escape Awaits! Three Laugh-Out-Loud Comedies Coming Next Week</p> 
 <p>Hi there,</p> 
 <p>Need a break from reality? We’ve got you covered with three fantastic comedies hitting our streaming platform next week!</p> 
 <p>## This Week’s Spotlight: Comedy Gems That Will Make Your Day</p> 
 <p>**There’s Something About Mary**<br /> This hilarious romantic comedy follows a man who finally gets a second chance with his high school dream girl—after their first date went hilariously wrong. With unforgettable laughs and heartwarming moments, it’s the perfect weekend watch!</p> 
 <p>**Shakespeare in Love**<br /> When the greatest playwright of all time faces writer’s block and money troubles, an unexpected romance changes everything! This award-winning comedy-drama blends history, romance, and witty humor as Shakespeare finds his muse and creates one of his most beloved plays. A delightful escape for literature lovers and romantics alike!</p> 
 <p>**The Birdcage**<br /> Prepare for non-stop laughter in this comedy classic! When a gay cabaret owner and his drag queen partner pretend to be straight to impress their future in-laws (who happen to be ultra-conservative), chaos and hilarity ensue. A perfect blend of humor and heart that still resonates today.</p> 
 <p>So grab your popcorn, get comfortable on the couch, and enjoy these comedy classics starting next week!</p> 
 <p>Happy streaming!</p> 
 <p>The Movies-On-Demand Team</p> 
 <p>P.S. Don’t forget to check out our complete catalog for more great films in every genre!</p>
</blockquote> 
<p>Although this is already a good outreach email because the recommendations are personalized to the user, we can personalize it further by adding more information about the user.</p> 
<h2>Generate personalized communication with recommended movies, user demographic information, and favorite genre</h2> 
<p>We will generate emails by assuming two different demographics for the users as well as their favorite genre.</p> 
<p>The version of the <code>ml-latest-small</code> dataset from the <a href="https://grouplens.org/datasets/movielens/" rel="noopener" target="_blank">MovieLens dataset</a> we used in this example doesn’t contain demographic data; therefore, we will try out multiple options. In a real-world scenario, you might know the demographics of your audience.</p> 
<p>To experiment, let’s use the following example demographic:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python"># Sample user demographics
user_demographic_1 = f'The user is a 50 year old adult called Otto.'</code></pre> 
</div> 
<p>We also add the user’s favorite genre to the prompt as follows:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">prompt_template =&nbsp;f'''You are a skilled publicist. Write a high-converting marketing email advertising several movies available in a video-on-demand streaming platform next week,
given the movie and user information below. Do not add additional information. Your email will leverage the power of storytelling and persuasive language.
You want the email to impress the user, so make it appealing to them based on the information contained in the &lt;user&gt; tags,
and take into account the user's favorite genre in the &lt;genre&gt; tags.
The movies to recommend and their information is contained in the &lt;movie&gt; tag.
All movies in the &lt;movie&gt; tag must be recommended. Give a summary of the movies and why the human should watch them.
Put the email between &lt;email&gt; tags.

&lt;user&gt;
{user_demographic}
&lt;/user&gt;

&lt;genre&gt;
{favorite_genre}
&lt;/genre&gt;

&lt;movie&gt;
{movie_list}
&lt;/movie&gt;

Assistant:

&lt;email&gt;
'''</code></pre> 
</div> 
<p>After adding the information, the new prompt is as follows:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">"You are a skilled publicist. Write a high-converting marketing email advertising several movies available in a video-on-demand streaming platform next week, given the movie and user information below. Do not add additional information. Your email will leverage the power of storytelling and persuasive language. You want the email to impress the user, so make it appealing to them based on the information contained in the &lt;user&gt; tags, and take into account the user's favorite genre in the &lt;genre&gt; tags. The movies to recommend and their information is contained in the &lt;movie&gt; tag. All movies in the &lt;movie&gt; tag must be recommended. Give a summary of the movies and why the human should watch them. Put the email between &lt;email&gt; tags.
\n
\n
&lt;user&gt;
\n
The user is a 50 year old adult called Otto.
\n
&lt;/user&gt;
\n
\n
&lt;genre&gt;
\n
Comedy
\n
&lt;/genre&gt;
\n
\n
&lt;movie&gt;
\n
[
{
'title': \"There's Something About Mary\",
'genres': 'Comedy and Romance',
'plot': 'A man gets a chance to meet up with his dream girl from high school, even though his date with her back then was a complete disaster.'
},
{
'title': 'Shakespeare in Love',
'genres': 'Comedy and Drama and History and Romance',
'plot': \"The world's greatest ever playwright, William Shakespeare, is young, out of ideas and short of cash, but meets his ideal woman and is inspired to write one of his most famous plays.\"
},
{
'title': 'The Birdcage',
'genres': 'Comedy',
'plot': \"A gay cabaret owner and his drag queen companion agree to put up a false straight front so that their son can introduce them to his fianc\u00e9e's right-wing moralistic parents.\"
}
]
\n
&lt;/movie&gt;
\n
\n
Assistant:
\n
&lt;email&gt;
\n &nbsp; &nbsp;"</code></pre> 
</div> 
<p>Amazon Bedrock returns a personalized email for the user:</p> 
<blockquote>
 <p>Subject: Otto, Get Ready for a Comedy Extravaganza on Your Screen Next Week!</p> 
 <p>Dear Otto,</p> 
 <p>We’re thrilled to bring you an exclusive lineup of comedy classics hitting our streaming platform next week! As someone who appreciates a good laugh, you’re in for a treat with these award-winning comedies that will brighten your evenings.</p> 
 <p>## “There’s Something About Mary”<br /> This hilarious romantic comedy follows the misadventures of a man who finally gets a second chance with his high school dream girl. After a first date that was nothing short of catastrophic, he’s determined to make things right years later. With its perfect blend of outrageous humor and heartwarming moments, this comedy classic delivers laughs that have stood the test of time.</p> 
 <p>## “Shakespeare in Love”<br /> Experience the witty and charming story of a young, broke William Shakespeare who finds his muse in the most unexpected place. This brilliant comedy-drama offers a fictional account of how the greatest playwright found inspiration through love. With its clever dialogue, historical setting, and romantic storyline, this Academy Award-winning film combines your love of comedy with rich storytelling that will keep you engaged from beginning to end.</p> 
 <p>## “The Birdcage”<br /> A comedy masterpiece that delivers non-stop laughs! When a gay cabaret owner and his flamboyant partner must pretend to be straight to impress their future in-laws (who happen to be ultra-conservative), chaos ensues. The brilliant performances and hilarious situations make this one of the most beloved comedies of its era. It’s the perfect film for when you need genuine belly laughs and brilliant comedic timing.</p> 
 <p>Otto, these comedies are among the best in their genre and will be available for your enjoyment starting next week. Whether you’re in the mood for slapstick humor, clever wit, or situational comedy, this collection has something perfect for your evening entertainment.</p> 
 <p>Grab your favorite snack, get comfortable on the couch, and prepare for an unforgettable comedy marathon!</p> 
 <p>Happy streaming!</p> 
 <p>The VOD Team</p>
</blockquote> 
<p>The email now contains information about the user’s favorite genre and is personalized to the user using their name and the recommended the movies the user is most likely to be interested in.</p> 
<h2>Clean up</h2> 
<p>Make sure you clean up any unused resources you created in your account while following the steps outlined in this post. You can delete filters, recommenders, datasets, and dataset groups using the <a href="http://aws.amazon.com/console" rel="noopener" target="_blank">AWS Management Console</a> or the Python SDK.</p> 
<h2>Conclusion</h2> 
<p>Traditional AI and generative AI allow you to build hyper-personalized experiences for your users. In this post, we showed how to generate personalized outbound communication by getting personalized recommendations for each user using Amazon Personalize and then using user preferences and demographic information to write a personalized email communication using Amazon Bedrock. By using AWS managed services, such as Amazon Personalize and Amazon Bedrock, you can create this content with only a few API calls—no ML experience required.</p> 
<p>For more information about Amazon Personalize, see the <a href="https://docs.aws.amazon.com/personalize/latest/dg/what-is-personalize.html" rel="noopener" target="_blank">Amazon Personalize Developer Guide</a>. For more information on working with generative AI on AWS, see <a href="https://aws.amazon.com/blogs/machine-learning/announcing-new-tools-for-building-with-generative-ai-on-aws/" rel="noopener" target="_blank">Announcing New Tools for Building with Generative AI on AWS</a>.</p> 
<hr /> 
<h3>About the Author</h3> 
<p style="text-align: left;"><img alt="" class="size-full wp-image-100890 alignleft" height="147" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/03/04/gruebler040325.png" width="100" /><strong>Anna Grüebler Clark</strong> is a Specialist Solutions Architect at AWS focusing on in Artificial Intelligence. She has more than 16 years experience helping customers develop and deploy machine learning applications. Her passion is taking new technologies and putting them in the hands of everyone, and solving difficult problems leveraging the advantages of using traditional and generative AI in the cloud.</p>
