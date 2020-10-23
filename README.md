# TCC - ETL Reativo

## Introdução

O código desse repositório foi desenvolvido utilizando Spring WebFlux com o objetivo de explorar a performance da uma aplicação com programação reativa.

Os submódulos desses repositório simulam um processo de ETL de dados, voltando seu fluxo para a tradução de arquivos.

## Pré requisitos

- Java (foi utilizado Java 11)
- Docker (recomendado)
- Um cluster Kubernetes (opcional)
- Credenciais de uma conta AWS
- Token de acesso de uma conta do GitHub

## Setup

As aplicações `extractor` e `loader` necessitam do token do GitHub para consumirem a sua API e acessar os arquivos para serem traduzidos. Deve-se definir a variável de ambiente `GITHUB_TOKEN` para esses serviços. Já a aplicação `translator` necessita das credenciais da AWS para consumir o serviço Translate e realizar a tradução dos arquivos. As variáveis de ambiente `AWS_ACCESS_KEY_ID`, `AWS_REGION` e `AWS_SECRET_KEY`.

É recomendado o uso do Docker para iniciar as aplicações através do `docker-compose.yaml`. Ele é responsável por subir todos os serviços além de iniciar o RabbitMQ, que é o serviço de mensageria usado para comunicação entre os mesmos.

O cluster Kubernetes foi utilizado para realizar os testes de performance da aplicação, todavia os arquivos descritivos dos recursos também estão no repositório (na pasta `k8s`) e podem ser aplicados tanto em um cluster local ou remoto. Para o desenvolvimento do trabalho foi utilizado o `microK8s`.

### Executando as aplicações

Para iniciar usando o Docker, basta executar esse comando no terminal: `docker-compose up --build`.

Caso as aplicações forem ser executadas no Kubernetes faz-se necessário mais modificações nos arquivos originais. Isso não será detalhado aqui.

PS: faz-se obrigatório a definição das variáveis de ambiente descritas acima ou as aplicações não funcionarão.

## Endpoints

A aplicação `extractor` possui um único endpoint na raíz, *i.e* `localhost:8080/`. Ele pode ser utilizado com o seguinte payload:

- [POST] Trigger do processo de tradução:

```json
{
  "repositoriesUrl": ["https://github.com/<user>/<repo-name>/..."],
  "fileExtension": [".<ext>"],
  "sourceLanguage": "pt",
  "targetLanguage": "en"
}
```

Atualmente essa request aceita apenas repositórios do GitHub, onde pode-se utilizar a URL do repositório em si, de alguma pasta dentro de um repositório, ou de um arquivo dentro de **uma única pasta** -- por motivos de tempo essa foi uma limitação: apenas um subnível de pastas, *i.e* `repositorio/pasta/arquivo`.

No mesmo payload ainda se especifica a extensão dos arquivos que se deseja traduzir, seu idioma de entrada e o de saída. É importante que todos os arquivos de entrada possuam o mesmo idioma de entrada para evitar possíveis erros de tradução (se isso for de fato relevante).

Exemplo de valores para `repositoriesUrl`:
- Repositório: `https://github.com/<user>/<repo>`
- Pasta: `https://github.com/<user>/<repo>/tree/<branch>/<pasta>`
- Arquivo: `https://github.com/<user>/<repo>/blob/<branch>/<pasta>/<arquivo>`

As extensões devem ser extensões de arquivos de texto. As únicas que utilizei foram `.md` e `.txt`. Os idiomas devem ser usados [conforme a documentação](https://docs.aws.amazon.com/translate/latest/dg/what-is.html) do AWS Translate.
