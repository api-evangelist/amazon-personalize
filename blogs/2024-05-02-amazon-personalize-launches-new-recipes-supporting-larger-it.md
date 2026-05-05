---
title: "Amazon Personalize launches new recipes supporting larger item catalogs with lower latency"
url: "https://aws.amazon.com/blogs/machine-learning/amazon-personalize-launches-new-recipes-supporting-larger-item-catalogs-with-lower-latency/"
date: "Thu, 02 May 2024 15:58:25 +0000"
author: "Jingwen Hu"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-personalize/feed/"
---
<p>Personalized customer experiences are essential for engaging today’s users. However, delivering truly personalized experiences that adapt to changes in user behavior can be both challenging and time-consuming. <a href="https://aws.amazon.com/personalize/" rel="noopener" target="_blank">Amazon Personalize</a> makes it straightforward to personalize your website, app, emails, and more, using the same machine learning (ML) technology used by Amazon, without requiring ML expertise. With the <a href="https://docs.aws.amazon.com/personalize/latest/dg/working-with-predefined-recipes.html" rel="noopener" target="_blank">recipes</a>—algorithms for specific uses cases—provided by Amazon Personalize, you can deliver a wide array of personalization, including product or content recommendations and personalized ranking.</p> 
<p>Today, we are excited to announce the general availability of two advanced recipes in Amazon Personalize, <strong>User-Personalization-v2</strong> and <strong>Personalized-Ranking-v2</strong> (v2 recipes), which are built on the cutting-edge <a href="https://aws.amazon.com/what-is/transformers-in-artificial-intelligence/" rel="noopener" target="_blank">Transformers</a> architecture to support larger item catalogs with lower latency.</p> 
<p>In this post, we summarize the new enhancements, and guide you through the process of training a model and providing recommendations for your users.</p> 
<h2>Benefits of new recipes</h2> 
<p>The new recipes offer enhancements in scalability, latency, model performance, and functionality.</p> 
<ul> 
 <li><strong>Enhanced scalability</strong> – The new recipes now support training with up to 5 million item catalogs and 3 billion interactions, empowering personalization for large catalogs and platforms with billions of usage events.</li> 
 <li><strong>Lower latency</strong> – The lower inference latency and faster training times for large datasets of these new recipes can reduce the delay for your end-users.</li> 
 <li><strong>Performance optimization</strong> – Amazon Personalize testing showed that v2 recipes improved recommendation accuracy by up to 9% and recommendation coverage by up to 1.8x compared to previous versions. A higher coverage means Amazon Personalize recommends more of your catalog.</li> 
 <li><strong>Return item metadata in inference responses</strong> – The new recipes enable item metadata by default without extra charge, allowing you to return metadata such as genres, descriptions, and availability in inference responses. This can help you enrich recommendations in your user interfaces without extra work. If you use Amazon Personalize with generative AI, you can also feed the metadata into prompts. Providing more context to large language models can help them gain a deeper understanding of product attributes to generate more relevant content.</li> 
 <li><strong>Highly automated operations</strong> – Our new recipes are designed to reduce your overhead for training and tuning the model. For example, Amazon Personalize simplifies training configuration and automatically selects the optimal settings for your custom models behind the scenes.</li> 
</ul> 
<h2>Solution overview</h2> 
<p>To use the <code>User-Personalization-v2</code> and <code>Personalized-Ranking-v2</code> recipes, you first need to set up Amazon Personalize resources. Create your dataset group, import your data, train a solution version, and deploy a campaign. For full instructions, see <a href="https://docs.aws.amazon.com/personalize/latest/dg/getting-started.html" rel="noopener" target="_blank">Getting started</a>.</p> 
<p>For this post, we follow the Amazon Personalize console approach to deploy a campaign. Alternatively, you can build the entire solution using the SDK approach. You can also get batch recommendations with an asynchronous batch flow. We use the <a href="https://grouplens.org/datasets/movielens/" rel="noopener" target="_blank">MovieLens public dataset</a> and User-Personalization-v2 recipe to show you the workflow.</p> 
<h2>Prepare the dataset</h2> 
<p>Complete the following steps to prepare your dataset:</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/data-prep-ds-group.html" rel="noopener" target="_blank">Create a dataset group</a>. Each dataset group can contain up to three datasets: users, items, and interactions, with the interactions dataset being mandatory for <code>User-Personalization-v2</code> and <code>Personalized-Ranking-v2</code>.</li> 
 <li>Create an interactions dataset using a <a href="https://docs.aws.amazon.com/personalize/latest/dg/data-prep-creating-datasets.html" rel="noopener" target="_blank">schema</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/personalize/latest/dg/bulk-data-import-step.html" rel="noopener" target="_blank">Import the interactions data</a> to Amazon Personalize from <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3).</li> 
</ol> 
<h2>Train a model</h2> 
<p>After the dataset import job is complete, you can analyze data before training. Amazon Personalize <strong>Data analysis</strong> shows you statistics about your data as well as actions you can take to meet training requirements and improve recommendations.</p> 
<p>Now you’re ready to train your model.</p> 
<ol> 
 <li>On the Amazon Personalize console, choose <strong>Dataset groups</strong> in the navigation pane.</li> 
 <li>Choose your dataset group.</li> 
 <li>Choose <strong>Create solutions</strong>.</li> 
 <li>For <strong>Solution name</strong>, enter your solution name.</li> 
 <li>For <strong>Solution type</strong>, select <strong>Item recommendation</strong>.</li> 
 <li>For <strong>Recipe</strong>, choose the new <code>aws-user-personalization-v2</code> recipe.<br /> <img alt="" class="alignnone size-full wp-image-75484" height="956" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/01/ML-16735-image001.png" style="margin: 10px 0px 10px 0px;" width="1428" /></li> 
 <li>In the <strong>Training</strong> <strong>configuration</strong> section, for <strong>Automatic training</strong>, select <strong>Turn on</strong> to maintain the effectiveness of your model by retraining it on a regular cadence.<br /> <img alt="" class="alignnone size-full wp-image-75486" height="714" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/01/ML-16735-image003.png" style="margin: 10px 0px 10px 0px;" width="1768" /></li> 
 <li>Under <strong>Hyperparameter configuration</strong>, select <strong>Apply recency bias</strong>. Recency bias determines whether the model should give more weight to the most recent item interactions data in your interactions dataset.<br /> <img alt="" class="alignnone size-full wp-image-75487" height="376" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/01/ML-16735-image005.png" style="margin: 10px 0px 10px 0px;" width="1752" /></li> 
 <li>Choose <strong>Create solution</strong>.</li> 
</ol> 
<p>If you turned on automatic training, Amazon Personalize will automatically create your first solution version. A solution version refers to a trained ML model. When a solution version is created for the solution, Amazon Personalize trains the model backing the solution version based on the recipe and training configuration. It can take up to 1 hour for the solution version creation to start.</p> 
<ol start="10"> 
 <li>Under <strong>Custom resources</strong> in the navigation pane, choose <strong>Campaigns</strong>.</li> 
 <li>Choose <strong>Create campaign</strong>.</li> 
</ol> 
<p>A campaign deploys a solution version (trained model) to generate real-time recommendations. Campaigns created with solutions trained on v2 recipes are automatically opted-in to include item metadata in recommendation results. You can choose metadata columns during an inference call.</p> 
<ol start="12"> 
 <li>Provide your campaign details and create your campaign.<br /> <img alt="" class="alignnone size-full wp-image-75488" height="1326" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/01/ML-16735-image007.png" style="margin: 10px 0px 10px 0px;" width="1604" /></li> 
</ol> 
<h2>Get recommendations</h2> 
<p>After you create or update your campaign, you can get a recommended list of items that users are more likely to interact with, sorted from highest to lowest.</p> 
<ol> 
 <li>Select the campaign and <strong>View details</strong>.</li> 
 <li>In the <strong>Test campaign results</strong> section, enter the User ID and choose <strong>Get recommendations</strong>.<br /> <img alt="" class="alignnone size-full wp-image-75489" height="629" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/01/ML-16735-image009.png" style="margin: 10px 0px 10px 0px;" width="1429" /></li> 
</ol> 
<p>The following table shows a recommendation result for a user that includes the recommended items, relevance score, and item metadata (Title and Genre).</p> 
<p><img alt="" class="alignnone wp-image-75514 size-full" height="254" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/02/ML-16735-image012-new.png" style="margin: 10px 0px 10px 0px;" width="657" /></p> 
<p>Your User-Personalization-v2 campaign is now ready to feed into your website or app and personalize the journey of each of your customers.</p> 
<h2>Clean up</h2> 
<p>Make sure you clean up any unused resources you created in your account while following the steps outlined in this post. You can delete campaigns, datasets, and dataset groups via the Amazon Personalize console or using the Python SDK.</p> 
<h2>Conclusion</h2> 
<p>The new Amazon Personalize <code>User-Personalization-v2</code> and <code>Personalized-Ranking-v2</code> recipes take personalization to the next level with support of larger item catalogs, reduced latency, and optimized performance. For more information about Amazon Personalize, see the <a href="https://docs.aws.amazon.com/personalize/latest/dg/what-is-personalize.html" rel="noopener" target="_blank">Amazon Personalize Developer Guide</a>.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-75491 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/01/ML-16735-image013.jpg" width="100" />Jingwen Hu</strong> is a Senior Technical Product Manager working with AWS AI/ML on the Amazon Personalize team. In her spare time, she enjoys traveling and exploring local food.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-75492 alignleft" height="129" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/01/ML-16735-image015.jpg" width="100" />Daniel Foley</strong> is a Senior Product Manager for Amazon Personalize. He is focused on building applications that leverage artificial intelligence to solve our customers’ largest challenges. Outside of work, Dan is an avid skier and hiker.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-75493 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/01/ML-16735-image017.jpg" width="100" />Pranesh Anubhav</strong> is a Senior Software Engineer for Amazon Personalize. He is passionate about designing machine learning systems to serve customers at scale. Outside of his work, he loves playing soccer and is an avid follower of Real Madrid.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-75494 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/01/ML-16735-image019.jpg" width="100" />Tianmin Liu</strong> is a senior software engineer working for Amazon personalize. He focuses on developing recommender systems at scale using various machine learning algorithms. In his spare time, he likes playing video games, watching sports, and playing the piano.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-75495 alignleft" height="120" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/01/ML-16735-image021.jpg" width="100" />Abhishek Mangal </strong>is a software engineer working for Amazon Personalize. He works on developing recommender systems at scale using various machine learning algorithms. In his spare time, he likes to watch anime and believes One Piece is the greatest piece of storytelling in recent history.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-75496 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/01/ML-16735-image023.jpg" width="100" />Yifei Ma</strong> is a Senior Applied Scientist at AWS AI Labs working on recommender systems. His research interests lie in active learning, generative models, time series analysis, and online decision-making. Outside of work, he is an aviation enthusiast.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-75497 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/05/01/ML-16735-image025.png" width="100" />Hao Ding</strong> is a Senior Applied Scientist at AWS AI Labs and is working on advancing the recommender system for Amazon Personalize. His research interests lie in recommendation foundation models, Bayesian deep learning, large language models, and their applications in recommendation.</p> 
<p style="clear: both;"><img alt="" class="alignleft size-full wp-image-67159" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2023/11/22/Rishabh.jpg" width="100" /><strong>Rishabh Agrawal</strong> is a Senior Software Engineer working on AI services at AWS. In his spare time, he enjoys hiking, traveling and reading.</p>
