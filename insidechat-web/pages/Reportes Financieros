import streamlit as st
from dotenv import load_dotenv
from PyPDF2 import PdfReader
from langchain.text_splitter import CharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings, HuggingFaceInstructEmbeddings
from langchain.vectorstores import FAISS
from langchain.chat_models import ChatOpenAI
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationalRetrievalChain
from htmlTemplates import css, bot_template, user_template
from langchain.llms import HuggingFaceHub

def get_pdf_text(pdf_docs):
    text = ""
    for pdf in pdf_docs:
        pdf_reader = PdfReader(pdf)
        for page in pdf_reader.pages:
            text += page.extract_text()
    return text


def get_text_chunks(text):
    text_splitter = CharacterTextSplitter(
        separator="\n",
        chunk_size=1000,
        chunk_overlap=200,
        length_function=len
    )
    chunks = text_splitter.split_text(text)
    return chunks


def get_vectorstore(text_chunks):
    embeddings = OpenAIEmbeddings()
    # embeddings = HuggingFaceInstructEmbeddings(model_name="hkunlp/instructor-xl")
    vectorstore = FAISS.from_texts(texts=text_chunks, embedding=embeddings)
    return vectorstore


def get_conversation_chain(vectorstore):
    llm = ChatOpenAI()
    conversation_chain = ConversationalRetrievalChain.from_llm(
        llm=llm,
        retriever=vectorstore.as_retriever(),
    )
    return conversation_chain

def handle_userinput(user_question):
    with st.spinner("Procesando..."):
        response = st.session_state.conversation({"question": user_question, "chat_history": st.session_state.chat_history})
    st.session_state.chat_history.append((user_question, response["answer"]))

    history = st.session_state.chat_history[::-1]
    for message in history:
        st.write(bot_template.replace("{{MSG}}", message[1]), unsafe_allow_html=True)
        st.write(user_template.replace("{{MSG}}", message[0]), unsafe_allow_html=True)
        
def main():
    load_dotenv()
    st.set_page_config(page_title="Consulta tus PDFs",
                       page_icon=":books:")
    st.write(css, unsafe_allow_html=True)

    tab1, tab2 = st.tabs([
    "Empezar Chat 🤖", 
    "Sube Tus Archivos 💾"]
    )

    if "conversation" not in st.session_state:
        st.session_state.conversation = None
    if "chat_history" not in st.session_state:
        st.session_state.chat_history = []  # Inicializa chat_history como una lista vacía
    
    with tab1:
        st.header("Consulta tus documentos PDFs :books:")
        user_question = st.text_input("Haz una pregunta sobre tus documentos:")
        if user_question:
            handle_userinput(user_question)


    with tab2:
        st.subheader("Tus documentos")
        pdf_docs = st.file_uploader(
            "Carga tus documentos PDFs aqui y dale click a 'Procesar'", 
            accept_multiple_files=True)

        # Estilos predeterminados
        st.markdown("""
            <style>
                .stButton button {
                    background-color: green;
                    color: white;
                    font-size: 30px;
                    padding: 15px 30px;
                }
                .stButton button:hover {
                    color: #e0e0e0; /* Gris claro */
                }
            </style>
        """, unsafe_allow_html=True)

        # Mostrar el botón solo si se han subido documentos PDF
        if pdf_docs:
            # Verificar que todos los archivos tengan la extensión .pdf
            pdf_files = [file for file in pdf_docs if file.name.lower().endswith('.pdf')]

            if pdf_files:
                st.write("¡Super! Ya tenemos tus archivos, ahora dale a 'Procesar' ⤵️ ")
                if st.button("Procesar", key="procesar_button", help="Haz clic para procesar"):
                    with st.spinner("Procesando"):
                        # get pdf text
                        raw_text = get_pdf_text(pdf_files)

                        # get the text chunks
                        text_chunks = get_text_chunks(raw_text)

                        # create vector store
                        vectorstore = get_vectorstore(text_chunks)

                        # create conversation chain
                        st.session_state.conversation = get_conversation_chain(
                            vectorstore)
            else:
                st.warning(" ⚠️ Por favor, sube solo archivos PDF 📋 ")
                
if __name__ == '__main__':
    main()

hide_st_style = """
            <style>
            #MainMenu {visibility: hidden;}
            footer {visibility: hidden;}
            header {visibility: hidden;}
            </style>
           """
st.markdown(hide_st_style, unsafe_allow_html=True)
