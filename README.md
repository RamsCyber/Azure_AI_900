# Azure_AI_900

Capa do Projeto de Tradução

📦 1 Criar um Grupo de Recursos


No portal do Azure, crie um grupo de recursos para organizar os recursos do projeto, como o Azure OpenAI.

img

passo1.1


🧠2 Provisionar o Serviço Azure OpenAI


Crie uma instância do serviço Azure OpenAI. Selecione a região desejada (East por exemplo), escolha um nome para a instância, e opte pela camada gratuita para começar.

passo2

passo2.1



🤖 3 Implantar o Modelo GPT-4 Mini


No Azure OpenAI Studio, implante o modelo GPT-4 Mini, selecionando o "Modo Básico" e a opção "Standard" para a implantação. Anote a chave de API e o endpoint do Azure OpenAI, pois você precisará deles mais tarde.

passo3

passo3.1



📰 Primeiro Projeto - Tradução de Artigos


📚 1 Importar as Bibliotecas


from bs4 import BeautifulSoup
import requests, uuid, json
import os
from dotenv import load_dotenv


🔐 2 Definir as Variáveis de Ambiente


Defina as variáveis de ambiente para a chave de assinatura do endpoint do Azure OpenAI:

# Carregar variáveis de ambiente do arquivo .env
load_dotenv()
azureai_endpoint = os.getenv("AZURE_ENDPOINT")
API_KEY = os.getenv("AZURE_OPENAI_KEY")
Obs: Precisar criar um arquivo .env na raiz do projeto com as suas credenciais obtidas na Azure, exemplo:

AZURE_OPENAI_KEY=sua_chave_openai
AZURE_ENDPOINT=seu_endpoint_openai


🔍 3 Criar a Função de Extração de Texto


Crie uma função Python chamada extract_text() para extrair texto de uma URL fornecida. Esta função deve:

Usar requests.get() para buscar o conteúdo HTML da URL.
Verificar se a resposta HTTP tem o código de status 200 (sucesso).
Criar um objeto BeautifulSoup para analisar o HTML.
Remover tags <script> e <style> do HTML.
Extrair o texto usando soup.get_text().
Limpar o texto removendo caracteres desnecessários, como quebras de linha extras.
Retornar o texto limpo.
url="https://dev.to/eric_dequ/steve-jobs-the-visionary-who-blended-spirituality-and-technology-3ppi"
def extract_text(url):
    response = requests.get(url)
    if response.status_code == 200:
        soup = BeautifulSoup(response.text, 'html.parser')
        for script in soup(["script", "style"]):
            script.decompose()
        text = soup.get_text(" ", strip=True)
        return text
    else:
        print("Falha ao buscar a URL. Código de status:", response.status_code)
        return None
extract_text(url)
Resultado:

'Steve Jobs The Visionary Who Blended Spirituality and Technology - DEV Community Skip to content Navigation menu Search Powered by Search Algolia Search Log in Create account DEV Community Close Add reaction Like Unicorn Exploding Head Raised Hands Fire Jump to Comments Save More... Copy link Copy link Copied to Clipboard Share to Twitter Share to LinkedIn Share to Reddit Share to Hacker News Share to Facebook Share to Mastodon Report Abuse Eric Dequevedo Posted on Jun 28 • Originally published at rics-notebook.com Steve Jobs The Visionary Who Blended Spirituality and Technology # stevejobs # apple # iphone # spirituality Steve Jobs: The Visionary Who Blended Spirituality and Technology 🍎🕉️ Steve Jobs,


🌎 4 Criar a Função de Tradução de Texto



Crie uma função Python chamada translate_article() para traduzir o texto extraído usando o Azure openai. Esta função deve:

Definir o endpoint da API de tradução.(ENDPOINT)
Criar um cabeçalho de solicitação com a chave de assinatura.(headers)
Definir o corpo da solicitação com o texto a ser traduzido.(payload)
Enviar uma solicitação POST para a API de tradução usando requests.post().
Extrair o texto traduzido da resposta JSON.
Retornar o texto traduzido.
def translate_article(text, lang):
    headers = {
    "Content-Type": "application/json",
    "api-key": API_KEY,
    }
    
    # Payload for the request
    payload = {
      "messages": [
        {
          "role": "system",
          "content": [
            {
              "type": "text",
              "text": "Você atua como tradutor de textos"
            }
          ]
        },
        {
          "role": "user",
          "content": [
            {
              "type": "text",
              "text": f"traduza: {text} para o idioma {lang} e responda apenas com a tradução no formato markdown"
            }
          ]
        },    
      ],
      "temperature": 0.7,
      "top_p": 0.95,
      "max_tokens": 900
    }
    
    ENDPOINT = azureai_endpoint
    
    # Send request
    try:
        response = requests.post(ENDPOINT, headers=headers, json=payload)
        response.raise_for_status()  # Will raise an HTTPError if the HTTP request returned an unsuccessful status code
    except requests.RequestException as e:
        raise SystemExit(f"Failed to make the request. Error: {e}")
    
    # Handle the response as needed (e.g., print or process)
    return (response.json()['choices'][0]['message']['content'])


Realizando testes:
translate_article("The future belongs to those who believe in the beauty of their dreams. - Eleanor Roosevelt","português")
resultado:
'O futuro pertence àqueles que acreditam na beleza de seus sonhos. - Eleanor Roosevelt'


Agora vamos testar com o artigo: 'https://dev.to/eric_dequ/steve-jobs-the-visionary-who-blended-spirituality-and-technology-3ppi'


url = "https://dev.to/eric_dequ/steve-jobs-the-visionary-who-blended-spirituality-and-technology-3ppi"
text = extract_text(url)
artigo = translate_article(text,"português")
print(artigo)
resultado(uma parte apenas):
