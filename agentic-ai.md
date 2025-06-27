'''
import streamlit as st
from langchain.agents import initialize_agent, load_tools
from langchain.chat_models import ChatOpenAI
from langchain.agents.agent_types import AgentType
from langchain.tools.python.tool import PythonREPLTool
import os
import pandas as pd

st.set_page_config(page_title="LangChain Agent Demo", layout="centered")
st.title("🧠 LangChain Agent ile Soru + Veri Analizi")

# Kullanıcıdan görev al
user_question = st.text_input("🔍 Ajanınıza bir görev verin:", "Bu tabloyu analiz et: Ortalama gelir nedir?")

# CSV yükleme
uploaded_file = st.file_uploader("📁 CSV Dosyası Yükleyin", type="csv")
dataframe = None
if uploaded_file:
    dataframe = uploaded_file.getvalue().decode("utf-8")
    st.success("✅ Dosya yüklendi")
    st.dataframe(pd.read_csv(uploaded_file))

if st.button("🚀 Ajanı Çalıştır"):
    try:
        os.environ["OPENAI_API_KEY"] = os.getenv("OPENAI_API_KEY")

        llm = ChatOpenAI(temperature=0, model_name="gpt-3.5-turbo")
        tools = load_tools(["serpapi"], llm=llm)

        # Python ortamı veri analizi için tanımlanır
        tools.append(PythonREPLTool())

        agent = initialize_agent(
            tools,
            llm,
            agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
            verbose=True
        )

        # Görev açıklamasına tabloyu da ekle
        if dataframe:
            prompt = f"Aşağıdaki veri çerçevesi ile çalış.\n\n{dataframe}\n\nGörev: {user_question}.\n\nSonucu sade, doğal Türkçe ile özetle."
        else:
            prompt = f"{user_question}. Lütfen sonucu sade, doğal Türkçe ile açıkla."

        response = agent.run(prompt)
        st.success("✅ Ajan görevi tamamladı:")
        st.write(response)

    except Exception as e:
        st.error("Ajan çalıştırılamadı. API anahtarı veya araç yapılandırmasını kontrol edin.")
        st.exception(e)

st.markdown("---")
st.caption("LangChain + LLM + Python + SerpAPI + CSV analizi + doğal dilde özetleme ajan")

'''


streamlit run agentic_demo.py
