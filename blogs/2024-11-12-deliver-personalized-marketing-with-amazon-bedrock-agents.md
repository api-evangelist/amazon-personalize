---
title: "Deliver personalized marketing with Amazon Bedrock Agents"
url: "https://aws.amazon.com/blogs/machine-learning/deliver-personalized-marketing-with-amazon-bedrock-agents/"
date: "Tue, 12 Nov 2024 16:17:51 +0000"
author: "Ray Wang"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-personalize/feed/"
---
<p>Creative content plays a crucial role in marketing, and personalized creative content in particular significantly boosts marketing performance. Generating personalized content can present a significant challenge for marketers because it requires considerable time and resources. This challenge stems from the need for multiple versions of creative content across various channels, such as paid media (ads) and owned media, including electronic direct mail (EDM), social media posts, app notifications, and SMS. Scaling this process can be challenging, especially for small and medium-sized businesses.</p> 
<p><a href="https://aws.amazon.com/generative-ai/" rel="noopener" target="_blank">Generative AI</a> now empowers marketers to efficiently create personalized content, even with limited resources. By using machine learning (ML) models, you can pinpoint customer preferences for specific merchandise and tailor your marketing campaigns accordingly. This enables the crafting of compelling promotional text and striking visuals that effectively resonate with each customer segment, thereby driving engagement and increasing sales. Using <a href="https://aws.amazon.com/bedrock/agents/">Amazon Bedrock Agents</a> to create your own marketing agent allows you to seamlessly accomplish list targeting and personalized material generation for specific marketing purposes.</p> 
<p>In this post, we demonstrate a solution using Amazon Bedrock Agents, <a href="https://aws.amazon.com/bedrock/knowledge-bases/">Amazon Bedrock Knowledge Bases</a>, <a href="https://aws.amazon.com/bedrock/developer-experience/" rel="noopener" target="_blank">Amazon Bedrock Developer Experience</a>, and <a href="https://aws.amazon.com/personalize/" rel="noopener" target="_blank">Amazon Personalize</a> that allow marketers to save time and deliver efficient personalized advertising using a generative AI enhanced solution. Our solution is a marketing agent that shows how Amazon Personalize can effectively segment target customers based on relevant characteristics and behaviors. Additionally, by using Amazon Bedrock Agents and foundation models (FMs), our tool generates personalized creative content specifically tailored to each purpose. It customizes the tone, creative style, and individual preferences according to each customer’s specific prompt, providing highly customized and effective marketing communications.</p> 
<h2>Marketing agent overview</h2> 
<p>In the following diagram, we show the components that power our marketing agent.</p> 
<p><img alt="" class="alignnone size-full wp-image-90465" height="256" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/10/24/Picture1-23.png" width="936" /></p> 
<p>The difference between an agent and a large language model (LLM) is that an agent comprises not only LLMs, but also includes planning skills, tool usage, and memory. This means that when you provide a natural language prompt, you receive user segment results along with creative content tailored to your specifications. For example, if you want to promote an oven through EDM, social media posts, or SMS, the marketing agent will use its tools to generate a customer list using a segmentation model trained on your data. Furthermore, it will generate creative content that uses your historical creative content as examples and incorporate detailed merchandise data from your database.</p> 
<p>The marketing agent solution includes three tools:</p> 
<ul> 
 <li><strong>Merchandise tool</strong> – Retrieve merchandise details from <a href="https://aws.amazon.com/dynamodb/" rel="noopener" target="_blank">Amazon DynamoDB</a> (item database) and deliver them to the creative content tool according to the customer’s prompt.</li> 
 <li><strong>User segment tool</strong> – Retrieve a list from <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) created by Amazon Personalize which is tailored to the merchandise plan for promotion. This process uses comprehensive user, merchandise (item), and interaction data.</li> 
 <li><strong>Creative content tool</strong> – Generate the personalized creative content using an LLM based on the augmented prompt. The augmented prompt is formed by retrieving creative assets data from Amazon Bedrock Knowledge Bases (historical creative content), the merchandise database from DynamoDB, and the user database from DynamoDB, based on the customer’s input prompt.</li> 
</ul> 
<p>This agent operates based on natural language prompts and your organization’s data. These managed agents serve as intelligent orchestrators, managing interactions between FMs, API integrations, user questions and instructions, and knowledge sources filled with your proprietary data. The agent skillfully coordinates and processes user inputs through various dynamic steps during its runtime.</p> 
<p>Amazon Bedrock is a fully managed service that offers a choice of high-performing FMs from leading AI companies like AI21 Labs, Anthropic, Cohere, Meta, Mistral AI, Stability AI, and Amazon through a single API, along with a broad set of capabilities to build generative AI applications with security, privacy, and responsible AI. The single API access, regardless of the models you choose, gives you the flexibility to use different FMs and upgrade to the latest model versions with minimal code changes.</p> 
<p>Amazon Bedrock agents plan and run multistep tasks using company systems and data sources—from answering customer questions about your product availability to taking their orders. With Amazon Bedrock, you can create an agent in just a few clicks by selecting an FM and providing it access to your enterprise systems, knowledge bases, and <a href="http://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a>&nbsp;functions to securely run your APIs. An agent analyzes the user request and automatically calls the necessary APIs and data sources to fulfill the request. Amazon Bedrock Agents enables you to do this securely and privately—you don’t have to engineer prompts, manage session context, or manually orchestrate tasks.</p> 
<p>Amazon Bedrock Knowledge Bases is a fully managed capability that helps you implement the entire retrieval augmented generation (RAG) workflow, from ingestion to retrieval and prompt augmentation, without having to build custom integrations to data sources or manage data flows. Session context management is built in, so your app can readily support multi-turn conversations. You can use the Retrieve API to fetch relevant results for a user query from knowledge bases. You can also add knowledge bases to Amazon Bedrock Agents to provide contextual information to agents. The information retrieved from the knowledge base is provided with citations to improve transparency and minimize hallucinations.</p> 
<p>Amazon Personalize is a fully managed ML service that uses your data to generate recommendations for your users and enables developers to quickly implement a customized personalization engine, without requiring ML expertise. It accelerates your digital transformation with ML, making it effortless to integrate personalized recommendations into existing websites, applications, email marketing systems, and more.</p> 
<h2>Solution overview</h2> 
<p>Amazon Bedrock Agents is our key component for developing our marketing agent. It enables you to build and configure autonomous agents in your application. Agents orchestrate interactions between FMs, data sources, software applications, and user conversations, and automatically call APIs to perform actions and invoke knowledge bases to supplement information for these actions. You can add actions for it to carry out and define how to handle them by writing Lambda functions in a programming language of your choice. For more details, refer to <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html" rel="noopener" target="_blank">Automate tasks in your application using conversational agents</a>.</p> 
<p>We implement the marketing agent through Amazon Bedrock Agents, and use the following key features:</p> 
<ul> 
 <li><strong>Foundation model</strong> – The agent invokes an FM to interpret user input, generate subsequent prompts in its orchestration process, and generate creative content based on the customer’s requirement.</li> 
 <li><strong>Instructions</strong> – Instructions tell the agent what it’s designed to do and how to do it.</li> 
 <li><strong>Action groups</strong> – Action groups are interfaces that an agent uses to interact with the different underlying components such as APIs (such as Amazon Personalize batch inference result on Amazon S3) and databases (such as user or merchandise databases). An agent uses action groups to carry out actions, such as making an API call to another tool.</li> 
 <li><strong>Knowledge base</strong> – The knowledge base is a link to an existing data source, consisting of the customer’s historical creative content, which allows the agent to query for extra context for the prompts.</li> 
</ul> 
<p>For details about supported models, refer to <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html" rel="noopener" target="_blank">Supported foundation models in Amazon Bedrock</a>, <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/agents-supported.html" rel="noopener" target="_blank">Supported regions and models for Amazon Bedrock Agents</a>, and <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-supported.html" rel="noopener" target="_blank">Supported regions and models for Amazon Bedrock Knowledge Bases</a>.</p> 
<p>The following diagram illustrates the solution workflow.</p> 
<p><img alt="" class="alignnone size-full wp-image-90466" height="188" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/10/24/Picture2-12.png" width="936" /></p> 
<p>There are two associated action groups:</p> 
<ul> 
 <li><strong>Segment targeted customer list</strong> – Useful for segmenting a customer list for specific merchandise that you aim to promote</li> 
 <li><strong>Generate personalized creative content</strong> – Useful for generating creative content tailored to specific purposes, such as diverse customer preferences, varying customer types, and different marketing channels</li> 
</ul> 
<p>We use two types of datasets in this solution:</p> 
<ul> 
 <li><strong>Structured customer data</strong> – We use customer data, merchandise (item) data, and interaction data to train the segmentation model using Amazon Personalize</li> 
 <li><strong>Unstructured data</strong> – We use historical creative content and merchandise (item) data as augmented prompts to make sure that the creative content generated by the LLM aligns with your brand’s style and marketing guidelines</li> 
</ul> 
<p>When the marketing agent receives a prompt from a business user, it follows a number of steps as part of its orchestration:</p> 
<ol> 
 <li>Outline the steps for the task by using an LLM within Amazon Bedrock according to the specifications provided in the prompt.</li> 
 <li>Follow chain-of-thought reasoning and instructions, and complete the steps using appropriate action groups. As part of the process, depending on the prompt, the agent will search and identify relevant context for RAG.</li> 
 <li>Pass the results with the prompt to an LLM within Amazon Bedrock.</li> 
 <li>Augment the prompt with the results of the tool execution or knowledge base search and send it to the LLM.</li> 
</ol> 
<p>The following diagram illustrates the technical architecture and key steps.</p> 
<p><img alt="" class="alignnone size-full wp-image-90467" height="959" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/10/24/Picture3-14.png" width="1430" /></p> 
<p>Amazon Bedrock Agents allows you to set up the entire process, including getting the user segmentation list from Amazon Personalize and generating the personalized promotional content with Anthropic’s Claude 3 on Amazon Bedrock. There are three steps: data preparation, agent development, and agent testing. You can find the sample code and the <a href="https://aws.amazon.com/cdk/" rel="noopener" target="_blank">AWS Cloud Development Kit</a> (AWS CDK) stack in the <a href="https://github.com/aws-samples/amazon-bedrock-samples/tree/main/agents-and-function-calling/bedrock-agents/use-case-examples/marketing-agent" rel="noopener" target="_blank">GitHub repo</a>.</p> 
<h2>Prepare the data</h2> 
<p>Complete the following steps to prepare your data:</p> 
<ol> 
 <li>Store your creative content on Amazon S3. Ingest your data by generating embeddings with an FM and storing them in a supported vector store like <a href="https://aws.amazon.com/opensearch-service/" rel="noopener" target="_blank">Amazon OpenSearch Service</a>.</li> 
 <li>Use Amazon Bedrock Knowledge Bases by specifying an S3 bucket that contains your exported creative content data. For instructions, refer to <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html" rel="noopener" target="_blank">Retrieve data and generate AI responses with knowledge bases</a>. 
  <ol> 
   <li>Use OpenSearch Service as the vector database.</li> 
   <li>Complete the knowledge base configuration and&nbsp;<a href="https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-ingest.html" rel="noopener" target="_blank">synchronize data</a> from Amazon S3 to OpenSearch Service so the vector database data remains up to date.</li> 
  </ol> </li> 
 <li>Initiate an Amazon Personalize job with the <a href="https://docs.aws.amazon.com/personalize/latest/dg/item-affinity-recipe.html" rel="noopener" target="_blank">USER_SEGMENTATION recipe</a> to create user segmentations and export the results to Amazon S3. For more information, see <a href="https://docs.aws.amazon.com/personalize/latest/dg/getting-user-segments.html" rel="noopener" target="_blank">Getting user segments</a>. 
  <ol> 
   <li>Upload your user dataset, interactions dataset, and item dataset into Amazon S3 for model training and <a href="https://docs.aws.amazon.com/personalize/latest/dg/creating-batch-seg-job.html" rel="noopener" target="_blank">create a batch segment job</a> to get your user segment list. This allows you to map item IDs to a list of users interested in these items.</li> 
   <li>The <a href="https://docs.aws.amazon.com/personalize/latest/dg/batch-segment-job-output-examples.html" rel="noopener" target="_blank">batch segment job output</a> will be a JSON file stored on Amazon S3 that looks like the following example:</li> 
  </ol> </li> 
</ol> 
<pre><code class="lang-json">{"input": {"itemId": "e1669081-8ffc-4dec-97a6-e9176d7f6651"}, "output": {"usersList": ["3184","4223","4301",...]}, "error": null}</code></pre> 
<h2>Build the agent</h2> 
<p>In this solution, you need a marketing agent, a creative content knowledge base, and three tools (the merchandise tool to get detailed merchandise information, the user segment tool to get the target audience list, and the creative content tool to generate the creative content by the LLM) tailored to automate the various tasks associated with delivering personalized creative content efficiently. Complete the following steps to build your agent:</p> 
<ol> 
 <li>Clone the repository to your local machine or AWS environment, set up a virtual environment and activate it, download the related data, and install the required Python packages using the following code:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">git clone https://github.com/aws-samples/amazon-bedrock-samples.git

cd ./amazon-bedrock-samples/agents-for-bedrock/use-case-examples/marketing-agent

python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
wget https://aws-blogs-artifacts-public.s3.amazonaws.com/artifacts/ML-16145/data.zip
unzip data.zip
wget https://code.retaildemostore.retail.aws.dev/images.tar.gz
mkdir -p data/image

tar xf images.tar.gz -C data/image</code></pre> 
</div> 
<ol start="2"> 
 <li>Deploy using the following code:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">cdk deploy</code></pre> 
</div> 
<ol start="3"> 
 <li>Locate the agent name in the output of the preceding command. In the following screenshot, for example, the Amazon Bedrock agent name is <code>bedrock-agent-markegent-2e1ece3c</code>.</li> 
</ol> 
<p><img alt="" class="alignnone size-full wp-image-90468" height="268" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/10/24/Picture4-10.png" width="1430" /></p> 
<p>To dive deeper into the solution and code for the complete steps, see the <a href="https://github.com/aws-samples/amazon-bedrock-samples/tree/main/agents-and-function-calling/bedrock-agents/use-case-examples/marketing-agent" rel="noopener" target="_blank">GitHub repo</a>.</p> 
<h2>Test the agent</h2> 
<p>You can test the agent on the Amazon Bedrock console. For instructions, see <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/agents-test.html" rel="noopener" target="_blank">Test and troubleshoot agent behavior</a>.</p> 
<p>The following screenshot shows an example of testing the agent.</p> 
<p><img alt="" class="alignnone wp-image-90872 size-full" height="466" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/10/28/scarfPicture5-8.png" width="400" /></p> 
<p>We used the following input message:</p> 
<p><code>I would like to write a social media post to promote the Sans pareil scarf (e1669081-8ffc-4dec-97a6-e9176d7f6651). Please assist me in crafting the marketing content.</code></p> 
<p>The following is the agent output:</p> 
<p><code>Elevate Your Style with Timeless Elegance ? Introducing the Sans Pareil Scarf - a true masterpiece of fashion designed for the sophisticated, style-conscious woman. Crafted with impeccable artistry from the finest materials, this $124.99 luxury accessory exudes an air of refinement that instantly elevates any outfit. Its rich colors and intricate patterns complement both casual and formal ensembles, adding an undeniable touch of elegance. Whether dressing for the office, a night out, or a special occasion, the Sans Pareil Scarf is the ultimate statement piece for the modern woman seeking timeless style. From the fashion-forward trendsetter to the classically chic professional, this exquisite scarf transcends age while remaining a treasured investment that will never go out of style. Indulge in enduring quality and sophistication with the Sans Pareil Scarf. Elevate your style and turn heads wherever you go. #LuxuryFashion #SansPareilScarf #TimelessElegance</code></p> 
<h2>Clean up</h2> 
<p>In this post, we demonstrated interactions using Lambda, Amazon Bedrock Agents, and Amazon Bedrock Knowledge Bases. To avoid incurring additional costs, delete these resources in the following order:</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/bedrock/latest/userguide/agents-delete.html" rel="noopener" target="_blank">Delete the Amazon Bedrock agent</a>.</li> 
 <li><a href="https://docs.aws.amazon.com/bedrock/latest/userguide/kb-delete.html" rel="noopener" target="_blank">Delete the Amazon Bedrock knowledge base</a> and its associated resources.</li> 
 <li>Delete the Lambda function and its related resources.</li> 
</ol> 
<h2>Summary</h2> 
<p>In this post, we discussed the use case of targeted marketing as an example to demonstrate the efficient delivery of personalized marketing creative content and target audience lists through a generative AI agent. The next step might involve developing a reinforcement learning-based agent to iterate on the performance of the agent.</p> 
<p>Our customer, Chunghwa Telecom, a leading telecom customer in Taiwan, followed this solution to <a href="https://www.youtube.com/watch?v=8AXVjy0IrDE" rel="noopener" target="_blank">implement generative AI enhanced marketing technology tool to enhance their business through Amazon Bedrock</a>. The marketing agent enabled CHT to initiate tailored campaigns promptly, leading to the realization of personalized marketing strategies and a 24-fold increase in their clickthrough rate.</p> 
<p>To use our marketing agent to enhance your marketing tasks, refer to the <a href="https://github.com/aws-samples/amazon-bedrock-samples/tree/main/agents-and-function-calling/bedrock-agents/use-case-examples/marketing-agent" rel="noopener" target="_blank">GitHub repo</a>.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><strong><img alt="" class="wp-image-90690 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/10/25/ray-wang.jpg" width="100" />Ray Wang</strong> is a Senior Solutions Architect at AWS. With 10 years of experience in the IT industry, Ray is dedicated to building modern solutions on the cloud, especially in NoSQL, big data, machine learning, and Generative AI. As a hungry go-getter, he passed all 12 AWS certificates to make his technical field not only deep but wide. He loves to read and watch sci-fi movies in his spare time.</p> 
<p style="clear: both;"><img alt="" class="wp-image-90691 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/10/25/weichil-1.jpg" width="100" /><strong>Paul Lu</strong> is a Senior Solution Architect at Amazon Web Services (AWS). He specialize in Serverless and modern application development, helping customers design high-performing, scalable cloud solutions. With extensive experience, he is passionate about driving innovation and delivering exceptional results.</p>
