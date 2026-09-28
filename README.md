# RAG com Llama 3.1, Ollama e LangChain

Aplicação simples de **Retrieval-Augmented Generation (RAG)** em Python. O programa carrega artigos da web, divide o conteúdo em trechos, cria embeddings locais com Ollama, recupera os trechos mais relevantes e usa o Llama 3.1 para responder à pergunta.

## Requisitos

- Windows
- Python 3.14 ou superior
- Ollama instalado e em execução
- Modelo `llama3.1` baixado
- Modelo de embeddings `nomic-embed-text` baixado
- Conexão com a internet para carregar os artigos

## Configuração do ambiente virtual

No PowerShell, dentro da pasta do projeto:

```powershell
.\.venv\Scripts\Activate.ps1
```

Confirme que o terminal mostra `(.venv)`. Também é possível executar sem ativar o ambiente:

```powershell
.\.venv\Scripts\python.exe app.py
```

No VS Code, selecione:

```text
C:\Users\nchni\Desktop\RAG\.venv\Scripts\python.exe
```

Use o comando `Python: Select Interpreter` para selecionar esse interpretador.

## Instalação das dependências

Com o `.venv` ativo:

```powershell
python -m pip install langchain langchain-community langchain-text-splitters langchain-ollama scikit-learn beautifulsoup4 tiktoken
```

O pacote `beautifulsoup4` é necessário pelo `WebBaseLoader`. O pacote `tiktoken` é usado por `RecursiveCharacterTextSplitter.from_tiktoken_encoder`.

## Configuração do Ollama

Instale o Ollama pelo site oficial:

https://ollama.com/

Depois baixe os dois modelos usados pelo programa:

```powershell
ollama pull llama3.1
ollama pull nomic-embed-text
```

Teste o modelo de resposta:

```powershell
ollama run llama3.1
```

O Ollama precisa estar disponível localmente quando o programa for executado.

## Execução

```powershell
.\.venv\Scripts\python.exe .\app.py
```

A aplicação carrega os três artigos definidos em `app.py`, cria o índice vetorial e executa a pergunta de exemplo:

```text
What is prompt engineering
```

## Como o código funciona

1. `WebBaseLoader` baixa os artigos da web.
2. `RecursiveCharacterTextSplitter` divide os documentos em trechos menores.
3. `OllamaEmbeddings` transforma cada trecho em um vetor usando `nomic-embed-text`.
4. `SKLearnVectorStore` armazena os vetores localmente.
5. O retriever busca os quatro trechos mais relacionados à pergunta.
6. `ChatOllama` usa o `llama3.1` para formular a resposta com base nos trechos recuperados.

## Problemas encontrados e correções

### 1. Python incorreto no terminal

O erro `No module named 'langchain_community'` aconteceu quando `python app.py` usou um Python diferente do `.venv`. O pacote estava instalado no ambiente virtual, mas não no interpretador global.

Use sempre:

```powershell
.\.venv\Scripts\python.exe .\app.py
```

ou ative o ambiente antes de usar `python`.

### 2. Import antigo de `RecursiveCharacterTextSplitter`

O tutorial usa:

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
```

Nas versões atuais, o import correto é:

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter
```

### 3. Import antigo de `PromptTemplate`

O caminho usado no tutorial pode não ser resolvido pelo Pylance nas versões atuais. O caminho utilizado neste projeto é:

```python
from langchain_core.prompts import PromptTemplate
```

### 4. URLs com sinais `< >`

O tutorial apresenta as URLs em formato parecido com Markdown:

```python
"<https://lilianweng.github.io/posts/2023-06-23-agent/>"
```

Esses sinais não fazem parte da URL. Enviá-los ao `requests` causa:

```text
InvalidSchema: No connection adapters were found
```

No código, as URLs precisam estar assim:

```python
"https://lilianweng.github.io/posts/2023-06-23-agent/"
```

### 5. Dependência `bs4` ausente

O `WebBaseLoader` usa BeautifulSoup para interpretar o HTML. Por isso, instalar somente as dependências listadas originalmente pelo tutorial pode não ser suficiente.

A dependência correta é instalada com:

```powershell
python -m pip install beautifulsoup4
```

O nome do pacote é `beautifulsoup4`, mas o import interno aparece como `bs4`.

### 6. Chave OpenAI fictícia no tutorial

O tutorial mostra:

```python
OpenAIEmbeddings(openai_api_key="api_key")
```

`api_key` é apenas um texto de exemplo. Não é uma chave válida e causa erro `401 Incorrect API key`.

Além disso, o tutorial usa Ollama para gerar a resposta, mas usa OpenAI para gerar os embeddings. Portanto, apesar do título indicar uma solução com Ollama, a versão original ainda depende da API da OpenAI.

O projeto foi ajustado para não exigir uma chave OpenAI:

```python
from langchain_ollama import OllamaEmbeddings

embedding=OllamaEmbeddings(model="nomic-embed-text")
```

Essa versão gera os embeddings localmente com Ollama.

Se preferir usar OpenAI, instale `langchain-openai`, crie uma chave em:

https://platform.openai.com/api-keys

E configure-a no PowerShell sem colocá-la no código:

```powershell
$env:OPENAI_API_KEY="sua-chave-real"
```

### 7. Aviso sobre `langchain-community`

O programa pode exibir um aviso informando que `langchain-community` está sendo descontinuado como pacote de integrações gerais. Esse aviso não é a causa da falha atual.

Issue de referência:

https://github.com/langchain-ai/langchain-community/issues/674

O pacote ainda fornece `WebBaseLoader` e `SKLearnVectorStore` usados neste projeto, por isso ele continua instalado.

### 8. Aviso sobre `USER_AGENT`

O `WebBaseLoader` pode exibir:

```text
USER_AGENT environment variable not set
```

É um aviso, não um erro. Para identificar as requisições, pode-se definir:

```powershell
$env:USER_AGENT="RAG-local/1.0"
```

### 9. Tamanho dos trechos

O tutorial descreve `chunk_size=250` como se fossem caracteres, mas o código usa `from_tiktoken_encoder`. Nesse caso, o tamanho é calculado em tokens, que não equivalem diretamente a caracteres.

### 10. Separação dos documentos recuperados

Ao juntar o conteúdo recuperado, use uma quebra de linha real:

```python
doc_texts = "\n".join(doc.page_content for doc in documents)
```

Uma string escrita como `"\\n"` produz os caracteres literais `\\n`, em vez de separar os documentos visualmente.

## Links do tutorial e da documentação

- Tutorial usado como base: https://www.datacamp.com/pt/tutorial/llama-3-1-rag
- Ollama: https://ollama.com/
- Text splitters do LangChain: https://python.langchain.com/docs/how_to/recursive_text_splitter/
- Pacote `langchain-text-splitters`: https://pypi.org/project/langchain-text-splitters/
- API keys da OpenAI: https://platform.openai.com/api-keys
- Issue de migração do `langchain-community`: https://github.com/langchain-ai/langchain-community/issues/674

## Observações

- O programa depende de internet para baixar os artigos.
- O programa depende do Ollama para gerar respostas e embeddings.
- Os modelos do Ollama ocupam espaço em disco e podem exigir memória significativa.
- Não publique chaves de API no Git ou em arquivos enviados para repositórios públicos.
- O tutorial original usa versões antigas do LangChain e precisa de pequenos ajustes para funcionar com as versões atuais.
