# Executar via linha de comando (opcional)

Para executar o teste via linha de comando, precisamos exportar do Postman a Collection e Environments.

## Estrutura

```bash
postman-serverest-crud/
├── collections/
│   └── ServeRest-CRUD.postman_collection.json
├── environments/
│   └── ServeRest-QA.postman_environment.json
├── docs
│   └── convite.md
├── reports
│   └── relatorio-serverest.html
└── README.md
```

Verificar se o arquivo 'JSON' está correto:

```bash
node -e "JSON.parse(require('fs').readFileSync('collections/ServeRest-CRUD.postman_collection.json', 'utf8')); console.log('JSON válido')"
```

Se quiser executar a Collection fora da interface gráfica, exporte a Collection e o Environment como arquivos JSON e use o Newman:

Instale o 'NEWMAN':

```bash
npm install -g newman

newman run collections/ServeRest-CRUD.postman_collection.json -e environments/ServeRest-QA.postman_environment.json
```

Com o NEWMAN instalado, execute a collection:

```bash
newman run collections/ServeRest-CRUD.postman_collection.json -e environments/ServeRest-QA.postman_environment.json
```

O Newman permite executar Collections do Postman em pipelines de CI/CD e gerar relatórios de execução.

Para Gerar um relatório HTML e deixar seu projeto mais profissional e facilitar a publicação no GitHub, você pode gerar um relatório HTML da execução.

1. Instale o reporter:

```bash
npm install -g newman-reporter-htmlextra
```

2. Execute a Collection com o relatório:

Crie a pasta reports antes de executar o comando, caso ela ainda não exista.

```bash
newman run collections/ServeRest-CRUD.postman_collection.json \
  -e environments/ServeRest-QA.postman_environment.json \
  -r cli,htmlextra \
  --reporter-htmlextra-export reports/relatorio-serverest.html
```

## 6. Estrutura final do projeto convertido

```bash
postman-serverest-crud/
├── collections/
│   └── ServeRest-CRUD.postman_collection.json
├── environments/
│   └── ServeRest-QA.postman_environment.json
├── docs
│   └── convite.md
├── reports
│   └── relatorio-serverest.html
└── README.md
```

Os exemplos acima já contêm os métodos HTTP, os corpos das requisições e os scripts de asserção. Para obter o projeto importável de uma só vez, o próximo passo é reunir essas requisições em um único arquivo JSON de Collection do Postman, com as variáveis e os testes configurados.