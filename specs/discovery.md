# Discovery — Aplicação de Previsão do Tempo

## Resumo executivo

- A empresa quer um app para consultar o clima de cidades, pensado principalmente para celular.
- Os usuários poderão buscar uma cidade, ver o clima atual, a previsão dos próximos 5 dias e trocar entre Celsius e Fahrenheit.
- Os dados virão de um serviço público de clima, sem custo de cadastro e sem contas de usuário nesta versão.
- Os riscos principais são a dependência desse serviço, a validação dos termos de uso comercial e a definição do que é "clima atual" para o usuário.
- Antes de especificar, precisamos fechar seis decisões (métrica de sucesso, uso comercial, formato do app, mensagens de erro, acessibilidade legal e definição de clima atual) e definir um responsável pela entrega.


## 1. Contexto

- **Problema:** a empresa precisa de uma aplicação que informe o clima de cidades de forma simples e acessível.
- **Público-alvo:** usuários finais que consultam o tempo, principalmente pelo celular.
- **Objetivo:** permitir consulta rápida do clima atual e da previsão de 5 dias para qualquer cidade, com preferência de unidade de temperatura.
- **Stack definida no projeto:** React + Vite, TypeScript strict, Tailwind (tema dark glassmorphism), Open-Meteo (geocoding e forecast, sem API key), Vitest, Playwright e Biome.
- **Fora do briefing:** contas de usuário, favoritos, notificações e histórico não foram mencionados.

## 2. Requisitos Funcionais

| ID | Requisito | Critério de aceite inicial |
|----|-----------|----------------------------|
| RF-01 | Buscar cidades por nome | Ao digitar um nome, o sistema lista cidades correspondentes (nome, país/região) |
| RF-02 | Selecionar uma cidade do resultado | Ao escolher uma cidade, a aplicação carrega o clima dela |
| RF-03 | Exibir clima atual | Mostra temperatura, sensação térmica, condição (ícone e texto), umidade e vento |
| RF-04 | Exibir previsão de 5 dias | Mostra 5 dias com temperatura mínima e máxima e condição de cada dia |
| RF-05 | Alternar Celsius/Fahrenheit | Um controle troca todas as temperaturas exibidas; a escolha é mantida ao buscar outra cidade |
| RF-06 | Exibir estados de loading, erro e vazio | Cada operação de busca e carregamento exibe feedback apropriado; erro de API mostra mensagem com opção de tentar novamente |

## 3. Requisitos Não-Funcionais

### Desempenho

- **NFR-01:** o clima atual e a previsão de 5 dias ficam visíveis em até 2,5 s (LCP) em 4G simulado. Valor a confirmar com a empresa.
- **NFR-02:** a busca de cidades responde em até 1 s após a digitação, com debounce.

### Disponibilidade e resiliência

- **NFR-03:** chamadas à Open-Meteo têm timeout de 5 s. Ao estourar, a interface não trava e exibe erro recuperável.
- **NFR-04:** resposta incompleta da API é tratada como erro, sem exibir campos vazios ou `NaN`.

### Acessibilidade

- **NFR-05:** conformidade com WCAG 2.1 AA.
- **NFR-06:** contraste mínimo de 4,5:1 para texto no tema dark glassmorphism.
- **NFR-07:** foco visível em todos os controles e operação completa por teclado.
- **NFR-08:** estados de loading e erro são anunciados a leitores de tela (`aria-live`).

### Responsividade

- **NFR-09:** layout utilizável de 320 px a 428 px, sem rolagem horizontal.
- **NFR-10:** alvos de toque com no mínimo 44×44 px.
- **NFR-11:** todas as funções estão disponíveis sem hover.

### Privacidade e segurança

- **NFR-12:** não coletar dados pessoais. Geolocalização, se existir, é opcional e exige consentimento.
- **NFR-13:** a entrada de busca é tratada como não confiável.
- **NFR-14:** a aplicação roda apenas em HTTPS.

### Compatibilidade

- **NFR-15:** interface em pt-BR, conforme as convenções do projeto.
- **NFR-16:** versões mínimas de navegadores móveis a definir (ver pergunta 8).

### Restrições (fora dos requisitos de produto)

- **Dependência externa:** a aplicação depende da disponibilidade e dos limites de uso da Open-Meteo. Registrada em Riscos e Suposições.
- **Qualidade de processo:** lint (Biome), build e testes (Vitest e Playwright) passando antes de qualquer entrega. Pertence ao Plan e à Definition of Done.

## 4. Riscos

| # | Tipo | Risco | Probabilidade | Impacto | Estratégia de mitigação |
|---|------|-------|---------------|---------|--------------------------|
| R1 | Técnico | Open-Meteo fora do ar, lenta ou com limite de uso excedido | Média | Alto | Timeout de 5 s (NFR-03), mensagem de erro com retentativa, cache em memória, monitoramento de disponibilidade |
| R2 | Produto/Legal | Uso comercial não permitido no plano gratuito da Open-Meteo | Média | Alto | Validar os termos com jurídico antes de começar (pergunta 16); avaliar plano pago ou provedor alternativo com camada de adaptação |
| R3 | Técnico | Formato de resposta da API muda sem aviso | Baixa | Alto | Validação de schema na camada de serviço, testes com respostas gravadas e tratamento de campos ausentes (NFR-04) |
| R4 | Técnico | Ambiguidade em cidades homônimas ou com nomes acentuados | Alta | Médio | Exibir país e região na lista de resultados; normalizar acentos na busca; testes com casos de homônimos |
| R5 | Técnico | Erro de arredondamento ou conversão de unidades | Média | Baixo | Converter sempre a partir do valor original, sem conversões encadeadas; testes de borda para 0 °C, -40 °C e arredondamento |
| R6 | Técnico | Fuso horário errado, dias da previsão deslocados | Média | Médio | Definir fuso da cidade como referência (pergunta 19); exibir o fuso junto aos horários; testes com cidades em fusos diferentes |
| R7 | Produto | Clima atual com definição diferente do esperado pelo usuário | Média | Alto | Fechar a definição de "clima atual" antes da Spec (pergunta 6); exibir horário da última atualização |
| R8 | Produto | Escopo crescer (favoritos, alertas, histórico, mapas) sem controle | Alta | Alto | Lista explícita de itens fora de escopo na Spec; priorização por MVP (pergunta 3) |
| R9 | Produto/Legal | Previsão errada gera dano ou reclamação, sem aviso de isenção | Média | Alto | Aviso de isenção aprovado pelo jurídico (pergunta 17); fonte dos dados exibida na tela |
| R10 | Produto | Ausência de alertas meteorológicos em regiões de risco | Baixa | Alto | Decisão explícita sobre alertas (pergunta 13); se não entrar, registrar a limitação no aviso de isenção |
| R11 | Acessibilidade | Não conformidade com WCAG 2.1 AA ou eMAG, bloqueando o lançamento | Média | Alto | Definir o padrão exigido (pergunta 28); auditorias automatizadas e testes com teclado e leitor de tela no Playwright |
| R12 | Acessibilidade | Contraste insuficiente no tema dark glassmorphism | Média | Médio | Checar contraste de todos os estados (NFR-06); tokens de cor validados antes da implementação |
| R13 | Privacidade | Coleta de dados ou geolocalização sem base legal (LGPD) | Baixa | Alto | Não coletar dados sem decisão (perguntas 14 e 29); geolocalização opcional com consentimento (NFR-12) |
| R14 | Técnico | Desempenho abaixo da meta em 4G, especialmente no celular | Média | Médio | Métrica de LCP (NFR-01); medição com Lighthouse em perfil móvel; carregamento de código sob demanda se necessário |
| R15 | Técnico | Layout quebra em telas pequenas ou em navegadores móveis específicos | Média | Médio | Testes E2E de Playwright em viewports de 320 px e 428 px; lista de navegadores suportados (pergunta 21) |
| R16 | Técnico | Dependência de versão de biblioteca ou ferramenta (Vite, Tailwind, Vitest) | Baixa | Baixo | Versões fixadas no `pnpm-lock.yaml`; atualização planejada e testada |
| R17 | Operação | Ausência de monitoramento e de responsável por incidentes | Média | Médio | Definir responsável e canal de alerta (pergunta 31); registrar erros da API no cliente |

**Prioridade de tratamento:** R2, R7 e R8 combinam probabilidade média ou alta com impacto alto e dependem de decisões de negócio.


## 5. Perguntas em Aberto

Cada pergunta traz o impacto de seguir sem resposta. As prioridades estão marcadas com **[P1]**.

### A. Negócio e objetivo

1. **[P1] Métrica de sucesso:** qual problema de negócio a aplicação resolve e como medir sucesso (uso, retenção, tempo até o resultado)?
   - *Impacto:* sem métrica, não há como priorizar funções nem encerrar o projeto.
2. **Público e modelo de negócio:** é um produto público, ferramenta interna ou produto pago? Há anúncios?
   - *Impacto:* muda prioridades de performance, marca e coleta de dados.
3. **Prazo, orçamento e MVP:** qual é a data de entrega e o que compõe o MVP?
   - *Impacto:* o escopo fica aberto, sem critério de corte.
4. **Dono e aprovador:** quem decide mudanças de requisito e aceita a entrega?
   - *Impacto:* decisões ficam sem dono e são adiadas.
5. **Abrangência geográfica:** a aplicação é global ou restrita a alguns países?
   - *Impacto:* busca global exige tratar homônimos, fusos, unidades e idiomas.

### B. Escopo funcional

6. **[P1] Definição de "clima atual":** é a observação mais recente, a previsão da hora corrente ou a média do dia?
   - *Impacto:* números diferentes para o mesmo horário geram a percepção de erro.
7. **Modos de busca:** além de nome, a busca aceita CEP, bairro ou coordenadas?
   - *Impacto:* a Open-Meteo faz geocodificação por nome; buscar por CEP sem suporte gera reclamações imediatas.
8. **Campos do clima atual:** quais são obrigatórios (umidade, vento, sensação térmica, UV, chuva, pressão)?
   - *Impacto:* sem lista fechada, telas divergem e os critérios de aceite ficam sem base.
9. **Previsão de 5 dias:** inclui hoje? Qual granularidade (diária ou horária)? Inclui probabilidade de chuva?
   - *Impacto:* pode haver 4 ou 6 cartões, e a ausência de chuva é um pedido comum.
10. **Alternância de unidades:** a troca vale só para temperatura ou também para vento e pressão?
    - *Impacto:* vento em km/h com temperatura em °F gera inconsistência.
11. **Preferências:** qual é a unidade padrão e a escolha é salva entre sessões (localStorage) ou só durante a sessão?
    - *Impacto:* o usuário precisa trocar a unidade a cada visita.
12. **Múltiplas cidades, histórico e favoritos:** entram nesta versão?
    - *Impacto:* se entrarem depois, a arquitetura de estado e persistência muda.
13. **Alertas meteorológicos:** são exigidos (tempestade, calor extremo)?
    - *Impacto:* exige nova fonte de dados e pode ter consequência real em regiões de risco.
14. **Geolocalização automática:** além da busca manual, deve localizar o usuário automaticamente?
    - *Impacto:* exige permissão do navegador e impacta a privacidade (NFR-12).
15. **Resultados de busca:** quantos resultados exibir no máximo e o que mostrar quando não houver nenhum?
    - *Impacto:* listas longas atrapalham no celular e a ausência de mensagem deixa a tela vazia.

### C. Dados e fontes

16. **[P1] Uso comercial da Open-Meteo:** os termos permitem uso comercial no plano gratuito? A empresa aceita a dependência?
    - *Impacto:* se o plano gratuito não cobrir o uso, o projeto pode ser bloqueado após o lançamento. Validar com jurídico antes de começar.
17. **Responsabilidade por erro de previsão:** haverá aviso de isenção? O texto foi validado pelo jurídico?
    - *Impacto:* exposição a reclamações e processos.
18. **Frequência de atualização e cache:** quanto tempo de defasagem é aceitável?
    - *Impacto:* cada tela pode buscar dados de forma diferente, com custo e latência imprevisíveis.
19. **Fuso horário e formato de data:** usar o fuso da cidade ou do usuário?
    - *Impacto:* "amanhã" e os dias da previsão podem aparecer deslocados.

### D. Plataforma e experiência

20. **[P1] Web, PWA ou app nativo:** qual formato será entregue?
    - *Impacto:* PWA permite instalação e uso parcial offline; app nativo exige lojas e outra equipe. Escolher errado custa semanas.
21. **Navegadores e dispositivos móveis:** quais sistemas e versões mínimas são suportados?
    - *Impacto:* sem isso não há como definir o que testar nem o que é bug.
22. **Desktop:** também é suportado?
    - *Impacto:* o layout desktop fica sem dono e sem teste.
23. **Offline e conexão lenta:** o app precisa funcionar sem rede?
    - *Impacto:* a estratégia de cache e as mensagens de erro ficam indefinidas.
24. **Idioma e localização:** só pt-BR ou também i18n? Os códigos de clima da API serão traduzidos?
    - *Impacto:* descrições como "parcialmente nublado" vêm em inglês e precisam de tradução mantida pelo time.

### E. Erros e casos limite

25. **Guia de marca:** há guia de estilo ou logotipo além do tema glassmorphism definido no projeto?
    - *Impacto:* sem guia, a identidade visual é improvisada e pode precisar de retrabalho na revisão de design.

26. **[P1] Estados de erro e vazio:** o que o usuário vê quando a cidade não tem dados, a busca não retorna nada ou a API cai?
    - *Impacto:* cada desenvolvedor cria uma mensagem diferente e os testes ficam sem critério.
27. **Entrada de busca:** há limite de caracteres? Como tratar acentos e símbolos (por exemplo, "São Paulo" e "Sao Paulo")?
    - *Impacto:* resultados inconsistentes e necessidade de tratamento de entradas maliciosas.

### F. Acessibilidade, privacidade e conformidade

28. **[P1] Obrigação legal de acessibilidade:** é necessário atender ao eMAG ou apenas WCAG?
    - *Impacto:* se for órgão público, a não conformidade pode impedir o lançamento.
29. **Coleta de dados e LGPD:** há analytics ou geolocalização? Qual base legal?
    - *Impacto:* coleta sem base legal exige correção cara depois.
30. **Dados no navegador:** a preferência de unidade é guardada no localStorage?
    - *Impacto:* precisa constar na política de privacidade.

### G. Operação

31. **Monitoramento e queda da API:** quem monitora e o que acontece quando a Open-Meteo cai?
    - *Impacto:* sem monitoramento, a queda só é percebida pelos usuários.
32. **Hospedagem e deploy:** onde será hospedado, com qual domínio e pipeline?
    - *Impacto:* sem isso, o projeto não vai para produção e o requisito de HTTPS (NFR-14) não tem implementação definida.

## 6. Suposições

- A Open-Meteo atende ao volume esperado sem API key, e seus limites gratuitos são suficientes para o treinamento.
- O clima atual e a previsão de 5 dias usam os dados diários fornecidos pela Open-Meteo.
- A unidade padrão é Celsius.
- A conversão para Fahrenheit é feita no cliente, sem nova chamada à API.
- Não há autenticação, favoritos nem histórico de buscas nesta versão.
- "Dispositivos móveis" significa layout responsivo em navegador, sem app nativo.
- A aplicação é de página única (SPA), sem backend próprio.

## 7. Personas

Personas hipotéticas, a validar com a empresa (pergunta 2).

### Persona 1: Mariana, viajante de fim de semana

- **Perfil:** 29 anos, planeja viagens curtas e precisa decidir o que levar.
- **Objetivo principal:** saber se vai chover nos próximos dias na cidade de destino, para escolher roupa e programação.
- **Contexto de uso:** mobile, no transporte ou na fila de embarque, com conexão instável. Consulta em menos de um minuto, muitas vezes com uma mão.
- **Métrica de sucesso:** tempo entre abrir o app e saber se vai chover no destino. Meta: menos de 15 segundos. Depende da previsão de chuva por dia (pergunta 9).

### Persona 2: Carlos, morador que acompanha o dia a dia

- **Perfil:** 42 anos, consulta o tempo toda manhã, antes de sair de casa.
- **Objetivo principal:** ver a temperatura atual e a sensação térmica para decidir se leva casaco ou guarda-chuva.
- **Contexto de uso:** mobile, rápido e recorrente, de manhã. Usa a cidade de residência como ponto de partida, com a busca como atalho.
- **Métrica de sucesso:** número de consultas por semana sem precisar buscar a cidade de novo. Meta: clima visível em até 2,5 s (NFR-01). Depende de decidir sobre favoritos ou sobre lembrar a última cidade (pergunta 12).

### Persona 3: Juliana, usuária de desktop e leitor de tela

- **Perfil:** 35 anos, trabalha em escritório e consulta o tempo no computador para planejar reuniões e eventos externos. Usa leitor de tela.
- **Objetivo principal:** comparar a previsão de 5 dias de duas cidades e ler os valores em °F quando precisa.
- **Contexto de uso:** desktop, com teclado e leitor de tela. Também pode usar o celular, mas não é o principal.
- **Métrica de sucesso:** concluir busca, troca de unidade e leitura da previsão só com teclado e leitor de tela, sem bloqueios (NFR-07 e NFR-08). Também conta o tempo para trocar de °C para °F e ver o resultado atualizado.

**Observação:** a persona 3 coloca desktop na mesma prioridade do mobile, o que conflita com o briefing e depende da pergunta 20. Se a decisão for mobile-first, ela vira secundária.

## 8. Decisões

Decisões fechadas nesta fase. Cada uma indica quais perguntas da seção 5 resolve e o que continua em aberto.

### D-01: Fonte de dados é a Open-Meteo, sem API key

- **Justificativa:** é a fonte já definida na stack do projeto e não exige cadastro nem gestão de chave secreta.
- **Resolve:** a escolha de fonte de dados.
- **Continua em aberto:** os termos de uso comercial (pergunta 16). A decisão não resolve se o plano gratuito pode ser usado comercialmente.

### D-02: "5 dias" significa hoje mais 4 dias

- **Justificativa:** a previsão começa no dia atual, e o usuário espera ver hoje na primeira posição.
- **Resolve:** parte da pergunta 9 (se inclui hoje).
- **Continua em aberto:** a granularidade (diária ou horária) e a probabilidade de chuva, ainda na pergunta 9.

### D-03: Unidade padrão é Celsius

- **Justificativa:** é o padrão de quase todo o público-alvo do briefing, e a conversão para Fahrenheit é feita no cliente.
- **Resolve:** a unidade inicial da pergunta 11.
- **Continua em aberto:** se a escolha do usuário é salva entre sessões (pergunta 11 e pergunta 30).

### D-04: Sem autenticação e sem persistência de servidor

- **Justificativa:** o briefing não pede contas nem histórico. Sem backend próprio, o custo, a superfície de segurança e a manutenção são menores.
- **Resolve:** parte da pergunta 12 (múltiplas cidades, histórico e favoritos ficam fora do lado do servidor).
- **Continua em aberto:** se a preferência de unidade é guardada no navegador (pergunta 30), já que isso não é persistência de servidor.

### D-05: Idioma da interface é pt-BR

- **Justificativa:** o público-alvo do briefing é brasileiro e as convenções do projeto já definem pt-BR. Internacionalização fica fora desta versão.
- **Resolve:** a pergunta 24 sobre idioma da interface, deixando i18n fora de escopo.
- **Continua em aberto:** a tradução dos códigos de clima da API (pergunta 24) e a abrangência geográfica (pergunta 5).

## 9. Revisão de arquitetura: lacunas antes da Spec

Revisão feita sobre as seções 1 a 8. Cada item indica o risco de retrabalho ou a decisão necessária.

### 9.1 Pontos vagos

- **Endpoints e contrato da Open-Meteo:** não está definido quais endpoints (atual, horário, diário) alimentam cada requisito, nem qual fuso horário é pedido à API. Sem isso, "dia" e "agora" podem ser calculados de forma diferente.
- **Unidades pela API ou pelo cliente:** a API aceita pedido de temperatura em °F diretamente. A D-03 e a Suposição dizem que a conversão é feita no cliente, sem decidir o motivo. Escolher agora evita divergência de arredondamento (R5).
- **Unidade do vento e da pressão:** não definidas (pergunta 10).
- **Geocodificação:** não está definido idioma de resposta, quantidade de resultados pedida à API e ordenação.
- **Definição de "clima atual"** continua em aberto (pergunta 6), mas os requisitos RF-03 e NFR-01 já dependem dela.
- **Política de cache:** a Suposição diz "cache leve em memória" (R1), sem tempo de vida definido.
- **Política de retentativa:** a NFR-03 define timeout, mas não o número de tentativas nem o intervalo.
- **Estado da aplicação:** não está definido o que a URL representa. Sem isso, não há como voltar à cidade anterior pelo botão do navegador nem compartilhar um link.

### 9.2 Pontos que geram retrabalho

- **Persona 2 contradiz D-04:** a métrica de Carlos depende de lembrar a última cidade, o que exige persistência no navegador. Se isso for descartado, a persona precisa mudar.
- **Persona 3 contradiz RF-06 e a seção de dispositivos:** o requisito fala só em mobile. Se desktop entrar, a estratégia de layout muda desde o início.
- **Web, PWA ou nativo (pergunta 20, P1):** se for PWA, a estratégia de cache e o service worker entram na arquitetura. Decidir depois força refazer a camada de dados.
- **Stack misturada com requisitos:** a seção 1 traz a stack como se fosse requisito. Stack é decisão do Plan. Manter as duas coisas juntas faz a Spec herdar escolhas técnicas.
- **Camada de adaptação da API (R2):** a mitigação sugere trocar de provedor, mas nenhum requisito obriga a isolar a API em um serviço. Pelas convenções do projeto, a lógica de acesso fica em `src/services/`. Isso precisa virar requisito técnico no Plan, para que a troca não exija reescrever componentes.
- **Números sem fonte:** 2,5 s, 1 s e 5 s (NFR-01 a NFR-03) foram propostos por mim. Se entrarem nos testes sem validação, viram critério de aceite sem base.

### 9.3 O que falta para começar a Spec com segurança

Perguntas da seção 5 que bloqueiam a Spec, por serem P1 ou afetarem critérios de aceite:

- **Pergunta 1:** métrica de sucesso.
- **Pergunta 6:** definição de "clima atual".
- **Pergunta 16:** termos de uso comercial da Open-Meteo.
- **Pergunta 20:** web, PWA ou nativo.
- **Pergunta 26:** estados de erro e vazio.
- **Pergunta 28:** obrigação legal de acessibilidade.

Além dessas, duas decisões faltam e não estão na seção 5:

- **Dono e aprovador da entrega** (pergunta 4): sem isso, nenhuma pergunta é fechada com autoridade.
- **Estratégia de testes com a API externa:** usar respostas gravadas, mocks ou chamadas reais nos testes E2E. Afeta o Plan, mas precisa ser decidida antes das tarefas, para que os testes de erro sejam possíveis.

**Recomendação:** fechar as seis perguntas P1 e as duas decisões acima antes de escrever a Spec. As demais perguntas podem virar Open Questions dentro da própria Spec.



