# POSTMAN

## 1. Estrutura no Postman

A Collection ficará organizada assim:

### ServeRest CRUD - Usuários

| Caso de Teste | Método HTTP | Descrição |
| --- | --- | --- |
| CT01 | — | Validar status HTTP e mensagens — testes de resposta da API |
| CT02 | POST | Cadastrar usuário |
| CT03 | GET | Consultar usuários |
| CT04 | GET | Consultar usuário cadastrado |
| CT05 | PUT | Atualizar usuário |
| CT06 | GET | Validar atualização do usuário |
| CT07 | DELETE | Excluir usuário |
| CT08 | GET | Validar exclusão do usuário |

A URL-base será `https://serverest.dev`, e o identificador do usuário criado será armazenado em uma variável para reutilização nas próximas requisições.

## 2. Configurar o ambiente

No Postman, crie um Environment chamado `ServeRest - QA` com estas variáveis:

| Variável | Valor inicial |
| --- | --- |
| `baseUrl` | `https://serverest.dev` |
| `usuarioId` | (vazio) |
| `nomeUsuario` | `Usuario QA Postman` |
| `emailUsuario` | (definido no script abaixo) |
| `senhaUsuario` | `Teste@123` |

O cadastro exige um e-mail ainda não utilizado. Por isso, vamos gerar um e-mail exclusivo a cada execução. Os endpoints e as mensagens abaixo seguem a documentação da API ServeRest.

## 3. Criar as requisições e os testes

Para cada requisição, abra a aba Scripts → Post-response (ou Tests, dependendo da versão do Postman) e cole o script correspondente.

### CT01 — Validar status HTTP e mensagens

Esse cenário pode ser implementado como validação do cadastro no CT02, evitando uma requisição extra. O exemplo de cadastro abaixo valida o status `201` e a mensagem de sucesso.

### CT02 — Cadastrar usuário

POST`{{baseUrl}}/usuarios`

Headers

`Content-Type: application/json`

Body → raw → JSON

```json
{
  "nome": "{{nomeUsuario}}",
  "email": "{{emailUsuario}}",
  "password": "{{senhaUsuario}}",
  "administrador": "false"
}
```

Pre-request Script

```javascript
const email = `qa.postman.${Date.now()}@example.com`;
pm.environment.set("emailUsuario", email);
```

Post-response Script

```javascript
pm.test("CT01 - Status HTTP 201", function () {
    pm.response.to.have.status(201);
});

pm.test("CT02 - Mensagem de cadastro", function () {
    const body = pm.response.json();
    pm.expect(body.message).to.eql("Cadastro realizado com sucesso");
    pm.expect(body._id).to.be.a("string").and.not.empty;
    pm.environment.set("usuarioId", body._id);
});
```

Importante: execute o CT02 antes dos demais. O ID retornado será salvo em `usuarioId`, permitindo que as próximas requisições usem `{{usuarioId}}`.

### CT03 — Consultar usuários

GET`{{baseUrl}}/usuarios`

Post-response Script


```javascript
pm.test("CT03 - Status HTTP 200", function () {
    pm.response.to.have.status(200);
});

pm.test("CT03 - Lista de usuários retornada", function () {
    const body = pm.response.json();
    pm.expect(body.quantidade).to.be.a("number");
    pm.expect(body.usuarios).to.be.an("array");
});
```

### CT04 — Consultar usuário cadastrado

GET`{{baseUrl}}/usuarios/{{usuarioId}}`

Post-response Script


```javascript
pm.test("CT04 - Status HTTP 200", function () {
    pm.response.to.have.status(200);
});

pm.test("CT04 - Dados do usuário conferidos", function () {
    const body = pm.response.json();

    pm.expect(body._id).to.eql(
        pm.environment.get("usuarioId")
    );
    pm.expect(body.nome).to.eql(
        pm.environment.get("nomeUsuario")
    );
    pm.expect(body.email).to.eql(
        pm.environment.get("emailUsuario")
    );
    pm.expect(body.administrador).to.eql("false");
});
```

### CT05 — Atualizar usuário

PUT`{{baseUrl}}/usuarios/{{usuarioId}}`

Body → raw → JSON

```json
{
  "nome": "Usuario QA Atualizado",
  "email": "{{emailUsuario}}",
  "password": "{{senhaUsuario}}",
  "administrador": "false"
}
```

Post-response Script


```javascript
pm.test("CT05 - Status HTTP 200", function () {
    pm.response.to.have.status(200);
});

pm.test("CT05 - Mensagem de atualização", function () {
    const body = pm.response.json();
    pm.expect(body.message).to.eql(
        "Registro alterado com sucesso"
    );

    pm.environment.set(
        "nomeUsuario",
        "Usuario QA Atualizado"
    );
});
```

### CT06 — Validar atualização do usuário

GET`{{baseUrl}}/usuarios/{{usuarioId}}`

Post-response Script


```javascript
pm.test("CT06 - Status HTTP 200", function () {
    pm.response.to.have.status(200);
});

pm.test("CT06 - Nome atualizado", function () {
    const body = pm.response.json();

    pm.expect(body.nome).to.eql(
        "Usuario QA Atualizado"
    );
    pm.expect(body.email).to.eql(
        pm.environment.get("emailUsuario")
    );
});
```

### CT07 — Excluir usuário

DELETE`{{baseUrl}}/usuarios/{{usuarioId}}`

Post-response Script


```javascript
pm.test("CT07 - Status HTTP 200", function () {
    pm.response.to.have.status(200);
});

pm.test("CT07 - Mensagem de exclusão", function () {
    const body = pm.response.json();

    pm.expect(body.message).to.eql(
        "Registro excluído com sucesso"
    );
});
```

### CT08 — Validar exclusão do usuário

GET`{{baseUrl}}/usuarios/{{usuarioId}}`

Post-response Script


```javascript
pm.test("CT08 - Usuário não encontrado", function () {
    pm.response.to.have.status(400);

    const body = pm.response.json();

    pm.expect(body.message).to.eql(
        "Usuário não encontrado"
    );
});
```

A API documenta status 400 para a consulta de um usuário inexistente.

## 4. Executar a suíte no Postman

1. Salve as oito requisições na Collection `ServeRest CRUD - Usuários`.
2. Selecione o Environment `ServeRest - QA`.
3. Organize as requisições na ordem CT01–CT08, considerando que CT01 é validado junto ao cadastro CT02.
4. Abra a Collection e clique em Run.
5. Execute a suíte pelo Collection Runner e consulte os resultados dos testes.

A ordem é importante porque o cadastro cria o usuário, a atualização altera os dados e a exclusão remove o registro. Os testes seguintes dependem do ID salvo na variável de ambiente.