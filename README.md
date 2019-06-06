# Início

## Motivação

Na Pluga, nós mesmos conectamos diversas ferramentas web, automatizando as tarefas diárias. Só que esse processo não é tão rápido quanto poderia ser. Pode ser muito melhor! E assim, decidimos escalar exponencialmente o número de empresas conectadas. Mas como?

Vamos adotar a tática do "crowdsource" e deixar que as empresas que queiram conectar suas aplicações web na Pluga possam fazer por elas mesmo.

Desta forma, permitiremos que as empresas parceiras da Pluga conectem as suas aplicações através de uma plataforma de aplicações.

## Primeiro passo

Antes de começar a codar sua integração você vai precisar instalar nosso CLI, ele será sua principal ferramenta para desenvolver seu módulo de integração com a Pluga.

A plataforma da Pluga foi criada para rodar módulos de integrações feitos em JavaScript \(Node.js\) com o auxílio de alguns arquivos de configuração em JSON, e o Pluga CLI vai te ajudar tanto com a estrutura de pastas e arquivos esperado pela plataforma, como também com testes e até mesmo o envio do seu código para que a nossa equipe analise.

Como o próprio CLI também é feito em Node.js, você pode instalar ele a partir do `npm` e iniciar seu projeto com o comando `init`.

```bash
$ npm install -g pluga-cli
$ pluga init my_app
```

_Você pode consultar mais informações sobre a instalação e comandos do Pluga CLI em_ [_github.com/PlugaDotCo/pluga-cli_](https://github.com/PlugaDotCo/pluga-cli)_._

Para entender como criar uma integração na Pluga, devemos pensar que a aplicação será composta por essencialmente **4 componentes**:

1. App, com descrição da sua aplicação e do método de autenticação da sua API;
2. Triggers, com funções onde a Pluga recebe informações vindas da sua API;
3. Actions, com funções onde a Pluga envia informações para a sua API;
4. Helper Methods, que auxiliam os triggers e actions com dados extras.

Após a criação do seu projeto, você pode iniciar sua configuração de App, onde vai definir informações como texto de descrição e autenticação da sua API, tudo em `lib/app.json`.

