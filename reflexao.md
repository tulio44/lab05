# Reflexão — Lab 05: RPC, REST e gRPC na Prática

## 1. Stubs e skeletons

Com base na Tarefa 2, deu para entender bem o papel do stub e do skeleton em um sistema RPC. O stub, no lado do cliente, pega a chamada que parece local, faz o marshalling dos argumentos, envia pela rede e depois recebe a resposta para transformar de volta em um dado utilizável. Já o skeleton, no lado do servidor, recebe essa mensagem, faz o unmarshalling, identifica qual método foi chamado e executa a lógica correspondente.

Esses dois componentes existem justamente para esconder a parte mais operacional da comunicação distribuída. Sem eles, cada chamada remota exigiria lidar manualmente com socket, serialização, protocolo de mensagem e tratamento de erro. Na Tarefa 2 isso ficou bem visível, porque o stub enviava um JSON com o nome do método e os argumentos, enquanto o skeleton recebia isso, fazia o dispatch e devolvia o resultado. Então, no fim, eles tornam viável a abstração de “chamada remota como se fosse local”.

---

## 2. REST não é RPC

A diferença fundamental entre REST e RPC está na forma de modelar a comunicação. No RPC, a ideia principal é invocar uma ação ou um procedimento remoto, então a comunicação gira em torno de métodos, como `calcular("soma", 10, 3)`. Já no REST, o foco não está em ações nomeadas, mas em recursos identificados por URI, manipulados por verbos HTTP como `GET`, `POST`, `PUT` e `DELETE`.

Isso apareceu de forma bem clara nas Tarefas 1 e 3. No XML-RPC, o cliente chama diretamente `proxy.calcular(...)`, o que reforça essa ideia de procedimento remoto. Na API REST, por outro lado, eu não chamo algo como `criarProduto()`: eu faço `POST /produtos`, `GET /produtos/1`, `PUT /produtos/3` e `DELETE /produtos/3`. Então o REST segue a lógica de interface uniforme, enquanto o RPC é mais orientado a operações específicas expostas pelo servidor.

---

## 3. Evolução de contrato

No gRPC, o arquivo `.proto` funciona como um contrato explícito entre cliente e servidor. Se fosse necessário adicionar um novo campo, como `unidade: string` em `RespostaCalculo`, isso poderia ser feito sem quebrar clientes antigos, desde que o novo campo fosse adicionado corretamente no schema com um novo identificador. O Protobuf já foi pensado para esse tipo de evolução compatível de contrato, então clientes antigos simplesmente ignorariam o campo que não conhecem.

No REST, isso também pode acontecer, mas de forma menos controlada. Se o servidor passar a retornar um novo campo em um JSON, muitos clientes continuarão funcionando normalmente, desde que não dependam de uma estrutura rígida. A diferença é que, no REST sem schema forte, essa compatibilidade depende mais de convenção, documentação e cuidado dos desenvolvedores. Assim, o gRPC traz uma evolução de contrato mais formal e previsível, enquanto o REST costuma ser mais flexível, mas também mais sujeito a inconsistências.

---

## 4. Escolha de tecnologia

No cenário de uma startup que precisa expor uma API de pagamentos para parceiros externos e também manter comunicação interna entre microsserviços, eu recomendaria tecnologias diferentes para cada caso. Para a API externa, REST seria a escolha mais adequada, porque HTTP e JSON são amplamente aceitos, simples de integrar e mais acessíveis para aplicações de terceiros. Além disso, os próprios códigos de status HTTP já ajudam bastante a comunicar o resultado da requisição de forma padronizada.

Para a comunicação entre os microsserviços internos, eu escolheria gRPC. Nesse contexto, faz mais sentido aproveitar o contrato forte do `.proto`, a tipagem mais rígida, a serialização binária com Protocol Buffers e o transporte via HTTP/2, que tende a ser mais eficiente. No laboratório isso ficou visível porque o gRPC ofereceu chamadas bem estruturadas e erros mais organizados, como `INVALID_ARGUMENT`. Então, de forma geral, REST faz mais sentido para integração pública e interoperabilidade, enquanto gRPC se destaca mais em comunicação interna e controlada.

---

## 5. Conexão com Labs anteriores

No Lab 04 foi discutido que transparência excessiva pode ser perigosa em sistemas distribuídos, e isso se aplica diretamente ao RPC. A grande vantagem do RPC é fazer a chamada remota parecer uma chamada local, mas isso também pode induzir o desenvolvedor a esquecer que existe rede envolvida. Quando isso acontece, fatores como latência, timeout, falhas parciais e custo de serialização podem ser ignorados no projeto.

Um exemplo de decisão ruim seria quebrar uma operação em várias chamadas remotas pequenas, como se todas tivessem o mesmo custo de chamadas locais. Na Tarefa 1, `proxy.calcular(...)` parecia algo muito simples, mas por trás havia serialização XML, transporte HTTP e execução remota no servidor. Então a transparência ajuda muito na abstração, mas pode atrapalhar quando mascara a natureza distribuída do sistema e leva a escolhas de design pouco eficientes.