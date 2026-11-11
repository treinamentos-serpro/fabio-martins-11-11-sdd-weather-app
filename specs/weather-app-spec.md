# Especificação de Produto — Aplicação de Previsão do Tempo

- **Versão:** 0.3 (decisões P1 adotadas; pendências externas em [Open Questions](#10-open-questions))
- **Data:** 2026-11-11
- **Fonte:** [discovery.md](./discovery.md)
- **Status:** decisões P1 confirmadas pelo time. Pendências externas: OQ-03, OQ-07, OQ-21, OQ-26 (parcial) e OQ-29.

> Esta spec é a fonte única da verdade para Plan, Tasks, Code e Test. Não contém
> detalhes de implementação. Stack e arquitetura pertencem ao Plan.

## 1. Overview

### 1.1 Problema

Pessoas que querem saber o clima de uma cidade precisam de uma forma rápida e
acessível de consultar o tempo atual e a previsão dos próximos dias, com
preferência de unidade de temperatura, principalmente pelo celular.

### 1.2 Objetivo do produto

Permitir que qualquer usuário:

1. Busque uma cidade pelo nome.
2. Veja o clima atual dessa cidade.
3. Veja a previsão de 5 dias, começando hoje (hoje mais os 4 dias seguintes, conforme D-02).
4. Alterne entre Celsius e Fahrenheit.

### 1.3 Escopo da versão

Aplicação web de página única, responsiva, em pt-BR, sem contas de usuário,
sem backend próprio e sem chave de API. Os dados vêm da Open-Meteo (geocodificação
e previsão). Ver [Out of Scope](#9-out-of-scope).

### 1.4 Decisões de base

As decisões D-01 a D-05 do discovery são tratadas como fechadas:

| ID   | Decisão                                                      |
| ---- | ------------------------------------------------------------ |
| D-01 | Fonte de dados é a Open-Meteo, sem API key                   |
| D-02 | "5 dias" significa hoje mais 4 dias                          |
| D-03 | Unidade padrão é Celsius                                     |
| D-04 | Sem autenticação e sem persistência de dados no servidor     |
| D-05 | Idioma da interface é pt-BR                                  |

### 1.5 Personas

Personas hipotéticas, a validar com a empresa (ver OQ-07).

- **Mariana, viajante de fim de semana:** usa o celular em trânsito, com conexão
  instável. Quer saber se vai chover no destino em menos de 15 segundos.
- **Carlos, morador do dia a dia:** consulta o tempo toda manhã pelo celular.
  Quer ver temperatura e sensação térmica rapidamente.
- **Juliana, usuária de desktop e leitor de tela:** compara previsões de duas
  cidades no computador, usa só teclado e leitor de tela e lê valores em °F.

## 2. Functional Requirements

Cada requisito tem ID estável (RF-XX) e pelo menos um critério de aceite na
seção [4. Acceptance Criteria](#4-acceptance-criteria).

| ID    | Requisito                                                                 | Prioridade | Status                                       |
| ----- | ------------------------------------------------------------------------- | ---------- | -------------------------------------------- |
| RF-01 | O sistema permite buscar cidades pelo nome e lista os resultados correspondentes | Must       | Definido                                     |
| RF-02 | O sistema carrega o clima da cidade escolhida no resultado da busca       | Must       | Definido                                     |
| RF-03 | O sistema exibe o clima atual da cidade selecionada                       | Must       | Definido (OQ-02, OQ-09)                      |
| RF-04 | O sistema exibe a previsão de 5 dias, incluindo hoje, com probabilidade de chuva | Must | Definido (OQ-10)                             |
| RF-05 | O usuário alterna entre Celsius e Fahrenheit e a escolha persiste no navegador | Must | Definido (OQ-12)                             |
| RF-06 | O sistema exibe estados de loading, erro e vazio em cada operação         | Must       | Definido; textos em OQ-05                    |
| RF-07 | O sistema informa a origem dos dados (Open-Meteo) na tela de clima        | Should     | Proposto pelo discovery (R9); aviso em OQ-21 |
| RF-08 | O sistema informa o horário da última atualização do clima exibido        | Should     | Proposto pelo discovery (R7); depende de OQ-19 |
| RF-09 | O sistema restaura a última cidade pesquisada ao abrir a aplicação        | Should     | Definido (OQ-13)                             |

### 2.1 Regras funcionais gerais

- **RG-01:** a busca só é disparada com texto de pelo menos 2 caracteres, sem contar espaços das bordas (AC-01.1, AC-01.10). Espaços em branco (
  contam como vazio).
- **RG-02:** a busca é disparada após pausa na digitação (debounce). Ver AC-01.9.
- **RG-03:** ao escolher uma cidade, o clima anterior é substituído pelo novo
  ou pelo estado de loading. Dados de duas cidades nunca aparecem misturados.
- **RG-04:** se uma resposta chegar fora de ordem, a resposta da última ação do
  usuário é a que vale. Respostas antigas são descartadas.
- **RG-05:** toda temperatura exibida respeita a unidade ativa. Não pode haver
  temperatura em °C dentro de um bloco em °F.

### 2.2 Parâmetros de comportamento

Valores usados diretamente pelos testes. "Fechado" vem do discovery ou de decisão
do usuário; "Decidido" foi adotado nesta spec e pode ser revertido com registro na seção 10.

| Parâmetro | Valor | Status |
| --- | --- | --- |
| Tamanho mínimo da busca (sem espaços das bordas) | 2 caracteres | Decidido (AC-01.10) |
| Tamanho máximo da busca | 255 caracteres | Decidido (AC-01.8) |
| Pausa de digitação antes da busca | 300 ms | Decidido (AC-01.9) |
| Resultados máximos da geocodificação | 10 | Decidido (OQ-17, AC-01.6) |
| Tempo limite por requisição | 5 s | Decidido (NFR-03, OQ-27) |
| Lista de resultados visível após a pausa | até 1 s | Decidido (NFR-02, OQ-27) |
| Dias de previsão | 5, começando hoje | Fechado (D-02) |
| Unidade padrão | °C | Fechado (D-03) |
| Casas decimais exibidas | 0 (inteiro) | Decidido (2.3) |
| Faixa válida de temperatura | -90 °C a 60 °C | Decidido (AC-03.4) |
| Faixa válida de umidade | 0 % a 100 % | Decidido (AC-03.4) |
| Faixa válida de vento | maior ou igual a 0 | Decidido (AC-03.4) |
| Retentativa automática | nenhuma | Decidido (NFR-05, OQ-20) |
| Largura mínima de viewport | 320 px | Fechado (NFR-11) |

### 2.3 Formatação

- Temperatura: número inteiro, espaço e unidade, por exemplo "32 °F".
- Arredondamento: ao inteiro mais próximo; em empate, para cima (0,5 → 1; -0,5 → 0).
- Conversão para °F parte do valor original em °C, sem arredondamento intermediário (AC-05.6).
- Valor que arredonda para zero aparece como "0", sem sinal (AC-05.7).
- Data: "dd/mm/aaaa". Hora: "HH:mm", em 24 h, no fuso da cidade (OQ-19).
- Vento: km/h. Pressão não é exibida (OQ-11).

### 2.4 Máquina de estados

| Estado atual | Evento | Próximo estado |
| --- | --- | --- |
| Ocioso | texto válido após a pausa de digitação | Buscando |
| Ocioso | texto vazio ou com 1 caractere | Ocioso, sem requisição (AC-01.1, AC-01.10) |
| Buscando | lista com ao menos 1 item | Resultados |
| Buscando | lista vazia | Sem resultados |
| Buscando | falha, timeout ou sem rede | Erro de busca |
| Resultados | escolha de uma cidade | Carregando clima |
| Carregando clima | clima atual e previsão válidos | Clima exibido |
| Carregando clima | só a previsão falha | Clima com erro de previsão (AC-06.6) |
| Carregando clima | clima atual falha | Erro de clima |
| Erro de busca, erro de clima ou erro de previsão | "Tentar novamente" | Repete só a requisição que falhou (AC-06.9) |
| Qualquer estado | nova busca ou nova cidade | Buscando ou Carregando clima; respostas anteriores são descartadas (RG-04) |
| Clima exibido | campo de busca apagado | Clima exibido, sem alteração (RG-01) |

## 3. User Stories

Formato: "Como [persona], quero [ação] para [valor]".

| ID    | História                                                                                                                                | Requisitos |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| US-01 | Como Mariana, quero buscar uma cidade pelo nome para ver o clima do meu destino.                                                        | RF-01, RF-02 |
| US-02 | Como Carlos, quero ver a temperatura atual e a sensação térmica para decidir se levo casaco ou guarda-chuva.                            | RF-03      |
| US-03 | Como Mariana, quero ver a previsão de 5 dias com mínima, máxima e condição de cada dia para saber se vai chover na viagem.              | RF-04      |
| US-04 | Como Carlos, quero trocar entre °C e °F para ler a temperatura na unidade que eu uso.                                                    | RF-05      |
| US-05 | Como Juliana, quero trocar a unidade e manter essa escolha ao buscar outra cidade para comparar cidades sem refazer a configuração.      | RF-05      |
| US-06 | Como Carlos, quero ver um aviso de carregamento, de erro ou de cidade não encontrada para saber se o app ainda está buscando o clima ou se algo falhou.                                                                   | RF-06      |
| US-07 | Como Mariana, quero tentar de novo quando a consulta falhar, sem recarregar a página, para não perder tempo com sinal instável no transporte.                                                                              | RF-06      |
| US-08 | Como Juliana, quero que os resultados e os estados sejam anunciados pelo leitor de tela para acompanhar a busca sem ver a tela.          | RF-01, RF-06 |
| US-09 | Como Juliana, quero concluir busca, troca de unidade e leitura da previsão só com teclado para usar o app sem mouse.                      | RF-01, RF-05 |
| US-10 | Como Mariana, quero escolher a cidade certa entre homônimos, vendo país e região, para não consultar o lugar errado.                     | RF-01      |
| US-11 | Como Carlos, quero saber de onde vêm os dados e quando foram atualizados para confiar na previsão que define minha roupa do dia.        | RF-07, RF-08 |
| US-12 | Como Carlos, quero que o app reabra com a última cidade que pesquisei para não repetir a busca todo dia. | RF-09 |

## 4. Acceptance Criteria

Critérios no formato Given/When/Then (Dado/Quando/Então). Cada critério usa valores
objetivos (textos, contagens, dimensões, status de resposta) e serve de base direta
para os testes automatizados (unit com Vitest e E2E com Playwright). Critérios
que dependem de uma Open Question indicam a questão entre parênteses.

### Textos da interface

Textos referenciados pelos critérios. Qualquer mudança de texto passa por esta tabela.

| Chave | Texto | Status |
| --- | --- | --- |
| Carregando | Carregando | Decidido (OQ-05) |
| Mínimo de caracteres | Digite pelo menos 2 caracteres. | Decidido (AC-01.10) |
| Sem resultados | Nenhuma cidade encontrada | Decidido (OQ-05, AC-06.3) |
| Sugestão de revisão | Revise o nome da cidade | Decidido (OQ-05, AC-06.3) |
| Erro de busca ou de clima | Não foi possível carregar os dados. | Decidido (OQ-05) |
| Repetir | Tentar novamente | Decidido (OQ-05) |
| Campo indisponível | Indisponível | Decidido (AC-03.3, AC-04.3, AC-08.1) |
| Atribuição | Dados meteorológicos: Open-Meteo | Provisório até texto jurídico (OQ-21) |

### AC — RF-01: Buscar cidades por nome

- **AC-01.1** Given o campo de busca contém vazio ou apenas espaços, When o usuário envia a busca (Enter ou pausa de digitação), Then nenhuma requisição de geocodificação é feita e nenhum item de resultado é renderizado.
- **AC-01.2** Given a geocodificação responde com ao menos uma cidade para a busca "Londres", When a lista é renderizada, Then cada item exibe nome e país não vazios; a região aparece ou, se a API não a fornecer, exibe "Indisponível" (NFR-04).
- **AC-01.3** Given a geocodificação responde com lista vazia para a busca "xyzabc", When a busca termina, Then o estado vazio de AC-06.3 é renderizado e zero itens aparecem.
- **AC-01.4** Given a geocodificação retorna "São Paulo" para a busca "Sao Paulo", When a lista é renderizada, Then existe um item com o nome "São Paulo" (depende de OQ-25).
- **AC-01.5** Given a resposta contém dois itens com o mesmo nome e países diferentes, When a lista é renderizada, Then os dois itens exibem valores de país diferentes.
- **AC-01.6** Given a resposta contém N itens, em que N é maior que 10, When a lista é renderizada, Then exatamente 10 itens são renderizados, na ordem devolvida pela API (OQ-17).
- **AC-01.7** Given a geocodificação responde em até 300 ms (pré-condição de teste, mock), When o intervalo entre o fim da pausa de digitação e a renderização da lista é medido, Then esse intervalo é de até 1 s (NFR-02).
- **AC-01.8** Given o campo de busca tem 255 caracteres, When o usuário digita mais um caractere, Then o campo não aceita o 256º caractere.
- **AC-01.9** Given o usuário digita vários caracteres em sequência, When a pausa de digitação termina, Then exatamente uma requisição de geocodificação é disparada, com o último texto digitado.
- **AC-01.10** Given o texto de busca tem 1 caractere (sem contar espaços das bordas), When a pausa de digitação termina, Then nenhuma requisição é feita, a mensagem "Digite pelo menos 2 caracteres." aparece e nenhum estado de erro é exibido.

### AC — RF-02: Carregar clima da cidade escolhida

- **AC-02.1** Given a lista contém o item "Londres, Reino Unido", When o usuário clica no item ou pressiona Enter com o foco nele, Then exatamente uma requisição de clima atual e uma de previsão são disparadas com as coordenadas desse item.
- **AC-02.2** Given "Londres" foi escolhida, When o clima chega, Then o título da seção de clima exibe "Londres".
- **AC-02.3** Given o usuário escolheu a cidade A e depois a cidade B, When a resposta de A chega após a de B, Then o título exibe "B", nenhum dado de A é renderizado e a resposta de A é descartada (RG-04).
- **AC-02.4** Given a busca ou o carregamento da cidade A está em andamento, When o usuário escolhe a cidade B, Then as requisições de A são canceladas e nenhuma resposta de A altera a tela (EC-10).

### AC — RF-03: Exibir clima atual

- **AC-03.1** Given uma resposta válida de clima atual, When o clima é renderizado, Then existem os rótulos "Temperatura", "Sensação térmica", "Condição", "Umidade" e "Vento", cada um com valor não vazio (campos definidos em OQ-09).
- **AC-03.2** Given a resposta indica a condição "parcialmente nublado", When o clima é renderizado, Then o texto da condição é "Parcialmente nublado" e o ícone tem atributo de texto alternativo não vazio.
- **AC-03.3** Given a resposta não contém o campo de temperatura, When o clima é processado, Then o campo de temperatura exibe "Indisponível", os demais campos disponíveis são exibidos e nenhum elemento contém "NaN", "undefined" ou valor vazio.
- **AC-03.4** Given a resposta traz temperatura de 200 °C ou de -95 °C (fora de -90 °C a 60 °C), When o clima é processado, Then o campo de temperatura exibe "Indisponível" e o valor inválido não aparece em nenhum elemento.

### AC — RF-04: Exibir previsão de 5 dias

- **AC-04.1** Given uma previsão com 5 dias válidos, When a seção é renderizada, Then existem exatamente 5 cartões e o primeiro tem a data de hoje no fuso de referência (OQ-19).
- **AC-04.2** Given um cartão de dia, When renderizado, Then contém data, temperatura mínima, temperatura máxima e condição, e nenhum deles está vazio, salvo os campos marcados como "Indisponível" (AC-04.3).
- **AC-04.3** Given um dia da resposta tem mínima maior que máxima, When a previsão é processada, Then o cartão desse dia exibe "Indisponível" no lugar das temperaturas, os demais dias válidos são renderizados e nenhum valor inválido aparece.
- **AC-04.4** Given a previsão tem menos de 5 dias válidos, When a seção é renderizada, Then os dias válidos são exibidos, nenhum dia é inventado e o clima atual permanece visível.
- **AC-04.5** Given a API devolve mais de 5 dias, When a seção é renderizada, Then são exibidos apenas hoje e os 4 dias seguintes.
- **AC-04.6** Given um cartão de dia, When renderizado, Then exibe a probabilidade de chuva do dia em número inteiro de 0 % a 100 %; se o valor estiver ausente ou fora da faixa, exibe "Indisponível" (NFR-04).

### AC — RF-05: Alternar Celsius e Fahrenheit

- **AC-05.1** Given a aplicação é aberta sem preferência salva, When o clima é renderizado, Then todas as temperaturas terminam com "°C" e nenhuma termina com "°F" (D-03).
- **AC-05.2** Given o clima está em °C com temperatura atual de 0 °C e máxima de um dia de 20 °C, When o usuário seleciona °F, Then a temperatura atual exibe "32 °F", a máxima exibe "68 °F" e nenhuma requisição à API é feita.
- **AC-05.3** Given a unidade ativa é °F, When o usuário busca outra cidade e o clima chega, Then todas as temperaturas dessa cidade terminam com "°F".
- **AC-05.4** Given a temperatura original da API é -40 °C, When é exibida em °F, Then aparece "-40 °F"; e a conversão usa sempre o valor original em °C, sem conversões encadeadas (R5).
- **AC-05.6** Given a temperatura original da API é 37,5 °C e a unidade ativa é °F, When é exibida, Then aparece "100 °F" (99,5 °F arredondado para cima), sem usar valor já arredondado em °C.
- **AC-05.7** Given a temperatura original da API é -0,4 °C, When é exibida em °C, Then aparece "0 °C", sem sinal de menos.
- **AC-05.8** Given o usuário escolheu °F e recarrega a aplicação, When a aplicação abre, Then a unidade ativa é °F e o clima exibido já aparece em °F (OQ-12).
- **AC-05.5** Given o foco está no controle de unidade, When o usuário pressiona Enter ou Espaço, Then a unidade muda sem uso de mouse e o controle expõe o estado ativo para leitor de tela com `aria-pressed`.

### AC — RF-06: Estados de loading, erro e vazio

- **AC-06.1** Given uma busca ou carregamento está pendente (requisição sem resposta), When a tela é renderizada, Then existe um elemento de loading com o texto "Carregando" e `aria-live` definido (NFR-09).
- **AC-06.2** Given a API responde com status 500, ou excede o timeout de 5 s (NFR-03), ou o dispositivo está sem rede, When a falha ocorre, Then a mensagem de erro e o botão "Tentar novamente" são exibidos na seção afetada e o campo de busca continua habilitado.
- **AC-06.3** Given a geocodificação retorna lista vazia, When a busca termina, Then o texto "Nenhuma cidade encontrada" e a sugestão "Revise o nome da cidade" (textos finais, OQ-05) são exibidos e nenhum item aparece.
- **AC-06.4** Given o estado de erro está exibido, When o usuário aciona "Tentar novamente" e a API responde com sucesso, Then a mensagem de erro desaparece, o clima aparece e houve exatamente uma nova requisição.
- **AC-06.5** Given o clima da cidade A está exibido, When a busca pela cidade B falha, Then nenhum título, temperatura ou previsão de A permanece visível.
- **AC-06.6** Given o clima atual da cidade B foi recebido e a previsão falha por timeout ou status 5xx, When a falha ocorre, Then o clima atual permanece visível, a mensagem de erro e "Tentar novamente" aparecem apenas na seção de previsão e o campo de busca continua habilitado.
- **AC-06.7** Given a API responde com limite de uso excedido, When a falha ocorre, Then a mensagem não exibe detalhes técnicos e o botão "Tentar novamente" aparece.
- **AC-06.8** Given o estado de erro ou o estado vazio é exibido, When a tela é renderizada, Then o elemento da mensagem tem `aria-live` definido (NFR-09).
- **AC-06.9** Given o clima atual foi exibido e a previsão falhou, When o usuário aciona "Tentar novamente", Then exatamente uma requisição de previsão é feita e a busca de cidades não é refeita.

### AC — RF-07: Origem dos dados

- **AC-07.1** Given o clima de uma cidade está exibido, When a região de clima é inspecionada, Then existe texto visível contendo "Open-Meteo" e o texto contém a atribuição aprovada em OQ-21. Enquanto OQ-21 estiver aberta, verifica-se apenas a presença de "Open-Meteo".

### AC — RF-08: Horário da última atualização

- **AC-08.1** Given o clima está exibido com horário de atualização da API, When a região de clima é inspecionada, Then existe um horário visível no formato definido em OQ-19, acompanhado do fuso. Se a API não informar o horário, o campo exibe "Indisponível" (NFR-04).

### AC — RF-09: Restaurar última cidade

- **AC-09.1** Given o usuário viu o clima de "Londres" e fechou a aplicação, When a aplicação é aberta novamente, Then o clima de "Londres" é carregado e o campo de busca está vazio (OQ-13).
- **AC-09.2** Given não há cidade salva ou o armazenamento do navegador está indisponível, When a aplicação abre, Then nenhuma requisição de clima é feita e nenhum erro é exibido.

### AC — Transversais (acessibilidade e responsividade)

- **AC-T.1** Given a página está carregada com clima exibido, When o usuário percorre os controles com Tab, Then busca, lista de resultados, escolha de cidade, unidade e "Tentar novamente" recebem foco com contorno de pelo menos 3:1 de contraste contra o fundo adjacente, e Enter ou Espaço os ativa (NFR-08).
- **AC-T.2** Given o viewport tem 320 px ou 428 px, When a página é renderizada em cada estado (loading, erro, vazio e com clima exibido), Then a largura de rolagem do documento é igual à largura visível (sem rolagem horizontal) (NFR-11).
- **AC-T.3** Given o viewport tem 320 px, When os controles interativos são medidos, Then cada alvo tem largura e altura de pelo menos 44 px (NFR-12).
- **AC-T.4** Given o teste emula toque sem hover, When cada função é acionada por toque ou clique, Then nenhuma função depende de tooltip ou de hover para ser executada (NFR-13).
- **AC-T.5** Given a busca é `<img src=x onerror=alert(1)>`, When a lista ou o estado vazio é renderizado, Then o texto aparece literal e nenhum elemento `img` é criado (NFR-16, EC-12).
- **AC-T.6** Given o tema dark está renderizado em loading, erro, vazio e foco, When o contraste do texto normal é medido, Then a razão é de pelo menos 4,5:1 (NFR-07).
- **AC-T.7** Given a página com todos os campos e botões, When a árvore de acessibilidade é inspecionada, Then cada campo tem label associado e cada botão tem nome acessível (NFR-10).
- **AC-T.8** Given a página em cada estado, When é analisada por ferramenta automática de acessibilidade, Then não há violações de WCAG 2.1 AA (NFR-06; padrão exato em OQ-06).
- **AC-T.9** Given a aplicação é acessada por `http://`, When a requisição é feita, Then o navegador é redirecionado para `https://` (NFR-17).
- **AC-T.10** Given a aplicação está em uso, When o armazenamento do navegador é inspecionado, Then a única chave gravada é a preferência de unidade (NFR-18, se OQ-12 for aprovada).
- **AC-T.11** Given a interface está renderizada, When os textos são inspecionados, Then nenhum rótulo, mensagem ou título está em outro idioma que não o pt-BR (NFR-19; códigos de condição conforme OQ-24).
- **AC-T.12** Given o viewport tem 1280 px, When a página é renderizada com clima exibido, Then o conteúdo é legível, não há rolagem horizontal e todas as funções estão acessíveis (NFR-14).
- **AC-T.13** Given a aplicação está em uso, When as requisições de rede são inspecionadas, Then nenhum dado pessoal é enviado além do termo de busca e das coordenadas da cidade escolhida (NFR-15).
- **AC-T.14** Given a rede é 4G simulada e as respostas de geocodificação, clima atual e previsão chegam em até 300 ms cada (mock), When a página é carregada e uma cidade é pesquisada, Then o clima atual e a previsão de 5 dias ficam visíveis em até 2,5 s medidos como LCP (NFR-01).
- **AC-T.15** Given a aplicação é testada em Chrome, Edge, Firefox e Safari (desktop), Chrome no Android e Safari no iOS (OQ-22), When o fluxo busca, escolha de cidade, troca de unidade e leitura da previsão é executado em cada navegador, Then o fluxo conclui sem erro no console e sem falha de layout (NFR-20).

## 5. Non-Functional Requirements

Os números de tempo (NFR-01, NFR-02 e NFR-03) são metas de produto desta spec. A validação por medição em teste está em OQ-27.

### 5.1 Desempenho

- **NFR-01:** o clima atual e a previsão de 5 dias ficam visíveis em até 2,5 s de
  LCP em rede 4G simulada. *Meta de produto (OQ-27).*
- **NFR-02:** a lista de resultados aparece até 1 s após a pausa na digitação. *Meta de produto (OQ-27).*

### 5.2 Disponibilidade e resiliência

- **NFR-03:** chamadas à Open-Meteo têm timeout de 5 s. Ao estourar, a interface
  não trava e mostra erro recuperável. *Meta de produto (OQ-27).*
- **NFR-04:** resposta incompleta ou inválida da API não derruba a tela. A
  aplicação exibe os dados válidos disponíveis e mostra "Indisponível" no lugar
  de cada campo ausente ou inválido. Nenhum campo vazio, `NaN` ou `undefined`
  aparece na tela.
- **NFR-05:** a política de retentativa automática e o tempo de vida
  do cache são definidos em OQ-20: não há cache de respostas e não há retentativa
  automática; a retentativa é manual (AC-06.4).

### 5.3 Acessibilidade

- **NFR-06:** conformidade com WCAG 2.1 AA (padrão exato em OQ-06).
- **NFR-07:** contraste mínimo de 4,5:1 para texto normal no tema dark
  glassmorphism, em todos os estados (inclusive erro, vazio e foco).
- **NFR-08:** foco visível em todos os controles e operação completa por teclado.
- **NFR-09:** loading, erro e vazio são anunciados a leitores de tela
  (`aria-live`).
- **NFR-10:** uso de semântica HTML e de labels associados a todos os campos e
  botões.

### 5.4 Responsividade

- **NFR-11:** layout utilizável de 320 px a 428 px, sem rolagem horizontal.
- **NFR-12:** alvos de toque com no mínimo 44×44 px.
- **NFR-13:** todas as funções disponíveis sem hover.
- **NFR-14:** layout de desktop (acima de 428 px) funcional e legível. *Escopo
  depende de OQ-04 e OQ-22.*

### 5.5 Privacidade e segurança

- **NFR-15:** não coletar dados pessoais. Geolocalização, se existir, é opcional e
  exige consentimento explícito (OQ-15).
- **NFR-16:** a entrada de busca é tratada como não confiável. Caracteres
  especiais e textos longos não quebram a interface nem são executados. Limite
  de tamanho: AC-01.8.
- **NFR-17:** a aplicação só é servida por HTTPS.
- **NFR-18:** a unidade de temperatura e a última cidade pesquisada (RF-09) são as
  únicas informações armazenadas no navegador (localStorage). Devem constar na política de privacidade.

### 5.6 Compatibilidade e idioma

- **NFR-19:** interface em pt-BR (D-05). Internacionalização fica fora de escopo.
- **NFR-20:** navegadores móveis e desktop com versões mínimas a definir (OQ-22).

### 5.7 Qualidade de processo

- **NFR-21:** lint (Biome), build e testes (Vitest e Playwright) passam antes de
  qualquer entrega. Requisito de processo; o Plan e a Definition of Done o detalham.

## 6. Edge Cases

| ID    | Situação                                                              | Comportamento esperado                                                                                  | Critério de referência |
| ----- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- | ---------------------- |
| EC-01 | Busca vazia ou só com espaços                                         | Nenhuma requisição; nenhuma mensagem de erro                                                            | AC-01.1                |
| EC-02 | Nome sem correspondência                                              | Estado vazio com sugestão de revisar o nome                                                             | AC-06.3                |
| EC-03 | Homônimos (ex.: duas cidades com o mesmo nome em países diferentes)   | Lista com país e região em cada item; nenhuma cidade é descartada                                       | AC-01.5                |
| EC-04 | Nome com acento ou sem acento                                         | Resultado equivalente; a normalização depende de OQ-25                                                  | AC-01.4                |
| EC-05 | Timeout (mais de 5 s)                                                 | Erro recuperável com "Tentar novamente"; a interface continua responsiva                                | AC-06.2                |
| EC-06 | Falha de rede ou API fora do ar (5xx)                                 | Mesmo tratamento de EC-05, com mensagem diferente só se OQ-05 pedir                                     | AC-06.2                |
| EC-07 | Resposta incompleta ou com valores inválidos (campo ausente, mínima maior que máxima, `NaN`) | Dados válidos são exibidos; campos inválidos mostram "Indisponível" (NFR-04) | AC-03.3, AC-03.4, AC-04.3 |
| EC-08 | Limite de uso da API atingido (resposta de limite excedido)           | Erro recuperável com mensagem clara; sem detalhes técnicos para o usuário                               | AC-06.2                |
| EC-09 | Cidade encontrada, mas sem dados de clima                             | Estado de erro específico ou vazio (texto em OQ-05); não exibe campos vazios                            | AC-06.2, NFR-04        |
| EC-10 | Usuário troca de cidade antes da resposta anterior chegar             | A resposta antiga é descartada                                                                          | AC-02.3, RG-04         |
| EC-11 | Usuário troca a unidade durante o carregamento                        | A unidade vale para o próximo dado exibido; nenhum valor fica em unidade mista                          | RG-05                  |
| EC-12 | Entrada muito longa ou com símbolos especiais                         | Limite de 255 caracteres (AC-01.8); sem quebra de layout nem execução de conteúdo              | NFR-16                 |
| EC-13 | Dispositivo sem rede ao abrir a aplicação                             | Estado de erro com "Tentar novamente"; não há modo offline (OQ-23)                         | AC-06.2                |
| EC-14 | Temperatura de 0 °C ou -40 °C                                         | Conversão correta para 32 °F e -40 °F                                                                   | AC-05.4                |
| EC-15 | Uso de teclado para escolher um resultado                             | Enter sobre o item carrega a cidade; o foco não se perde                                                | AC-02.1, AC-T.1        |
| EC-16 | Preferência de unidade salva e a aplicação reaberta                   | Depende de OQ-12; se não salva, volta para °C (D-03)                                                    | AC-05.1                |
| EC-17 | Conexão lenta, com carregamento acima de 2,5 s                        | O loading permanece visível e anunciado; não há tela em branco                                          | AC-06.1, NFR-01        |
| EC-18 | Falha parcial: clima atual recebido, previsão falha (timeout ou 5xx)  | Clima atual permanece; erro e "Tentar novamente" só na seção de previsão                                | AC-06.6                |

## 7. Assumptions

Premissas adotadas para seguir com a spec. Se alguma for falsa, a spec precisa
ser revista.

- **A-01:** a Open-Meteo atende ao volume esperado sem API key, e os limites
  gratuitos bastam para o treinamento (ver R1 e OQ-03).
- **A-02:** o clima atual e a previsão de 5 dias usam os dados diários e atuais
  fornecidos pela Open-Meteo, sem fonte adicional.
- **A-03:** a unidade padrão é Celsius (D-03).
- **A-04:** a conversão para Fahrenheit é feita no cliente, a partir do valor
  original em Celsius, sem nova chamada à API (ver também Plan).
- **A-05:** não há autenticação, favoritos nem histórico de buscas nesta versão; apenas a última cidade pesquisada é restaurada ao abrir (RF-09)
  (D-04).
- **A-06:** "dispositivos móveis" significa layout responsivo em navegador, sem
  app nativo.
- **A-07:** a aplicação é de página única, sem backend próprio.
- **A-08:** a interface é somente em pt-BR (D-05).
- **A-09:** os testes de erro usam uma estratégia de simulação da API decidida
  no Plan (ver OQ-08).

## 8. Risks

| ID  | Risco                                                                                          | Prob. | Impacto | Mitigação                                                                                                  | Referência |
| --- | ---------------------------------------------------------------------------------------------- | ----- | ------- | ---------------------------------------------------------------------------------------------------------- | ---------- |
| R1  | Open-Meteo fora do ar, lenta ou com limite de uso excedido                                     | Média | Alta    | Timeout de 5 s (NFR-03); erro com retentativa (AC-06.2, AC-06.4); cache leve (OQ-20); monitoramento (OQ-26) | EC-05, EC-08 |
| R2  | Uso comercial não permitido no plano gratuito da Open-Meteo                                    | Média | Alta    | Validar termos com jurídico antes de começar (OQ-03); avaliar plano pago ou provedor com camada de adaptação | OQ-03      |
| R3  | Formato da resposta muda sem aviso                                                             | Baixa | Alta    | Validação de dados na camada de serviço; testes com respostas gravadas; tratamento de campos ausentes       | NFR-04, EC-07 |
| R4  | Homônimos ou nomes acentuados confundem o usuário                                              | Alta  | Média   | País e região na lista (AC-01.5); normalização de acentos (OQ-25); testes com homônimos                    | EC-03, EC-04 |
| R5  | Erro de arredondamento ou conversão de unidades                                                | Média | Baixa   | Converter sempre a partir do valor original; testes de borda                                               | AC-05.4, EC-14 |
| R6  | Fuso horário errado, dias da previsão deslocados                                               | Média | Média   | Definir fuso de referência (OQ-19); exibir o fuso junto aos horários                                       | OQ-19      |
| R7  | Clima atual com definição diferente da esperada pelo usuário                                   | Média | Alta    | Fechar definição (OQ-02) antes do Plan; exibir horário da última atualização (RF-08)                       | RF-03, RF-08 |
| R8  | Escopo crescer (favoritos, alertas, histórico, mapas) sem controle                             | Alta  | Alta    | Lista de Out of Scope (seção 9); priorização do MVP (OQ-29)                                                | Seção 9    |
| R9  | Previsão errada gera dano ou reclamação, sem aviso de isenção                                  | Média | Alta    | Aviso de isenção aprovado pelo jurídico (OQ-21); origem dos dados visível (RF-07)                          | OQ-21      |
| R10 | Ausência de alertas meteorológicos em regiões de risco                                         | Baixa | Alta    | Decisão explícita (OQ-14); se não entrar, registrar a limitação no aviso de isenção                       | OQ-14      |
| R11 | Não conformidade com WCAG 2.1 AA ou eMAG, bloqueando o lançamento                              | Média | Alta    | Padrão definido (OQ-06); auditorias automatizadas e testes com teclado e leitor de tela                    | NFR-06     |
| R12 | Contraste insuficiente no tema dark glassmorphism                                              | Média | Média   | Checar todos os estados (NFR-07); tokens de cor validados antes da implementação                           | NFR-07     |
| R13 | Coleta de dados ou geolocalização sem base legal (LGPD)                                        | Baixa | Alta    | Não coletar sem decisão (OQ-15, OQ-31); geolocalização opcional com consentimento (NFR-15)                  | OQ-31      |
| R14 | Desempenho abaixo da meta em 4G, especialmente no celular                                      | Média | Média   | Medição de LCP em perfil móvel; carregamento sob demanda se necessário                                     | NFR-01     |
| R15 | Layout quebra em telas pequenas ou em navegadores móveis específicos                           | Média | Média   | Testes E2E em 320 px e 428 px; lista de navegadores (OQ-22)                                                | NFR-11     |
| R16 | Versões de bibliotecas e ferramentas (Vite, Tailwind, Vitest) quebram o projeto                | Baixa | Baixa   | Versões fixadas no lockfile; atualização planejada e testada                                               | NFR-21     |
| R17 | Ausência de monitoramento e de responsável por incidentes                                      | Média | Média   | Definir responsável e canal de alerta (OQ-26); registrar erros da API no cliente                           | OQ-26      |

## 9. Out of Scope

Itens explicitamente fora desta versão:

- Contas de usuário, login e cadastro.
- Favoritos, lista de cidades salvas e histórico de buscas (D-04).
- Notificações push e alertas de tempo.
- Mapas e camadas meteorológicas.
- Backend próprio ou qualquer persistência de dados no servidor.
- Aplicativo nativo (iOS ou Android) e PWA (OQ-04).
- Previsão horária (OQ-10).
- Busca por CEP, bairro ou coordenadas (ver OQ-16; por padrão, fora).
- Geolocalização automática (OQ-15).
- Internacionalização. Interface apenas em pt-BR (D-05).
- Modo offline completo (OQ-23).
- Busca por CEP, bairro ou coordenadas (OQ-16).
- Previsão além de 5 dias e histórico climático.
- Qualidade do ar, índice UV e pólen.
- Comparação lado a lado de duas cidades.
- Compartilhamento de link e rotas por URL (OQ-28).
- Analytics e coleta de dados de uso (OQ-31).
- Exportação de dados e integrações com outras APIs de tempo.
- Coleta de analytics (ver OQ-31; por padrão, fora).

## 10. Open Questions

Perguntas que a spec não respondia, com a decisão adotada nesta versão. As decisões
estão na tabela abaixo; o registro original segue depois. Itens **Pendente externo**
dependem de pessoa ou área fora do repositório e não bloqueiam o desenvolvimento.

### 10.1 Decisões adotadas

| ID    | Decisão                                                                                                                                 | Critério afetado            |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| OQ-01 | Métrica do MVP: clima visível em até 2,5 s (NFR-01) e taxa de buscas que terminam com clima exibido. Retenção não é medida (OQ-31).     | NFR-01                      |
| OQ-02 | "Clima atual" é a observação mais recente da API (campo `current`).                                                                      | RF-03                       |
| OQ-03 | **Pendente externo.** Confirmar se os termos da Open-Meteo permitem uso comercial. Bloqueia o lançamento comercial, não o desenvolvimento. | R2                          |
| OQ-04 | Web responsiva (SPA em React e Vite), sem PWA e sem app nativo. Desktop é suportado.                                                    | Escopo, Out of Scope        |
| OQ-05 | Textos da seção "Textos da interface" são os textos do MVP.                                                                              | AC-06.3, RF-06              |
| OQ-06 | WCAG 2.1 AA é obrigatório. eMAG não é exigido nesta versão; se a entrega for de órgão público, confirmar com OQ-07.                     | NFR-06                      |
| OQ-07 | **Pendente externo.** Dono e aprovador da entrega. Não bloqueia a implementação.                                                         | —                           |
| OQ-08 | Testes unitários e de integração usam respostas gravadas. E2E intercepta as requisições e não chama a API real. Teste real é opcional.   | Plan, Test                  |
| OQ-09 | Campos obrigatórios do clima atual: temperatura, sensação térmica, condição, umidade e vento.                                            | AC-03.1                     |
| OQ-10 | Previsão diária de 5 dias, com probabilidade de chuva em cada cartão. Sem previsão horária.                                              | RF-04, AC-04.6              |
| OQ-11 | A unidade de temperatura alterna entre °C e °F. Vento em km/h. Pressão não é exibida.                                                    | 2.3, RG-05                  |
| OQ-12 | A unidade persiste entre sessões no localStorage.                                                                                        | RF-05, AC-05.8, NFR-18      |
| OQ-13 | Sem lista de cidades, favoritos ou histórico. Só a última cidade pesquisada é restaurada ao abrir.                                       | RF-09, AC-09.1, AC-09.2     |
| OQ-14 | Alertas meteorológicos ficam fora desta versão.                                                                                          | Out of Scope                |
| OQ-15 | Geolocalização automática fica fora desta versão.                                                                                        | Out of Scope, NFR-15        |
| OQ-16 | A busca aceita apenas nome de cidade.                                                                                                    | Out of Scope                |
| OQ-17 | No máximo 10 resultados. Sem resultados: texto de AC-06.3.                                                                               | AC-01.6                     |
| OQ-18 | Busca global. Cada resultado mostra cidade, região e país para separar homônimos.                                                        | AC-02.1                     |
| OQ-19 | Horário no fuso da cidade, retornado pela API, em formato "HH:mm" de 24 h. Data em "dd/mm/aaaa".                                         | AC-04.1, AC-08.1            |
| OQ-20 | Sem cache de respostas e sem retentativa automática. A retentativa é manual.                                                             | NFR-05                      |
| OQ-21 | **Pendente externo.** Texto jurídico de atribuição. Até lá vale "Dados meteorológicos: Open-Meteo".                                      | AC-07.1                     |
| OQ-22 | Últimas duas versões principais de Chrome, Edge, Firefox e Safari (desktop), Chrome no Android e Safari no iOS.                          | NFR-20                      |
| OQ-23 | Sem modo offline. Sem rede, a tela mostra erro com "Tentar novamente".                                                                   | EC-13, AC-06.2              |
| OQ-24 | Interface em pt-BR. Códigos de condição são traduzidos por tabela fixa no app; código desconhecido mostra "Indisponível".               | AC-03.2                     |
| OQ-25 | O app envia o texto sem espaços das bordas à API e não normaliza acentos. Resultados são exibidos como a API devolve.                    | AC-01.4                     |
| OQ-26 | Build estático (SPA) servido por HTTPS (NFR-17). Monitoramento fica fora desta versão. Domínio e hospedagem: pendente de infraestrutura. | NFR-17                      |
| OQ-27 | 2,5 s, 1 s e 5 s são metas de produto, validadas por medição em teste.                                                                  | NFR-01, NFR-02, NFR-03      |
| OQ-28 | A URL não representa a cidade. Não há links compartilháveis nesta versão.                                                                | Out of Scope                |
| OQ-29 | **Pendente externo.** Data de entrega, modelo de negócio e corte do MVP. Esta spec define o MVP.                                         | R8                          |
| OQ-30 | Tema glassmorphism definido no projeto. Sem logotipo novo nesta versão.                                                                  | Seção 5 (visual)            |
| OQ-31 | Sem analytics e sem geolocalização. Armazenam-se só a unidade e a última cidade (NFR-18), sem dados pessoais.                           | NFR-15, NFR-18              |
| OQ-32 | Open-Meteo: previsão com `timezone=auto`; `temperature_unit=celsius` (conversão no cliente); geocodificação com `count=10` e `language=pt`. | AC-04.1, RG-05              |

### 10.2 Registro original das perguntas

Coluna **Bloqueia** mostra o estado antes da decisão.

| ID    | Pergunta | Prioridade | Bloqueia | Impacto se não respondida | Origem |
| ID    | Pergunta                                                                                                          | Prioridade | Bloqueia | Impacto se não respondida                                                  | Origem     |
| ----- | ----------------------------------------------------------------------------------------------------------------- | ---------- | -------- | -------------------------------------------------------------------------- | ---------- |
| OQ-01 | Qual problema de negócio a aplicação resolve, e como medir sucesso (uso, retenção, tempo até o resultado)? Meta de Mariana: menos de 15 s? | P1 | Sim      | Sem métrica, não há como priorizar funções nem encerrar o projeto           | Pergunta 1 |
| OQ-02 | O que é "clima atual": observação mais recente, previsão da hora corrente ou média do dia?                       | P1         | Sim      | Números diferentes para o mesmo horário geram percepção de erro; afeta RF-03 e NFR-01 | Pergunta 6 |
| OQ-03 | Os termos da Open-Meteo permitem uso comercial no plano gratuito? A empresa aceita a dependência?                | P1         | Sim      | Risco de bloqueio após o lançamento (R2)                                  | Pergunta 16 |
| OQ-04 | Web, PWA ou app nativo? Desktop é suportado?                                                                     | P1         | Sim      | PWA muda estratégia de cache e service worker; nativo muda a equipe         | Perguntas 20 e 22 |
| OQ-05 | Quais são os textos exatos e o comportamento de erro e vazio (cidade sem dados, busca sem resultado, API fora do ar)? | P1 | Sim      | Cada desenvolvedor cria uma mensagem diferente; testes sem critério         | Pergunta 26 |
| OQ-06 | É obrigatório atender ao eMAG, além de WCAG 2.1 AA?                                                               | P1         | Sim      | Se for órgão público, a não conformidade pode impedir o lançamento          | Pergunta 28 |
| OQ-07 | Quem é o dono e o aprovador da entrega? Personas e métricas precisam ser validadas com a empresa.                 | P1         | Sim      | Decisões ficam sem dono e são adiadas                                       | Pergunta 4 |
| OQ-08 | Como os testes lidam com a API externa: respostas gravadas, mocks ou chamadas reais no E2E?                       | P1         | Sim      | Testes de erro não são possíveis de forma confiável                         | Seção 9.3 do discovery |
| OQ-09 | Quais campos do clima atual são obrigatórios: sensação térmica, UV, chuva, pressão, além dos já listados em RF-03? | P2 | Não      | Telas divergem e critérios ficam sem base                                   | Pergunta 8 |
| OQ-10 | A previsão de 5 dias inclui probabilidade de chuva? A granularidade é diária ou horária?                          | P2         | Não      | Pode haver 4 ou 6 cartões; a chuva é pedido comum (Persona 1)               | Pergunta 9 |
| OQ-11 | A troca de unidade vale só para temperatura ou também para vento e pressão? Quais unidades de vento e pressão?    | P2         | Não      | Vento em km/h com temperatura em °F gera inconsistência                     | Pergunta 10 e 9.1 |
| OQ-12 | A preferência de unidade é salva entre sessões (localStorage) ou só durante a sessão? Afeta EC-16 e NFR-18.        | P2         | Não      | Usuário troca a unidade a cada visita                                       | Perguntas 11 e 30 |
| OQ-13 | Múltiplas cidades, histórico e favoritos entram nesta versão? Persona 2 depende de lembrar a última cidade.        | P2         | Não      | Se entrar depois, a arquitetura de estado e persistência muda; se não entrar, Persona 2 precisa mudar | Pergunta 12 e 9.2 |
| OQ-14 | Alertas meteorológicos (tempestade, calor extremo) são exigidos?                                                  | P2         | Não      | Exige nova fonte de dados; consequência real em regiões de risco (R10)      | Pergunta 13 |
| OQ-15 | Geolocalização automática é exigida? Se sim, como é o consentimento?                                              | P2         | Não      | Exige permissão do navegador e impacta NFR-15                               | Pergunta 14 |
| OQ-16 | Além do nome, a busca aceita CEP, bairro ou coordenadas?                                                          | P2         | Não      | Buscar por CEP sem suporte gera reclamações imediatas                       | Pergunta 7 |
| OQ-17 | Quantos resultados exibir no máximo? Qual a mensagem quando não há resultados (além de AC-06.3)?                  | P2         | Não      | Listas longas atrapalham no celular (AC-01.6)                               | Pergunta 15 |
| OQ-18 | A aplicação é global ou restrita a alguns países?                                                                 | P2         | Não      | Busca global exige tratar homônimos, fusos, unidades e idiomas              | Pergunta 5 |
| OQ-19 | Qual fuso horário é usado: o da cidade ou o do usuário? Qual formato de data e hora?                              | P2         | Não      | "Amanhã" e os dias da previsão podem aparecer deslocados (R6)               | Pergunta 19 |
| OQ-20 | Qual é a defasagem aceitável dos dados, o tempo de vida do cache e a política de retentativa automática (número e intervalo)? | P2 | Não | Cada tela busca dados de forma diferente, com custo e latência imprevisíveis | Perguntas 18 e 9.1 |
| OQ-21 | Qual é o aviso de isenção de responsabilidade e o texto de atribuição da Open-Meteo? Foi validado pelo jurídico? | P2 | Não | Exposição a reclamações e processos (R9); afeta RF-07 | Pergunta 17 |
| OQ-22 | Quais navegadores móveis e desktop e quais versões mínimas são suportados?                                        | P2         | Não      | Sem isso não há como definir o que testar nem o que é bug (NFR-20)          | Pergunta 21 |
| OQ-23 | O app precisa funcionar sem rede ou com conexão lenta além do estado de erro?                                     | P2         | Não      | Estratégia de cache e mensagens de erro ficam indefinidas (EC-13)           | Pergunta 23 |
| OQ-24 | A interface é só pt-BR? Os códigos de condição da API (ex.: "parcialmente nublado") serão traduzidos e mantidos? | P2 | Não | Descrições ficam em inglês ou exigem tradução mantida pelo time             | Pergunta 24 |
| OQ-25 | Limite de caracteres decidido: 255. Como tratar acentos e símbolos ("São Paulo" e "Sao Paulo")?                    | P2         | Não      | Resultados inconsistentes e necessidade de tratamento de entradas maliciosas (EC-04, EC-12) | Pergunta 27 |
| OQ-26 | Quem monitora a aplicação, o que acontece quando a Open-Meteo cai, e onde/como é feito o deploy e o domínio?      | P2         | Não      | Queda só é percebida pelos usuários; HTTPS (NFR-17) sem implementação definida | Perguntas 31 e 32 |
| OQ-27 | Os valores de NFR-01, NFR-02 e NFR-03 (2,5 s, 1 s, 5 s) têm fonte ou precisam ser validados com a empresa?          | P2         | Não      | Viram critérios de teste sem base                                           | Seção 9.2 do discovery |
| OQ-28 | O que a URL representa? Deve ser possível voltar à cidade anterior pelo botão do navegador ou compartilhar link?  | P2         | Não      | Sem isso, não há como voltar à cidade anterior nem compartilhar                 | Seção 9.1 do discovery |
| OQ-29 | Qual é a data de entrega, o modelo de negócio (público, interno ou pago, com anúncios) e o que compõe o MVP?      | P2         | Não      | Escopo fica aberto, sem critério de corte (R8)                              | Perguntas 2 e 3 |
| OQ-30 | Há guia de estilo ou logotipo além do tema glassmorphism definido no projeto?                                     | P3         | Não      | Identidade visual improvisada; retrabalho na revisão de design              | Pergunta 25 |
| OQ-31 | Há analytics ou geolocalização? Qual é a base legal (LGPD)?                                                       | P2         | Não      | Coleta sem base legal exige correção cara depois (R13)                      | Pergunta 29 |
| OQ-32 | Qual é o padrão de contrato técnico da Open-Meteo (endpoints, fuso pedido à API, idioma e quantidade da geocodificação)? | P2 | Não | "Dia" e "agora" podem ser calculados de formas diferentes                   | Seção 9.1 do discovery |

## 11. Matriz de rastreabilidade

Cadeia de rastreio: User Story → Requisito Funcional (RF) → Acceptance Criteria (AC) → NFR. Os ACs de RF-01 a RF-09 estão na seção 4 com o mesmo prefixo (ex.: AC-03.x pertence a RF-03).

### 11.1 User Story → RF → AC → NFR

| US    | Persona  | RF             | Acceptance Criteria                                              | NFR relevantes               |
| ----- | -------- | -------------- | ---------------------------------------------------------------- | ---------------------------- |
| US-01 | Mariana  | RF-01, RF-02   | AC-01.1 a AC-01.10; AC-02.1 a AC-02.4                            | NFR-02, NFR-03, NFR-04, NFR-16 |
| US-02 | Carlos   | RF-03          | AC-03.1 a AC-03.4                                                | NFR-01, NFR-04               |
| US-03 | Mariana  | RF-04          | AC-04.1 a AC-04.6                                                | NFR-01, NFR-04               |
| US-04 | Carlos   | RF-05          | AC-05.1, AC-05.2, AC-05.4, AC-05.5, AC-05.6, AC-05.7             | NFR-07, NFR-08, NFR-18       |
| US-05 | Juliana  | RF-05          | AC-05.3, AC-05.8                                                 | NFR-18                       |
| US-06 | Carlos   | RF-06          | AC-06.1, AC-06.2, AC-06.3, AC-06.5, AC-06.8                      | NFR-03, NFR-09               |
| US-07 | Mariana  | RF-06          | AC-06.2, AC-06.4, AC-06.6, AC-06.9                               | NFR-03, NFR-05, NFR-09       |
| US-08 | Juliana  | RF-01, RF-06   | AC-06.1, AC-06.8, AC-T.7                                         | NFR-06, NFR-09, NFR-10       |
| US-09 | Juliana  | RF-01, RF-05   | AC-05.5, AC-T.1, AC-T.4, AC-T.7                                  | NFR-06, NFR-08, NFR-13       |
| US-10 | Mariana  | RF-01          | AC-01.2, AC-01.5                                                 | NFR-04                       |
| US-11 | Carlos   | RF-07, RF-08   | AC-07.1, AC-08.1                                                 | NFR-04                       |
| US-12 | Carlos   | RF-09          | AC-09.1, AC-09.2                                                 | NFR-18                       |

### 11.2 NFR → Acceptance Criteria

| NFR    | Acceptance Criteria                  | Situação                                          |
| ------ | ------------------------------------ | ------------------------------------------------- |
| NFR-01 | AC-T.14                              | Coberto                                           |
| NFR-02 | AC-01.7                              | Coberto                                           |
| NFR-03 | AC-06.2, AC-06.6                     | Coberto                                           |
| NFR-04 | AC-01.2, AC-03.3, AC-04.3, AC-04.6, AC-08.1 | Coberto                                    |
| NFR-05 | AC-06.4, AC-06.9                     | Coberto                                           |
| NFR-06 | AC-T.8                               | Coberto                                           |
| NFR-07 | AC-T.6                               | Coberto                                           |
| NFR-08 | AC-T.1, AC-05.5                      | Coberto                                           |
| NFR-09 | AC-06.1, AC-06.8                     | Coberto                                           |
| NFR-10 | AC-T.7                               | Coberto                                           |
| NFR-11 | AC-T.2                               | Coberto                                           |
| NFR-12 | AC-T.3                               | Coberto                                           |
| NFR-13 | AC-T.4                               | Coberto                                           |
| NFR-14 | AC-T.12                              | Coberto                                           |
| NFR-15 | AC-T.13                              | Coberto                                           |
| NFR-16 | AC-T.5                               | Coberto                                           |
| NFR-17 | AC-T.9                               | Coberto                                           |
| NFR-18 | AC-T.10, AC-05.8, AC-09.1            | Coberto                                           |
| NFR-19 | AC-T.11                              | Coberto                                           |
| NFR-20 | AC-T.15                              | Coberto                                           |
| NFR-21 | —                                    | Processo: checklist do projeto (lint, build, test) |

### 11.3 Lacunas de rastreabilidade (resolvidas)

- **RF-09 (AC-09.1, AC-09.2):** coberto pela US-12.
- **NFR-01:** coberto pelo AC-T.14.
- **NFR-20:** coberto pelo AC-T.15, que usa a lista de navegadores de OQ-22.
- **Transversais sem US** (AC-T.2, T.3, T.5, T.6, T.8 a T.15): rastreados diretamente pelos NFRs da seção 11.2.


