
# RAG-based chatbot using Langchain and MongoDB Atlas
This starter template implements a Retrieval-Augmented Generation (RAG) chatbot using LangChain and MongoDB Atlas. RAG combines AI language generation with knowledge retrieval for more informative responses. LangChain simplifies building the chatbot logic, while MongoDB Atlas' Vector database capability provides a powerful platform for storing and searching the knowledge base that fuels the chatbot's responses.

## Setup 
Follow the steps below to set up a RAG chatbot, that can create a knowledge base using the data from PDF files you provide.


### Prerequisites

Before you begin, make sure you have the following ready:

- **MongoDB Atlas URI**: Set up your account if you don't already have one ([Create Account](https://www.mongodb.com/docs/guides/atlas/account/)). Then create an Atlas cluster.

  _**NOTE** : Make sure to allow access from your new RAG service (once it's hosted) to your MongoDB instance using the [ip-access-list](https://www.mongodb.com/docs/atlas/security/ip-access-list/) in MongoDB Atlas_.
    
- **OpenAI API Key** Set up an OpenAI account. [Then retrieve your API keys here](https://platform.openai.com/api-keys).

- **A PDF of your choice**. This PDF represents your knowledge base. (Here's an [example PDF](https://drive.google.com/file/d/1yIHmqe5-D_32tlSN1LZq1LJY8TStziXx/view?usp=drive_link) if you need one.)

### Option 1 (Recommended)

#### Step 1: Configure Render Web Service

- Fork [mongodb-partners/MongoDB-RAG-Render](https://github.com/mongodb-partners/MongoDB-RAG-Render/) on GitHub.
  
- Create a [new **Web Service** on Render](https://docs.render.com/web-services#deploy-your-own-code). Choose "Build and deploy from a Git repository" and select your forked GitHub repo.

- Use the following values during Web Service creation:

  ```
  Runtime/Language    Node
  Build Command       npm install; npm run build
  Start Command       npm run start
  ```

- Populate the values of the **Environment Variables** as follows:

  ````
  OPENAI_API_KEY = <YOUR_OPENAI_KEY>           # API Key copied from the OpenAI portal
  MONGODB_URI = <YOUR_MONGODB_URI>             # Connection URI to MongoDB Instance
  ````


#### Step 2: Deploy Render Web Service
- Once you have inputted the above values, create the Web Service.
- Wait for the Web Service to be deployed and start serving traffic.


#### Step 3: Upload PDF files to MongoDB Atlas
- Head to the `Train` tab on the MongoDB website and upload a PDF document of your choice. 

- If everything is deployed correctly, your document should start uploading to your cluster under the `chatter > training_data` collection.

- Your data should now start appearing as below in the collection.

  ![image](https://github.com/utsavMongoDB/MongoDB-RAG-NextJS/assets/114057324/316af753-8f7b-492f-b51a-c23c109a3fac)



#### Step 4: Create Vector Index on Atlas
- Now for the RAG (QnA) to work, you need to create a Vector Search Index on Atlas so the vector data can be fetched and served to LLMs.

  #### Create a search index as below.

-  Let’s head over to our MongoDB Atlas user interface to create our Vector Search Index. First, click on the “Search” tab and then on “Create Search Index.” You’ll be taken to this page (shown below). Please click on “JSON Editor.”
    ![image](https://github.com/utsavMongoDB/MongoDB-RAG-NextJS/assets/114057324/b41a09a8-9875-4e5d-9549-e62652389d33)

- Next, input the values shown in the image below and create the Vector.

  ````
    {
      ields": [
        {
          "type": "vector",
          "path": "text_embedding",
          "numDimensions": 1536,
          "similarity": "cosine",
        }
      ]
    }
  ````

  ![image](https://github.com/utsavMongoDB/MongoDB-RAG-NextJS/assets/114057324/d7e560b3-695c-4210-8a6d-ea50c589bc70)

- You should start seeing a vector index getting created. You should get an email once index creation is completed.
  ![image](https://github.com/utsavMongoDB/MongoDB-RAG-NextJS/assets/114057324/c1842069-4080-4251-8269-08d9398e09aa)


#### Step 5: Ask questions
- Once completed, head to the QnA section to start asking questions based on your trained data, and you should get the desired response.

  ![image](https://github.com/utsavMongoDB/MongoDB-RAG-NextJS/assets/114057324/c76c8c19-e18a-46b1-834a-9a6bda7fec99)




### Option 2
You can also use Render Blueprint to build and deploy the application. Go to the deployment page using the below button and follow the steps.

[![Deploy with Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/mongodb-partners/MongoDB-RAG-Render)



## Reference Architechture 

![image](https://github.com/utsavMongoDB/MongoDB-RAG-NextJS/assets/114057324/85ce551b-c6b2-43d6-bc4c-bc4df374142d)


This architecture depicts a Retrieval-Augmented Generation (RAG) chatbot system built with LangChain, OpenAI, and MongoDB Atlas Vector Search. Let's break down its key players:

- **PDF File**: This serves as the knowledge base, containing the information the chatbot draws from to answer questions. The RAG system extracts and processes this data to fuel the chatbot's responses.
- **Text Chunks**: These are meticulously crafted segments extracted from the PDF. By dividing the document into smaller, targeted pieces, the system can efficiently search and retrieve the most relevant information for specific user queries.
- **LangChain**: This acts as the central control unit, coordinating the flow of information between the chatbot and the other components. It preprocesses user queries, selects the most appropriate text chunks based on relevance, and feeds them to OpenAI for response generation.
- **Query Prompt**: This signifies the user's question or input that the chatbot needs to respond to.
- **Actor**: This component acts as the trigger, initiating the retrieval and generation process based on the user query. It instructs LangChain and OpenAI to work together to retrieve relevant information and formulate a response.
- **OpenAI Embeddings**: OpenAI, a powerful large language model (LLM), takes centre stage in response generation. By processing the retrieved text chunks (potentially converted into numerical representations or embeddings), OpenAI crafts a response that aligns with the user's query and leverages the retrieved knowledge.
- **MongoDB Atlas Vector Store**: This specialized database is optimized for storing and searching vector embeddings. It efficiently retrieves the most relevant text chunks from the knowledge base based on the query prompt's embedding. These retrieved knowledge nuggets are then fed to OpenAI to inform its response generation.


This RAG-based architecture seamlessly integrates retrieval and generation. It retrieves the most relevant knowledge from the database and utilizes OpenAI's language processing capabilities to deliver informative and insightful answers to user queries.


## Implementation 

The below components are used to build up the bot, which can retrieve the required information from the vector store, feed it to the chain and stream responses to the client.

#### LLM Model 

        const model = new ChatOpenAI({
            temperature: 0.8,
            streaming: true,
            callbacks: [handlers],
        });


#### Vector Store

        const retriever = vectorStore().asRetriever({ 
            "searchType": "mmr", 
            "searchKwargs": { "fetchK": 10, "lambda": 0.25 } 
        })

#### Chain

       const conversationChain = ConversationalRetrievalQAChain.fromLLM(model, retriever, {
            memory: new BufferMemory({
              memoryKey: "chat_history",
            }),
          })
        conversationChain.invoke({
            "question": question
        })
