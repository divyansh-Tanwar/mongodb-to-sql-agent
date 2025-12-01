<h1>MongoDB-SQL Query Conversion Agent</h1>
<h2>Problem</h2>
A lot of the time, enterprise teams need to turn MongoDB searches into SQL for:
<ol>
  <li>Data Validation</li>
  <li>Data store migration from MongoDB to SQL</li>
  <li>Reverse engineering of legacy application logic</li>
  <li>Enabling the use of document-oriented queries in relational systems</li>
</ol>
However:
<ol>
  <li>The paradigms of MongoDB and SQL are essentially different.</li>
  <li>Interpretation is necessary for aggregations, filters, projections, and update procedures.</li>
  <li>Manual rewriting is slow, inconsistent, and error-prone</li>
</ol>
The entire translation issue is automatically resolved by this agent.
<h2>Solution</h2>
I used Gemini 2.5 Flash-Lite and Google ADK to create a two-stage AI pipeline:
<h3>Stage 1: MongoDB Analysis Agent (LLM Agent + Tools)</h3>
<ul>
  <li>The MongoDB query is parsed.</li>
  <li>Transforms it into a structured JSON plan.</li>
  <li>Retrieves table metadata by calling a schema lookup tool.</li>
  <li>keeps output as analysis_json in the session state.</li>
</ul>
<h3>Stage 2: SQL Converter Agent (LLM Agent)</h3>
<ul>
  <li>Reads analysis_json</li>
  <li>Creates proper SQL from operations, projections, filters, sorting, grouping, and limits.</li>
  <li>Produces the final SQL query.</li>
</ul>
<h3>Key Capabilities</h3>
<ul>
  <li>Sequential agents for multi-step reasoning</li>
  <li>SQL creation with schema awareness using custom tools</li>
  <li>InMemorySessionService is used to preserve context.</li>
  <li>Find, insert, update, delete, and aggregate pipelines are among the supported operations.</li>
  <li>Included the Evaluation Framework</li>
  <li>Comparison based on rules</li>
</ul>
<h2>Architecture</h2>
<div align="center">                 ┌────────────────────────┐</div>
<div align="center">                    User / Workflow      </div>
<div align="center">                  (Notebook / API Call)  </div>
<div align="center">                 └───────────┬────────────┘</div>
<div align="center">                              Mongo Query</div>
<div align="center">                             ▼</div>
<div align="center">             ┌────────────────────────────────┐</div>
<div align="center">               Sequential Agent Pipeline     </div>
<div align="center">              (Google ADK)                   </div>
<div align="center">             └────────────────────────────────┘</div>
<div align="center">                        Executes in Order</div>
<div align="center">                       ▼</div>
<div align="center">       ┌────────────────────────────┐</div>
<div align="center">        1. Analysis Agent (LLM)    </div>
<div align="center">           + Custom Schema Tool    </div>
<div align="center">       └────────────────────────────┘</div>
<div align="center">                    writes 'analysis_json'</div>
<div align="center">                   ▼</div>
<div align="center">           ┌────────────────────────────┐</div>
<div align="center">              Session/State (ADK)      </div>
<div align="center">           └────────────────────────────┘</div>
<div align="center">                    reads 'analysis_json'</div>
<div align="center">                   ▼</div>
<div align="center">       ┌────────────────────────────┐</div>
<div align="center">        2. Converter Agent (LLM)   </div>
<div align="center">       └────────────────────────────┘</div>
<div align="center">                    SQL Output</div>
<div align="center">                   ▼</div>
<div align="center">            ┌──────────────────────────────┐</div>
<div align="center">               Evaluation Engine           </div>
<div align="center">             (Rule-based + LLM-as-Judge)   </div>
<div align="center">            └──────────────────────────────┘</div>

<h2>Component</h2>
<table>
  <tr>
    <th>Component Name</th>
    <th>Description</th>
  </tr>
  <tbody>
    <tr>
      <td>analysis_agent</td>
      <td>Generates a normalized JSON "analysis plan" after parsing a raw MongoDB query and calling the schema tool.</td>
    </tr>
    <tr>
      <td>converter_agent</td>
      <td>Reads analysis_json from state and generates equivalent ANSI SQL (SELECT/INSERT/UPDATE/DELETE).</td>
    </tr>
    <tr>
      <td>mongo_to_sql_pipeline</td>
      <td>Orchestrates the multi-step flow: Analysis → Conversion, passing state between sub-agents.</td>
    </tr>
    <tr>
      <td>get_collection_schema</td>
      <td>Python function tool that returns schema metadata for a given collection (columns, types, table).</td>
    </tr>
    <tr>
      <td>InMemorySessionService</td>
      <td>Stores per-session state, including analysis_json and sql_query between agent steps.</td>
    </tr>
    <tr>
      <td>convert_mongo_to_sql</td>
      <td>Simple Python wrapper that sends a Mongo query through the pipeline and returns final SQL Query.</td>
    </tr>
    <tr>
      <td>evaluate_agent</td>
      <td>Checks the accuracy of normalized SQL strings by running the agent on TEST_CASES.</td>
    </tr>
  </tbody>
</table>
<h3>Essential Tools</h3>
<table>
  <tr>
    <th>Technology</th>
    <th>Purpose</th>
  </tr>
  <tbody>
    <tr>
      <td>Google ADK</td>
      <td>Build structured agents, pipelines, and stateful runners.</td>
    </tr>
    <tr>
      <td>Gemini 2.5 Flash-Lite</td>
      <td>Fast, lightweight LLM for conversion logic.</td>
    </tr>
    <tr>
      <td>Python</td>
      <td>Evaluation, orchestration, and test harnessing.</td>
    </tr>
  </tbody>
</table>
<h2>Setup Instructions</h2>
<h3>1. Install dependencies</h3>
pip install google-adk google-genai
<h3>2. Setup API Key</h3>
<p>import os</p>
<p>os.environ["GOOGLE_API_KEY"] = "YOUR_API_KEY"</p>

<h2>Value Statement</h2>
This project produces an effective AI bot that can:
<ol>
  <li>Understanding MongoDB semantics based on documents</li>
  <li>Transforming them into relational SQL</li>
  <li>Making use of multi-step thinking, tools, and schema</li>
  <li>Generating accurate SQL appropriate for BI/ETL tasks</li>
</ol>
