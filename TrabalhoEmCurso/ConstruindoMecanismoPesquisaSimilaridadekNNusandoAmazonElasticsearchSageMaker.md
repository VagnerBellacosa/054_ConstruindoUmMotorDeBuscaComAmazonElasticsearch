# Construindo um mecanismo de pesquisa de similaridade k-NN usando Amazon Elasticsearch e SageMaker

## Guia passo a passo para criar um mecanismo de pesquisa de similaridade de documentos eficiente e escalonável



![img](https://ichi.pro/assets/images/max/724/1*djYJeZ7jVij-EByYnMJOig.jpeg)Foto de NeONBRAND no Unsplash

O serviço Amazon Elasticsearch recentemente adicionou suporte para pesquisa de vizinho mais próximo. Ele permite que você execute pesquisas k-NN de alta escala e baixa latência em milhares de dimensões com a mesma facilidade de executar qualquer consulta Elasticsearch regular.

A pesquisa por similaridade k-NN é fornecida pela Open Distro for Elasticsearch, uma distribuição licenciada pelo Apache 2.0 do Elasticsearch.



Neste post, vou mostrar como construir uma API de pesquisa de perguntas de similaridade escalonável usando Amazon Sagemaker, Amazon Elasticsearch, Amazon Elastic File System (EFS) e Amazon ECS.

# O que vamos cobrir neste exemplo:

- Implante e execute uma instância de notebook Sagemaker no VPC.
- Monte o EFS na instância do notebook.
- Baixe o conjunto de dados Quora Question Pairs e, em seguida, mapeie as questões de comprimento variável do conjunto de dados para vetores de comprimento fixo usando o modelo DistilBERT.
- Crie uma tarefa downstream para reduzir as dimensões de incorporação e salvar o incorporador de frases no EFS.
- Transforme o texto das perguntas em vetores e indexe todos os vetores no Elasticsearch.
- Implante um Flask rest api em contêiner para ECS.

![img](https://ichi.pro/assets/images/max/724/1*Q_c2o6l_HqQcrnxIuNwySQ.png)

# Implantar e executar a instância do notebook Sagemaker no VPC

Primeiro, vamos criar uma instância de notebook Sagemaker que se conecta ao Elasticsearch e certifique-se de que estão no mesmo VPC.



Para configurar as opções de VPC no **console Sagemaker,** na seção **Rede** da página **Criar instância de notebook** , defina os detalhes de configuração da rede VPC, como IDs de sub-rede VPC e IDs de grupo de segurança:

![img](https://ichi.pro/assets/images/max/724/1*S6aaS4CeCnwhKEVQh2_oBg.jpeg)

# Montagem de EFS em instância de notebook

Faremos todas as etapas necessárias de transformação de frases em um **bloco de notas SageMaker** (código encontrado [**aqui**](https://github.com/yai333/knnelasticsearch/blob/master/knn-search.ipynb) ).

Agora, montando o EFS no diretório do **modelo** , para obter mais detalhes sobre o EFS, verifique o [documento oficial da AWS](https://docs.aws.amazon.com/efs/latest/ug/gs-step-two-create-efs-resources.html) .



```shell


%%sh
mkdir model
sudo mount -t nfs \
    -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 \
    fs-xxxxxx.efs.ap-southeast-2.amazonaws.com:/ \
    ./model
```



**NOTA** :

- `fs-xxxxx.efs.ap-southeast-2.amazonaws.com` é o nome DNS de EFS.
- Os destinos do EFS Mount e o Sagemaker estão no mesmo VPC.

![img](https://ichi.pro/assets/images/max/724/1*Pjsz05_yOIMn9AOO8d3sQQ.jpeg)

Para executar a pesquisa do vizinho mais próximo, temos que obter os encaixes de frases e tokens. Podemos usar [transformadores de frases](https://github.com/UKPLab/sentence-transformers) que são embeddings de frases usando BERT / RoBERTa / DistilBERT / ALBERT / XLNet com PyTorch. Ele nos permite mapear frases em representações de comprimento fixo em apenas algumas linhas de código.

**Usaremos o** modelo **DistilBERT** leve para gerar embeddings de frases neste exemplo, observe que o número de unidades ocultas do **DistilBERT** é 768. Esta dimensão parece muito grande para o índice Elasticsearch, podemos reduzir a dimensão para 256 adicionando uma camada densa após o pooling:



```python


from sentence_transformers import models, losses, SentenceTransformer


word_embedding_model = models.DistilBERT('distilbert-base-uncased')


pooling_model = models.Pooling(word_embedding_model.get_word_embedding_dimension(),
                            pooling_mode_mean_tokens=True,
                            pooling_mode_cls_token=False,
                            pooling_mode_max_tokens=False)
# reduce dim from 768 to 256
dense_model = models.Dense(in_features=768, out_features=256)
transformer = SentenceTransformer(modules=[word_embedding_model, pooling_model, dense_model])
```



Em seguida, salve o incorporador de frase no diretório montado no EFS:



```
transformer.save("model/transformer-v1/")
```



```python
import pandas as pd
from kaggle.api.kaggle_api_extended import KaggleApi


api = KaggleApi()
api.authenticate()
api.dataset_download_files("quora/question-pairs-dataset", path='quora_dataset', unzip=True)
```



Em seguida, extraia o texto completo de cada pergunta para o dataframe:



```python
import pandas as pd


pd.set_option('display.max_colwidth', -1)
df = pd.read_csv("quora_dataset/questions.csv", usecols=["qid1", "question1"], index_col=False)
df = df.sample(frac=1).reset_index(drop=True)
df_questions_imp = df[:5000]
```



## Transformar o texto das perguntas em vetores e indexar todos os vetores no Elasticsearch

Para começar, crie um índice kNN,



```python
import boto3
from requests_aws4auth import AWS4Auth
from elasticsearch import Elasticsearch, RequestsHttpConnection


region = 'ap-southeast-2'
service = 'es'
ssm = boto3.client('ssm', region_name=region)
es_parameter = ssm.get_parameter(Name='/KNNSearch/ESUrl')
es_host = es_parameter['Parameter']['Value']
credentials = boto3.Session().get_credentials()
awsauth = AWS4Auth(credentials.access_key, credentials.secret_key,
                   region, service, session_token=credentials.token)
                   
es = Elasticsearch(
    hosts=[{'host': es_host, 'port': 443}],
    http_auth=awsauth,
    use_ssl=True,
    verify_certs=True,
    connection_class=RequestsHttpConnection
)


knn_index = {
    "settings": {
        "index.knn": True
    },
    "mappings": {
        "properties": {
            "question_vector": {
                "type": "knn_vector",
                "dimension": 256
            }
        }
    }
}


es.indices.create(index="questions",body=knn_index,ignore=400)
```



em seguida, transforme e indexe os vetores de questão em Elasticsearch.



```python
def es_import(df):
    for index, row in df.iterrows():
        vectors = local_transformer.encode([row["question1"]])
        es.index(index='questions',
                 id=row["qid1"], 
                 body={"question_vector": vectors[0].tolist(), 
                       "question": row["question1"]})
        
es_import(df_questions_imp)
```



As perguntas no Elasticsearch têm a seguinte estrutura:



```
{'question_vector': [-0.06435434520244598, ... ,0.0726890116930008],
'question': 'How hard is it to learn to play piano as an adult?'}
```

# Implantar um Flask rest api em contêineres

Usaremos um modelo de formação de nuvem de amostra para criar o cluster ECS e serviço em VPC (modelos e scripts bash encontrados [**aqui**](https://github.com/yai333/knnelasticsearch/tree/master/cf-templates) ).

Usaremos **volumes EFS com ECS** , o fluxo de pesquisa é **1) O** aplicativo Flask carrega o embedder de frase salva no volume EFS, **2)** transforma a frase do parâmetro de entrada em vetores, **3)** consulta os vizinhos K-mais próximos no Elasticsearch.



```python
import json
import boto3
from flask import Flask
from flask_restful import reqparse, Resource, Api
from elasticsearch import Elasticsearch, RequestsHttpConnection
from requests_aws4auth import AWS4Auth
from sentence_transformers import SentenceTransformer


app = Flask(__name__)
api = Api(app)
region = 'ap-southeast-2'
ssm = boto3.client('ssm', region_name=region)
es_parameter = ssm.get_parameter(Name='/KNNSearch/ESUrl')
host = es_parameter['Parameter']['Value']
service = 'es'
credentials = boto3.Session().get_credentials()
awsauth = AWS4Auth(credentials.access_key, credentials.secret_key,
                   region, service, session_token=credentials.token)


parser = reqparse.RequestParser()
parser.add_argument('question')
parser.add_argument('size')
parser.add_argument('min_score')


es = Elasticsearch(
    hosts=[{'host': host, 'port': 443}],
    http_auth=awsauth,
    use_ssl=True,
    verify_certs=True,
    connection_class=RequestsHttpConnection
)


transform_model = SentenceTransformer(
    'model/transformer-v1/')


class SimilarQuestionList(Resource):
    def post(self):
        args = parser.parse_args()
        sentence_embeddings = transform_model.encode([args
                                                      ["question"]])
        res = es.search(index="questions",
                        body={
                            "size": args.get("size", 5),
                            "_source": {
                                "exclude": ["question_vector"]
                            },
                            "min_score": args.get("min_score", 0.3),
                            "query": {
                                "knn": {
                                    "question_vector": {
                                        "vector": sentence_embeddings[0].tolist(),
                                        "k": args.get("size", 5)
                                    }
                                }
                            }
                        })
        return res, 201




api.add_resource(SimilarQuestionList, '/search')


if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=8000)
```



Agora temos flask api em execução no contêiner ECS, vamos usar a função de pesquisa básica para encontrar perguntas semelhantes quando se trata da consulta: “ ***Qual é a melhor maneira de ganhar dinheiro online\*** ?”:



```
$curl --data 'question=What is best way to make money online?' --data 'size=5' --data 'min_score=0.3'  -X POST http://knn-s-publi-xxxx-207238135.ap-southeast-2.elb.amazonaws.com/search
```



```json
{
    "took": 10,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 15,
            "relation": "eq"
        },
        "max_score": 0.69955945,
        "hits": [
            {
                "_index": "questions",
                "_type": "_doc",
                "_id": "210905",
                "_score": 0.69955945,
                "_source": {
                    "question": "What is an easy way make money online?"
                }
            },
            {
                "_index": "questions",
                "_type": "_doc",
                "_id": "547612",
                "_score": 0.61820024,
                "_source": {
                    "question": "What is the best way to make passive income online?"
                }
            },
            {
                "_index": "questions",
                "_type": "_doc",
                "_id": "1891",
                "_score": 0.5624176,
                "_source": {
                    "question": "What are the easy ways to earn money online?"
                }
            },
            {
                "_index": "questions",
                "_type": "_doc",
                "_id": "197580",
                "_score": 0.46031988,
                "_source": {
                    "question": "What is the best way to download YouTube videos for free?"
                }
            },
            {
                "_index": "questions",
                "_type": "_doc",
                "_id": "359930",
                "_score": 0.45543614,
                "_source": {
                    "question": "What is the best way to get traffic on your website?"
                }
            }
        ]
    }
}
```



Como você pode ver, o resultado foi incrível, você também pode ajustar seus próprios métodos de incorporação de frases, de modo que obtenha embeddings de frases específicas de tarefas para a pesquisa k-NN.

Ótimo! Nós temos o que precisamos! Espero que você tenha achado este artigo útil.