import os
import time
import langchain
from langchain.agents import create_agent
from langchain_groq import ChatGroq
from langchain_google_genai import ChatGoogleGenerativeAI
from tavily import TavilyClient
import pytesseract as pyt
import numpy as np
import streamlit as st

#================API-KEYS==================
GOOGLE_KEY=st.sidebar.text_input("Google -API",type="password")
GROQ_KEY=st.sidebar.text_input("Groq-API",type="password")
TAVILY_KEY=st.sidebar.text_input("Tavily -API",type="password")

os.environ["GOOGLE_API_KEY"]=GOOGLE_KEY
os.environ["GROQ_API_KEY"]=GROQ_KEY
os.environ["TAVILY_API_KEY"]=TAVILY_KEY
ALL_API=[GOOGLE_KEY,GROQ_KEY,TAVILY_KEY]
if not all(ALL_API):
  st.sidebar.info("Pass API keys")
elif:
  st.sidebar.info("Must Pass all API keys")
else:
  st.sidebar.success("API keys loades successfully")
# Step1:model call
  model=ChatGoogleGenerativeAI(
    model="gemini-3.5-flash-lite",
    google_api_key=GOOGLE_API_KEY
  )
#================Frontend=================
st.title("AI-Agent-Powered PPT Generator")
user_query=st.text_area("Write your PPT topic or prompt:")

#===================Assests=================
# tool 1
def search_latest_info(query):
  """This function search latest news or content from website using tavily,helpful to check trending content"""
  client= TavilyClient(api_key=TAVILY_API_KEY)
  response=client.search(query)
  return response
  
# tool_2
def generate_image(img_prompt):
  """This function ,helps to generate image using free api, with given img_prompt using pllinations"""
  url=f"https://image.pollinations.ai/{img_prompt}"
  # file handling
  import requests as r
  content =r.get(url).content
  with open(f"Image.jpeg",'wb') as f:
    f.write(content)
  from PIL import Image
  return Image.open("Image.jpeg")
  
# with tabs
tab1,tab2,tab3,=st.tabs(["Generate Image",
                         "Check latest news",
                         "Generate PPT"])
# detailed prompt generator
def prompt_generate(model,query):
  prompt=f"""Your task is to give detailed prompt instructions for given.
  prompt:
  You are a Professional PPT generatos, where user will give the query and based on that,
  you have to generate dynamic,HTML output based ppt with advanced CSS and Dynamic UI and UX with PPT toggle
  button , Based on query take image reference to generate and embed the same in ppt,using
  Image ref:url=https://images.unsplash.com/photo,
  or url= https://image.pollinations.ai/, 
  make sure img src must be valid, and image must be
  present inside html, Generate
  with image caption, and no markdowns
  user query given below:{query}"""
  response =model.invoke(prompt)
  final_prompt=response.content[-1]['text']
  with open("ppt_prompt.txt",'w') as f:
    f.write(final_prompt)
  return final_prompt

agent=create_agent(
   model=model,
   tools=[search_latest_info,generate_image]
)  

#=================Display Agent==============
st.sidebar.image(agent)

#============With Tabs===============
with tab1:
  st.header("Generate Image give prompt")
  if st.button("Click to generate:"):
    with st.spinner("Running Agent.."):
      data=generate_image(user_query)
      st.image(data)
      st.image("Image.jpeg")
with tab2:
  st.header("Check latest news")
  if st.button("Fetch news:"):
    with st.spinner("Running Agent.."):
      prompt =""" Give latest news India or world news related to tech,business,jobs, or user request
      output in proper HTML news templates"""+user_query
      response = agent.invoke({'messages':[{'role':"user",
                                     "content":final_prompt}]})
      code = response['messages'][-1].content[-1]['text']
      st.html(code,width="stretch",unsafe_allow_javascript=True)
with tab3:
  st.header("Create PPT")
  if st.button("Click to generate:"):
    with st.spinner("Running Agent.."):
      final_prompt=prompt_generate(model,user_query)
      response = agent.invoke({'messages':[{'role':"user","content":final_prompt}]})
      code = response['messages'][-1].content[-1]['text']
      st.html(code,width="stretch",unsafe_allow_javascript=True)
      st.download_button(label="DOWNLOAD PPT",data=code,file_name="ppt.html",mine='text/html')
      st.success("PPT Download Successfilly!!")
