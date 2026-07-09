# pos-IA

Este repositorio contem exemplos de uso de IA em aplicacoes web.

O exemplo principal atualmente esta em `exemplo03-webai01/index.html` e demonstra o uso da Prompt API do Chrome com Gemini Nano, executando o modelo localmente no navegador.

## Exemplo: Prompt API com Gemini Nano

Arquivo:

```text
exemplo03-webai01/index.html
```

Este HTML cria uma interface simples para enviar perguntas ao modelo local Gemini Nano usando a API experimental `LanguageModel`, disponivel em versoes compativeis do Google Chrome.

## Recursos da tela

- Campo de texto para o usuario digitar uma pergunta ou instrucao.
- Upload opcional de imagem.
- Upload opcional de audio.
- Botao `Gerar resposta` para executar a consulta.
- Botao `Limpar` para remover texto, anexos, resposta e estado da tela.
- Area de status para mostrar as etapas da execucao.
- Area de resposta com renderizacao Markdown quando a biblioteca externa estiver disponivel.
- Tratamento de erro visivel na tela.
- Indicacao de progresso quando o modelo precisa ser baixado.

## O que o exemplo faz

O fluxo principal e:

1. O usuario digita uma pergunta ou instrucao.
2. Opcionalmente, seleciona uma imagem e/ou um audio.
3. Ao clicar em `Gerar resposta`, o codigo verifica se a API `LanguageModel` esta disponivel no navegador.
4. O codigo monta as opcoes do modelo de acordo com os tipos de entrada selecionados:
   - texto;
   - imagem;
   - audio.
5. O codigo chama `LanguageModel.availability(modelOptions)` para verificar se o modelo consegue atender aquela combinacao de entradas.
6. Se o modelo ainda precisar ser baixado, a tela mostra o progresso.
7. O codigo cria uma sessao com `LanguageModel.create(...)`.
8. O prompt e enviado com `session.promptStreaming(...)`.
9. A resposta e apresentada progressivamente na tela.
10. Ao final, a sessao e destruida com `session.destroy()`.

## Como funciona

### Verificacao da API

Antes de usar o modelo, o arquivo verifica se `LanguageModel` existe em `window`.

Isso evita erros silenciosos quando:

- o Chrome nao suporta a API;
- as flags experimentais nao estao habilitadas;
- a pagina nao esta sendo aberta por uma origem aceita, como `localhost`.

### Verificacao de disponibilidade

O exemplo usa:

```js
const availability = await LanguageModel.availability(modelOptions);
```

As mesmas opcoes usadas para criar a sessao tambem sao usadas na verificacao de disponibilidade. Isso e importante porque texto, imagem e audio podem ter requisitos diferentes.

### Criacao da sessao

A sessao e criada com:

```js
const session = await LanguageModel.create({
    ...modelOptions,
    initialPrompts: [
        {
            role: "system",
            content: "Voce e um assistente de IA que analisa textos, imagens e audios de forma clara e objetiva."
        }
    ],
    monitor(monitor) {
        monitor.addEventListener("downloadprogress", (event) => {
            if (availability === "downloadable" || availability === "downloading") {
                status.textContent = `Baixando modelo: ${Math.round(event.loaded * 100)}%`;
            }
        });
    }
});
```

O `monitor` acompanha o download do modelo quando necessario.

### Prompt somente texto

Quando nao ha anexos, o prompt enviado e uma string simples:

```js
return question;
```

### Prompt com imagem e/ou audio

Quando ha anexos, o prompt e enviado como uma lista de mensagens com conteudo multimodal:

```js
return [
    {
        role: "user",
        content: [
            {
                type: "text",
                value: question
            },
            {
                type: "image",
                value: imageFile
            },
            {
                type: "audio",
                value: audioFile
            }
        ]
    }
];
```

O codigo adiciona `image` e `audio` apenas quando o usuario seleciona esses arquivos.

### Resposta em streaming

A resposta e consumida com:

```js
const responseStream = session.promptStreaming(promptPayload);
```

Cada trecho recebido e acumulado e exibido na tela.

## Como executar

Nao abra o arquivo diretamente com `file:///`.

Execute um servidor local na pasta do exemplo:

```bash
cd exemplo03-webai01
py -3 -m http.server 8000 --bind 127.0.0.1
```

Depois acesse no Chrome:

```text
http://127.0.0.1:8000/index.html
```

## Configuracao necessaria no Chrome

Como a Prompt API ainda e experimental, verifique as flags:

```text
chrome://flags/#prompt-api-for-gemini-nano
chrome://flags/#optimization-guide-on-device-model
```

Depois de alterar flags, reinicie o Chrome.

Tambem e util verificar o estado do modelo em:

```text
chrome://on-device-internals
```

## Execucao local e internet

O Gemini Nano e executado localmente no navegador depois que o modelo foi baixado.

Na primeira execucao, pode ser necessario acesso a internet para baixar o modelo. Depois disso, a inferencia deve funcionar localmente, sem enviar o prompt para Google ou terceiros.

Observacao: o HTML carrega a biblioteca Markdown por CDN:

```html
<script src="https://cdn.jsdelivr.net/npm/markdown@0.5.0/lib/markdown.min.js"></script>
```

Se a internet estiver indisponivel, a resposta ainda aparece como texto puro por causa do fallback implementado no codigo.

## Limitacoes conhecidas

- A API `LanguageModel` e experimental e pode mudar entre versoes do Chrome.
- A analise de audio pode exigir GPU compativel.
- Nem todos os idiomas ou modalidades podem estar disponiveis em todos os ambientes.
- O Chrome pode remover o modelo local se houver pouco espaco em disco.
- Abrir por `file:///` pode impedir ou limitar o funcionamento esperado.
- Usar outro perfil do Chrome, outra porta, outra origem ou outro computador pode exigir nova preparacao do modelo.

## Como implementar em outro HTML

Para reutilizar a ideia em outro projeto:

1. Sirva a pagina por `localhost` ou por uma origem segura.
2. Verifique se `LanguageModel` existe em `window`.
3. Monte `expectedInputs` conforme os tipos de entrada que serao enviados.
4. Chame `LanguageModel.availability(modelOptions)` antes de criar a sessao.
5. Crie a sessao com `LanguageModel.create(...)`.
6. Envie o prompt com `promptStreaming(...)` para exibir respostas progressivamente.
7. Trate erros com `try/catch`.
8. Destrua a sessao ao final com `session.destroy()`.
9. Informe o usuario quando o modelo estiver baixando ou indisponivel.

## Estrutura atual

```text
.
+-- README.md
`-- exemplo03-webai01
    `-- index.html
```
