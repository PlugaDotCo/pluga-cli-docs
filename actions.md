# Actions

Os actions \(ou ações\) são funções que definem como a Pluga deve inserir informações na sua aplicação.

As aplicações dentro da Pluga podem ter um, vários, ou mesmo nenhum action. Tudo depende se sua aplicação possui informações que sejam interessantes para serem inseridas a partir dos triggers de [outras aplicações integradas na Pluga](https://pluga.co/ferramentas).

Para que uma automatização na Pluga funcione é necessário que exista um trigger numa aplicação A e um action numa aplicação B. O objetivo dessa seção é mostrar como criar actions que permitirão à Pluga ligar sua aplicação aos triggers já presentes na nossa plataforma.

Cada action da sua aplicação deve ficar numa pasta em `lib/actions`, sendo nomeada com o padrão [snake\_case](https://en.wikipedia.org/wiki/Snake_case), contendo um arquivo JSON \(`meta.json`\) e um JavaScript \(`index.js`\). Como fizemos em outras seções, vamos explicar o processo a partir de exemplos reais.

## Configuração em JSON \(meta.json\)

No arquivo `meta.json` você vai configurar parâmetros como nome, campos que precisam ser preenchidos pelo usuário e outras informações estáticas do seu action.

Abaixo temos a configuração do action de **criar/atualizar usuário** da aplicação [Intercom](https://pluga.co/ferramentas/intercom).

{% code-tabs %}
{% code-tabs-item title="lib/actions/upsert\_user/meta.json" %}
```javascript
{
  "name": "Criar/atualizar usuário",
  "description": "Cria ou atualiza um usuário, junto com a empresa associada.",
  "action_fields": {
    "fields": [
      {
        "key": "email",
        "name": {
          "pt_BR": "E-mail",
          "en": "Email"
        },
        "description": {
          "pt_BR": "E-mail do usuário",
          "en": "User email"
        },
        "required": true,
        "advanced": false,
        "field_type": "custom",
        "data_type": "string"
      },
      {
        "key": "name",
        "name": {
          "pt_BR": "Nome",
          "en": "Name"
        },
        "description": {
          "pt_BR": "Nome do usuário",
          "en": "User name"
        },
        "required": false,
        "advanced": false,
        "field_type": "custom",
        "data_type": "string"
      },
      {
        "key": "company.name",
        "name": {
          "pt_BR": "Nome da empresa",
          "en": "Company name"
        },
        "description": {
          "pt_BR": "Nome da empresa do usuário",
          "en": "User's company name"
        },
        "required": false,
        "advanced": false,
        "field_type": "custom",
        "data_type": "string"
      },
      // ...
    ]
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Vamos passar campo a campo para entender os seus significados e seus possíveis valores.

* **name**: Nome do seu action e como ele será chamado nas automatizações que serão geradas com ele. 
* **description**: Uma breve descrição sobre seu action e que informações ele resgata. Isso é bastante importante para que a equipe da Pluga possa entender melhor as possibilidades de combinação com outras aplicações. 
* **action\_fields**: A Pluga vai montar um objeto JSON para ser processado pelo seu action a cada evento retornado pelo trigger associado pelo usuário. As configurações de `action_fields` definem quais atributos esse objeto deve ter. 
  * **fields**: Lista de campos que serão exibidos no painel da Pluga para que o usuário possa preencher com informações estáticas ou dinâmicas, através dos atributos dos triggers. 
    * **key**: Identificador do atributo em dot notation. Ou seja, para gerar o objeto `{ "email": "johndoe@example.com" }` usamos `email` e para gerar `{ "payer": { "email": "johndoe@example.com" } }` usamos `payer.email`. 
    * **name**: Nome do campo que será exibido para o usuário. Lembre-se de preencher para todos os idiomas disponíveis na Pluga. 
    * **description**: Uma breve descrição sobre o campo, será exibido como um tooltip acima do campo para informar usuários que tiverem dúvidas sobre como preencher esse campo. Lembre-se de preencher para todos os idiomas disponíveis na Pluga. 
    * **required**: Indica se o campo é obrigatório ou não. 
    * **advanced**: Define se o campo deve aparecer numa área de "campos avançados" no painel da Pluga. É recomendado incluir apenas campos opcionais como `"advanced": true`. 
    * **field\_type**: Indica que tipo de campo será exibido para o usuário preencher no painel da Pluga. Os valores possíveis são: 
      * **string**: Campo padrão de texto, apenas com conteúdo estático. 
      * **custom**: Campo de texto com a possibilidade do usuário inserir informações dinâmicas a partir de referências a atributos do trigger que ele estiver ligando ao seu action. 
      * **custom\_textarea**: Campo de texto com as mesmas características do tipo **custom**, porém usando o elemento HTML `textarea` como base. 
      * **dropdown**: Campo de multipla escolha, as opções serão exibidas de acordo com a configuração do parâmetro **list**. 
    * **list**: Define as opções que serão exibidas para o usuário em campos do tipo **dropdown**. 
      * **type**: Define que estratégia a Pluga deve usar para listar as opções do seu campo. Os valores possíveis são `local` e `remote`. 
      * **data**: Lista de opções para o tipo `local`. Deve ser configurada como um array de objetos, onde cada objeto deve ter um atributo `label` e um `value`. 
      * **helper\_method**: Identificador do helper method que será usado para gerar a lista de opções para o tipo `remote`. Você pode entender melhor esse assunto na seção [Helper methods](helper-methods.md). 
    * **data\_type**: Indica o tipo do atributo para que a Pluga possa fazer algumas conversões, quando necessário. Os valores possíveis são `string`, `integer`, `decimal`, `datetime` e `boolean`.

{% hint style="info" %}
A configuração de `key` dos `action_fields.fields` não suporta arrays, logo caso o seu action precise de algum dado em array você deve tratar esse dado na parte em JavaScript da integração, geralmente fazendo um **split**.
{% endhint %}

## Configuração em JavaScript \(index.js\)

No arquivo `index.js` você vai configurar o funcionamento dinâmico do seu action. Você deve export uma função `handle` que recebe 2 objetos como argumentos e retorna uma [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise), esses argumentos sâo:

* **plg**: Objeto contendo bibliotecas auxiliares para o desenvolvimento do seu action, como por exemplo a axios. 
* **event**: Objeto contendo os dados que seu action vai enviar para a sua API, como chaves de autenticação e dados de input mapeado pelo usuário da automatização.

Sua função `handle` deve fazer requisições na sua API com as informações recebidas e retornar \(dentro de uma Promise\) o objeto inserido pela ação.

Abaixo temos a configuração do action de **criar tarefa** da aplicação [Asana](https://pluga.co/ferramentas/asana).

{% code-tabs %}
{% code-tabs-item title="lib/actions/create\_task/index.js" %}
```javascript
/**
 * Action handler
 *
 * @param {object} plg - Pluga developer platform toolbox.
 * @param {object} plg.axios - [axios](https://github.com/axios/axios)
 *
 * @param {object} event - Event bundle to handle.
 * @param {object} event.meta - Pluga event meta data.
 * @param {string} event.meta.baseURI - Environment base URI.
 * @param {object} event.auth - Your app.json auth fields.
 * @param {object} event.input - Your meta.json fields.
 *
 * @returns {Promise} Promise object with the action result.
 */

const parseArray = (target, key) => {
  if (!target[key]) return;

  if (target[key].length) {
    target[key] = target[key].join();
  } else {
    delete target[key];
  }
};

const compactObj = (obj) => {
  const res = {}
  Object.keys(obj).forEach((key) => {
    if (obj[key]) res[key] = obj[key];
  });
  return res;
};

exports.handle = (plg, event) => {
  if (event.input.project) {
    event.input.memberships = [{ project: event.input.project }];

    if (event.input.section) {
      event.input.memberships[0].section = event.input.section;
    }
  }

  if (!event.input.notes) {
    delete event.input.notes;
  }

  event.input.custom_fields = compactObj(event.input.custom_fields || {});

  delete event.input.project;
  delete event.input.section;

  parseArray(event.input, 'followers');
  parseArray(event.input, 'tags');

  return plg.axios({
    method: 'post',
    url: `${event.meta.baseURI}/tasks`,
    headers: {
      Authorization: `Bearer ${event.auth.access_token}`
    },
    data: {
      data: event.input
    }
  }).then(res => res.data.data).catch((err) => {
    throw new Error(err.response.data.errors[0].message);
  });
};
```
{% endcode-tabs-item %}
{% endcode-tabs %}

