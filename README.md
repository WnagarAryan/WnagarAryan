<div align="center">

<img src="https://raw.githubusercontent.com/WnagarAryan/WnagarAryan/main/assets/wordmark.svg" alt="Aryan Nagar - ML and Agentic AI Engineer" width="100%">

</div>

<div align="center">

I build explainable ML and agentic AI systems. Models that can justify the call
they just made, and agents that carry a task from prompt to finished work.

</div>

<div align="center">

<img src="https://img.shields.io/badge/Python-08090b?style=flat-square&logo=python&logoColor=00c9d7" alt="Python">
<img src="https://img.shields.io/badge/LangChain-08090b?style=flat-square&logo=langchain&logoColor=00c9d7" alt="LangChain">
<img src="https://img.shields.io/badge/FastAPI-08090b?style=flat-square&logo=fastapi&logoColor=00c9d7" alt="FastAPI">
<img src="https://img.shields.io/badge/Pandas-08090b?style=flat-square&logo=pandas&logoColor=00c9d7" alt="Pandas">
<img src="https://img.shields.io/badge/scikit--learn-08090b?style=flat-square&logo=scikitlearn&logoColor=00c9d7" alt="scikit-learn">
<img src="https://img.shields.io/badge/SHAP-08090b?style=flat-square&logoColor=00c9d7" alt="SHAP">

</div>

<div align="center">

<a href="https://portfolio-pied-two-57.vercel.app"><img height="38" src="https://raw.githubusercontent.com/WnagarAryan/WnagarAryan/main/assets/btn-demo.svg" alt="Portfolio"></a>
<a href="https://www.linkedin.com/in/aryan-nagar-aa870a290"><img height="38" src="https://raw.githubusercontent.com/WnagarAryan/WnagarAryan/main/assets/btn-linkedin.svg" alt="LinkedIn"></a>
<a href="mailto:nagar.aaryan04@gmail.com"><img height="38" src="https://raw.githubusercontent.com/WnagarAryan/WnagarAryan/main/assets/btn-gmail.svg" alt="Email"></a>

Open to ML, data science and agentic AI internships.

</div>

<img src="https://raw.githubusercontent.com/WnagarAryan/WnagarAryan/main/assets/divider.svg" width="100%" alt="">

## FraudLens

Explainable ML for job-scam detection. 2026.

Trained on 17,880 postings. Five algorithms benchmarked with SMOTE, and
LinearSVC selected on Fake-class F1 rather than accuracy, because a false
positive costs a real person a real opportunity. SHAP explains each call. A
LangChain and Groq layer turns that attribution into a readable sentence. Live
company verification runs against OpenCorporates and Tavily before the verdict
is shown.

Result: a 34.2% fraud rate in flagged listings against a 4.8% base rate. A 7x
lift.

```mermaid
flowchart LR
    A["17,880 postings"] --> B["TF-IDF + SMOTE<br/>5 algorithms benchmarked"]
    B --> C["LinearSVC<br/>selected on Fake-class F1"]
    C --> D["SHAP attribution"]
    C --> E["OpenCorporates + Tavily<br/>company verification"]
    D --> F["LangChain + Groq<br/>plain-English reason"]
    E --> F
    F --> G["34.2% vs 4.8%<br/>7x lift"]

    style A fill:#101114,color:#dde5f0,stroke:#22232a
    style B fill:#101114,color:#f4f7fb,stroke:#22232a
    style C fill:#0e7490,color:#f4f7fb,stroke:#00c9d7
    style D fill:#101114,color:#00c9d7,stroke:#00c9d7
    style E fill:#101114,color:#f4f7fb,stroke:#22232a
    style F fill:#0e7490,color:#f4f7fb,stroke:#00c9d7
    style G fill:#101114,color:#00c9d7,stroke:#00c9d7
```

<a href="https://fraudlens-project-showcase.onrender.com"><img height="38" src="https://raw.githubusercontent.com/WnagarAryan/WnagarAryan/main/assets/btn-demo.svg" alt="Live demo"></a>
<a href="https://github.com/WnagarAryan/FraudLens-Project-Showcase"><img height="38" src="https://raw.githubusercontent.com/WnagarAryan/WnagarAryan/main/assets/btn-code.svg" alt="Code"></a>

<img src="https://raw.githubusercontent.com/WnagarAryan/WnagarAryan/main/assets/divider.svg" width="100%" alt="">

## Kuberis

AI business-intelligence agent. 2026.

Upload a spreadsheet, get KPIs, then ask questions in English. A Pandas pipeline
classifies any CSV or Excel file and computes 10 or more KPIs without being told
the schema. The insights engine runs on LangChain and Groq's Llama 3.3 70B and
handles 8 or more categories of natural-language query.

Pandas calculates every figure. The model only describes the result, so a
business number is never something the model invented. FastAPI, custom JS
frontend, deployed on Render.
