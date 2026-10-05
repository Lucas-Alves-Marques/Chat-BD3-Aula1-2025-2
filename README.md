# Chat 💬

Aplicacao web de chat em tempo real desenvolvida com Node.js, Express, Socket.IO e MongoDB.

## Contexto 🧭

Este repositorio foi utilizado como meio para armazenar o codigo-base do projeto **Chat_Mongo_DB**. Ele contem uma implementacao de estudo de um chat simples, com uma interface web, troca de mensagens entre usuarios conectados e persistencia das mensagens no MongoDB.

## Funcionalidades ✨

- Exibe uma pagina de chat servida pelo Express.
- Permite informar um nome de usuario e escrever uma mensagem.
- Atualiza a conversa em tempo real usando Socket.IO.
- Armazena mensagens no MongoDB por meio do Mongoose.
- Carrega mensagens existentes quando o servidor inicia e as envia a cada novo usuario conectado.

## Tecnologias e componentes ⚙️

| Componente | Responsabilidade |
| --- | --- |
| Node.js | Executa o servidor JavaScript. |
| Express | Serve os arquivos estaticos e a pagina HTML. |
| HTTP | Cria o servidor usado pelo Express e pelo Socket.IO. |
| Socket.IO | Mantem a conexao em tempo real entre navegador e servidor. |
| Mongoose | Define o modelo das mensagens e faz a comunicacao com o MongoDB. |
| EJS | Configurado como mecanismo para renderizar o arquivo `index.html`. |
| jQuery | Le os campos do formulario e atualiza a lista de mensagens na pagina. |
| Nodemon | Reinicia o servidor automaticamente durante o desenvolvimento. |

## Estrutura do projeto 📁

```text
.
├── index.js            # Inicializacao do servidor, banco e eventos do chat
├── package.json        # Dependencias e comando de inicializacao
├── package-lock.json   # Versoes resolvidas das dependencias
└── public/
    ├── index.html      # Interface, formulario e cliente Socket.IO
    └── styles.css      # Estilos da interface
```

## Requisitos ✅

- Node.js e npm instalados.
- Uma instancia MongoDB acessivel. O codigo foi preparado para conectar ao MongoDB Atlas; tambem e possivel usar outra instancia MongoDB se a URI for ajustada.
- Para Atlas, um cluster, um usuario de banco de dados e uma regra de acesso de rede que permita a conexao da maquina que executa o projeto.
- Acesso a internet para carregar as bibliotecas jQuery e Socket.IO referenciadas por CDN no HTML.

## Configuracao do MongoDB 🗄️

Antes de iniciar o servidor, configure em `index.js`, dentro da funcao `connectDB()`, uma URI valida do MongoDB para `dbURL`. O valor presente no repositorio esta mascarado e nao e uma URI utilizavel.

Uma URI Atlas geralmente tem este formato:

```text
mongodb+srv://<usuario>:<senha>@<cluster>/<banco>?retryWrites=true&w=majority
```

Substitua os campos pelos dados do seu proprio cluster. Se usuario ou senha tiverem caracteres especiais, eles devem ser codificados conforme o formato de URI. Nao publique credenciais nem envie a URI real para o repositorio. Para um uso mais seguro, prefira obter a URI de uma variavel de ambiente em vez de mante-la no codigo-fonte.

O modelo Mongoose usado atualmente e `Message`, com estes campos:

| Campo | Tipo | Finalidade |
| --- | --- | --- |
| `usuario` | String | Nome informado no campo de usuario. |
| `data_hora` | String | Data e hora formatadas pelo navegador no momento do envio. |
| `message` | String | Conteudo da mensagem. |

O nome da colecao e derivado do modelo pelo Mongoose (normalmente `messages`).

## Instalacao e execucao 🚀

No terminal, entre na pasta do repositorio e instale as dependencias:

```bash
npm install
```

Depois de configurar a URI do MongoDB, inicie o servidor:

```bash
npm start
```

O script `start` executa `nodemon index.js`. Quando a inicializacao estiver concluida, acesse:

```text
http://localhost:3000
```

Abra a pagina em duas ou mais abas ou navegadores para verificar a troca de mensagens entre clientes. Para encerrar o servidor, pressione `Ctrl+C` no terminal.

> Nao abra `public/index.html` diretamente pelo sistema de arquivos. A pagina depende do servidor Express e da conexao Socket.IO disponibilizada pela aplicacao.

## Como a aplicacao funciona 🔄

1. `index.js` cria a aplicacao Express, um servidor HTTP e uma instancia Socket.IO.
2. O Express disponibiliza os arquivos da pasta `public` e a rota `/` tenta renderizar `index.html`.
3. Ao iniciar, o servidor tenta conectar ao MongoDB e consulta os documentos do modelo `Message` para preencher o array em memoria `messages`.
4. Quando um navegador conecta ao Socket.IO, o servidor envia o array carregado usando o evento `previousMessage`.
5. Ao enviar o formulario, o navegador monta um objeto com `usuario`, `data_hora` e `message`, renderiza a mensagem localmente e envia o objeto com o evento `sendMessage`.
6. O servidor cria um documento Mongoose e o salva no MongoDB. Depois do salvamento, envia `receivedMessage` aos demais clientes conectados. O cliente que enviou a mensagem ja a exibe localmente.

### Eventos Socket.IO 📡

| Evento | Direcao | Uso |
| --- | --- | --- |
| `previousMessage` | Servidor -> cliente | Envia as mensagens carregadas quando o servidor iniciou. |
| `sendMessage` | Cliente -> servidor | Solicita que uma nova mensagem seja salva. |
| `receivedMessage` | Servidor -> demais clientes | Distribui uma mensagem depois do salvamento no banco. |

## Observacoes sobre a implementacao atual ⚠️

- A configuracao da pasta de views usa `app.set('view', ...)`, mas o Express espera a opcao `views` (no plural). Com o codigo como esta, a rota `/` pode falhar ao procurar `index.html` na pasta padrao `views`. Para a pagina funcionar, ajuste essa linha em `index.js` para `app.set('views', path.join(__dirname, 'public'));`.
- A consulta inicial de mensagens ocorre uma vez durante a inicializacao do processo. Mensagens salvas depois desse carregamento nao sao acrescentadas ao array em memoria; portanto, novos clientes recebem a lista carregada no inicio, nao necessariamente o historico mais recente. As novas mensagens sao, ainda assim, persistidas no MongoDB e transmitidas aos clientes ja conectados.
- O cliente Socket.IO esta configurado com o endereco `http://localhost:3000`. Para hospedar a aplicacao em outro endereco, sera necessario ajustar essa configuracao.
- O projeto e uma base simples de estudo: nao possui autenticacao, validacao de mensagens ou gerenciamento de salas.
- A interface monta o conteudo das mensagens diretamente no HTML. Nao use esta versao com entradas publicas ou nao confiaveis sem adicionar validacao e renderizacao segura.
- A conexao ao banco e iniciada no codigo com uma URI; o projeto nao carrega configuracoes de um arquivo `.env` atualmente.

## Solucao de problemas 🛠️

### O servidor inicia, mas nao conecta ao MongoDB 🧪

- Confirme que substituiu o valor mascarado de `dbURL` por uma URI valida.
- Verifique usuario, senha, nome do cluster e nome do banco na URI.
- No Atlas, confira se o IP da maquina esta permitido na configuracao de acesso de rede.
- Consulte as mensagens de erro exibidas no terminal.

### A pagina nao abre em `localhost:3000` 🌐

- Confirme que `npm start` continua em execucao e que o terminal informa que o servidor esta online na porta 3000.
- Se a requisicao falhar com erro de template/view, aplique a correcao descrita acima: troque a opcao `view` por `views` na configuracao do Express.
- Verifique se outro processo ja esta usando essa porta.
- Acesse a aplicacao por `http://localhost:3000`, nao abrindo o HTML diretamente.

### A interface abre, mas nao recebe mensagens em tempo real 📡

- Confira se o servidor permanece ativo e se o cliente esta acessando o mesmo host/porta configurado no HTML.
- Verifique se o navegador tem acesso a internet para carregar os scripts externos usados pela pagina.
- Consulte o console do navegador e o terminal do servidor em busca de erros.
