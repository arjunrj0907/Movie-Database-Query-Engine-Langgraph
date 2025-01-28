# Movie Database Query Engine with LangGraph

This repository contains a Flask application that uses LangGraph, Groq's Gemma2-9b-It LLM, and a Neo4j database to answer movie-related questions. It also uses Tavily search for out-of-context queries.

## Features

* **Natural Language to Cypher:** Converts natural language questions into Cypher queries for a Neo4j movie database.
* **Query Validation and Correction:** Validates and corrects generated Cypher queries using the LLM.
* **Database Execution:** Executes valid Cypher queries against the Neo4j database.
* **Response Generation:** Generates natural language responses based on the database results.
* **Out-of-Context Handling:** Uses Tavily search to answer questions outside the movie database context.
* **Guardrails:** Determines if the question is related to movies or not.
* **Gradio Interface:** Provides a user-friendly Gradio interface for interacting with the application.
* **LangGraph workflow:** Uses LangGraph to create a stateful, cyclical, directed graph application.

## Prerequisites

* Python 3.7+
* Neo4j database
* Groq API key
* Tavily API key
* Environment variables configured

## Installation

1.  Clone the repository:

    ```bash
    git clone [repository_url]
    cd [repository_directory]
    ```

2.  Create a virtual environment (recommended):

    ```bash
    python -m venv venv
    source venv/bin/activate  # On macOS and Linux
    venv\Scripts\activate  # On Windows
    ```

3.  Install the required packages:

    ```bash
    pip install -r requirements.txt
    ```

4.  Set up environment variables:

    * `GROQ_API_KEY`: Your Groq API key.
    * `NEO4J_URI`: Your Neo4j database URI.
    * `NEO4J_USERNAME`: Your Neo4j username.
    * `NEO4J_PASSWORD`: Your Neo4j password.
    * `TAVILY_API_KEY`: Your Tavily API key.
    * Create a `.env` file in the root directory and add these variables.

5.  Load the movie dataset into Neo4j:

    * Run the provided Cypher query in the Neo4j browser or using the Neo4j driver.

    ```cypher
    LOAD CSV WITH HEADERS FROM
    '[https://raw.githubusercontent.com/tomasonjo/blog-datasets/main/movies/movies_small.csv](https://raw.githubusercontent.com/tomasonjo/blog-datasets/main/movies/movies_small.csv)' as row

    MERGE(m:Movie{id:row.movieId})
    SET m.released = date(row.released),
        m.title = row.title,
        m.imdbRating = toFloat(row.imdbRating)
    FOREACH (director in split(row.director, '|') |
        MERGE (p:Person {name:trim(director)})
        MERGE (p)-[:DIRECTED]->(m))
    FOREACH (actor in split(row.actors, '|') |
        MERGE (p:Person {name:trim(actor)})
        MERGE (p)-[:ACTED_IN]->(m))
    FOREACH (genre in split(row.genres, '|') |
        MERGE (g:Genre {name:trim(genre)})
        MERGE (m)-[:IN_GENRE]->(g))
    ```

## Usage

1.  Run the Flask application:

    ```bash
    python app.py
    ```

2.  Access the Gradio interface in your web browser at the provided URL (usually `http://0.0.0.0:7860`).

3.  Enter your movie-related question in the text box and click "Submit."

## Code Description

* `app.py`: Contains the Flask application and Gradio interface.
* `requirements.txt`: Lists the required Python packages.
* `.env`: Stores environment variables.

## LangGraph Workflow

The LangGraph workflow consists of the following nodes:

1.  **Guardrails:** Determines if the question is related to movies.
2.  **Generate Cypher:** Generates a Cypher query from the natural language question.
3.  **Tavily Search Node:** Searches the web for out-of-context queries.
4.  **Validate Cypher:** Validates the generated Cypher query.
5.  **Correct Cypher:** Corrects the Cypher query if validation fails.
6.  **Execute Cypher:** Executes the valid Cypher query against the Neo4j database.
7.  **Generate Final Answer:** Generates a natural language response based on the database results.

## Dependencies

* Flask
* Gradio
* LangChain
* LangGraph
* LangChain-Neo4j
* LangChain-Groq
* Pydantic
* Requests
* python-dotenv
* langchain-huggingface
* chromadb
* langchain-community

## Additional Resources

* LangGraph documentation: [https://langchain-ai.github.io/langgraph/](https://langchain-ai.github.io/langgraph/)

## Example questions:

* "Who directed Inception?"
* "List all movies starring Tom Hanks."
* "What are the genres of the movie 'The Matrix'?"
* "What is the weather today?" (This will use Tavily search)