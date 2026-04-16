# BoltJX

## Descrição

O **BoltJX** é uma biblioteca JavaScript desenvolvida pela **Bolt Consultoria** para ser utilizada em conjunto com o **Addon de Consultas da Bolt**, como alternativa ao acesso nativo do dbExplorer do Sankhya. É especialmente indicado para ambientes onde não é possível liberar o acesso dos usuários ao dbExplorer nativo, permitindo que consultas e operações de banco de dados sejam realizadas de forma controlada e segura dentro das telas customizadas.

A classe `JX` expõe uma coleção de métodos estáticos para facilitar a manipulação de requisições HTTP, consultas ao banco de dados via addon, salvamento e deleção de registros, interação com páginas web e gerenciamento de parâmetros e cookies.

---

## Pré-requisito: Addon de Consultas da Bolt

> **Atenção:** Para utilizar os métodos de banco de dados (`consultar`, `salvar`, `novoSalvar`, `deletar`), é obrigatório que o **Addon de Consultas da Bolt** esteja instalado e ativo no ambiente Sankhya. Sem o addon, as chamadas de consulta ao banco não funcionarão.

O BoltJX foi desenvolvido para suprir a necessidade de ambientes que possuem restrições de segurança e que não permitem a liberação do dbExplorer nativo para os usuários finais. O addon atua como intermediário, garantindo que as consultas sejam executadas com as permissões corretas sem expor o acesso direto ao banco de dados.

---

## Instalação

A instalação pode ser feita baixando o arquivo `jx.js` (Homologação e Debug) ou `jx.min.js` (Produção) e importando-o no seu projeto:

```html
<script src="jx.js"></script> <!-- Homologação e Debug -->
<script src="jx.min.js"></script> <!-- Produção -->
```

A forma mais prática é utilizar a última versão atualizada diretamente do repositório do GitHub via CDN do [jsDelivr](https://www.jsdelivr.com/).

> **Obs.:** A atualização do cache do CDN do jsDelivr pode demorar até 24 horas, portanto implementações recentes podem não estar disponíveis imediatamente.

```html
<script src="https://cdn.jsdelivr.net/gh/ismael221/BoltJX@main/jx.js"></script>   <!-- Homologação -->
<script src="https://cdn.jsdelivr.net/gh/ismael221/BoltJX@main/jx.min.js"></script> <!-- Produção -->
```

---

## Uso e Exemplos

### Banco de Dados

> Os métodos abaixo dependem do **Addon de Consultas da Bolt** para funcionar.

- **consultar(query)**: Realiza consultas SQL via addon. Retorna uma promessa com os resultados da consulta.
```javascript
/* Consulta ao banco com resposta formatada em JS */
JX.consultar('SELECT * FROM TGFMAR').then(console.log);
```

- **salvar(dados, instancia, chavesPrimarias)**: Salva registros no banco com a service auxiliar (`CRUDServiceProvider.saveRecord`). Aceita um objeto com os dados, o nome da tabela e as chaves primárias.
```javascript
/* Criar múltiplos NOVOS registros (deixar as chaves primárias vazias) */
JX.salvar({ DESCRICAO: 'Qualquer Marca' }, 'MarcaProduto', [{}, {}, {}]).then(console.log);

/* Atualizar múltiplos registros (informar as chaves primárias de cada registro) */
/* Repare que, intencionalmente, estou forçando o erro em apenas um dos salvamentos, */
/* mas por não ser blocante, ele continuará realizando os outros salvamentos com sucesso */
JX.salvar({ DESCRICAO: 'Outro produto' }, 'MarcaProduto', [{ CODIGO: 'asd' }, { CODIGO: 9998 }, { CODIGO: 9999 }]).then(console.log);
```

- **novoSalvar(dados, instancia, chavesPrimarias)**: (BETA) Salva registros no banco com a service oficial das telas nativas (`DatasetSP.save`). Aceita um objeto com os dados, o nome da tabela e as chaves primárias.
```javascript
/* Criar um NOVO registro */
JX.novoSalvar({ DESCRICAO: 'Qualquer Marca' }, 'MarcaProduto').then(console.log);

/* Atualizar um registro existente (informar as chaves primárias) */
JX.novoSalvar({ DESCRICAO: 'Outro produto' }, 'MarcaProduto', { CODIGO: 9999 }).then(console.log);
```

- **deletar(instancia, chavesPrimarias)**: Deleta registros. Requer o nome da tabela e as chaves primárias dos registros a serem excluídos.
```javascript
/* Apaga múltiplos registros (informar as chaves primárias de cada registro) */
/* O primeiro registro não existe (PK 9997), o que gerará um erro nessa requisição, */
/* mas por não ser blocante, ele continuará realizando as outras deleções com sucesso */
JX.deletar('MarcaProduto', [{ CODIGO: 9997 }, { CODIGO: 9998 }, { CODIGO: 9999 }]).then(console.log);
```

### Manipulação de Página

- **acionarBotao(parametros, configuracoes)**: Aciona botões de ação remotamente.
```javascript
JX.acionarBotao(
    {
        PARAMETRO_A: 'Valor',
        Parametro_B: 'false',    // Enviar valores booleanos como string
        pARameTRo_c: 2           // Validar o nome do parâmetro a ser recebido
    },
    {
        tipo         : 'JS',         // Tipo do botão de ação (JS, JAVA e SQL)
        idBotao      : 30,           // ID do botão de ação (JS, JAVA e SQL)
        entidade     : 'TELA_TAL',   // Nome da Entidade que possui o botão de ação (apenas SQL)
        nomeProcedure: 'AD_PROC_TAL' // Nome da Procedure a ser executada (apenas SQL)
    }
).then(console.log);
```

- **removerFrame(configuracoes)**: Remove o frame de uma página de BI.
```javascript
JX.removerFrame({ instancia: 'TELA_HTML5', paginaInicial: 'paginas/entidade/index.jsp' });
```

- **novaGuia(forcado)**: Abre a página atual em uma nova aba.
```javascript
JX.novaGuia();
```

- **abrirPagina(resourceID, chavesPrimarias)**: Abre uma página específica dentro do sistema.
```javascript
JX.abrirPagina('br.com.sankhya.core.cad.marcas', { CODIGO: 999 });
```

- **fecharPagina()**: Fecha a página atual.
```javascript
JX.fecharPagina();
```

### Retorno de Valores

- **getUrl(path)**: Retorna a URL atual da página, permitindo adicionar um caminho específico.
```javascript
console.log(JX.getUrl());                                       // http://localhost/mge
console.log(JX.getUrl('js/dashboardGrid/dashboardGrid.css'));   // http://localhost/mge/js/dashboardGrid/dashboardGrid.css
```

- **getCookie(nome)**: Retorna o valor de um cookie pelo nome.
```javascript
let valorCookie = JX.getCookie('nomeCookie');
console.log(valorCookie);
```

- **getArquivo(caminhoArquivo)**: Busca o conteúdo de um arquivo no caminho especificado.
```javascript
JX.getArquivo('/caminho/do/arquivo.txt')
   .then(conteudo => console.log(conteudo))
   .catch(erro => console.error(erro));
```

- **getParametro(nomesParametros)**: Retorna valores de parâmetros específicos.
```javascript
JX.getParametro(['PERCSTCAT137SP', 'BASESNKPADRAO']).then(console.log);
JX.getParametro('123').then(console.log);
JX.getParametro().then(console.log); // Retorna todos os parâmetros
```

### Chamada de Serviço

- **chamarServico(nomeServico, dados, dadosAdicionais)**: Permite a chamada de serviços web do Sankhya.
```javascript
JX.chamarServico("mgecom@admin.getVersao").then(console.log);
JX.chamarServico('WorkspaceSP.getStartupData', '<serviceRequest serviceName="WorkspaceSP.getStartupData"><requestBody><resourceIDs/><clientEventList/></requestBody></serviceRequest>');
```

---

## Considerações

- O BoltJX foi criado para ser usado **em conjunto com o Addon de Consultas da Bolt**, não como biblioteca standalone para consultas diretas ao banco.
- É a solução recomendada para ambientes com restrições de segurança que impedem a liberação do dbExplorer nativo aos usuários.
- Muitos métodos são assíncronos e retornam `Promises` — utilize `.then()` ou `async/await`.
- Implemente tratamento de erros para garantir a robustez da aplicação.
