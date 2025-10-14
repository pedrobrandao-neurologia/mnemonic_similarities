# Teste de Similaridade Mnemônica (MST)

## Visão Geral

Este projeto é uma implementação web do Teste de Similaridade Mnemônica (Mnemonic Similarity Test - MST), uma tarefa neuropsicológica projetada para avaliar a capacidade do cérebro de discriminar entre memórias muito semelhantes. Essa função, conhecida como **separação de padrões** (*pattern separation*), é fortemente associada ao funcionamento do **giro denteado** no hipocampo.

O teste é construído como uma única página HTML que utiliza JavaScript para toda a lógica e Tailwind CSS para a estilização, tornando-o responsivo, moderno e fácil de implantar.

### Principais Características

* **Design Responsivo:** Interface limpa e funcional em desktops, tablets e dispositivos móveis, graças ao Tailwind CSS.
* **Lógica Robusta:** O fluxo do teste é controlado por funções `async/await`, garantindo precisão na temporização e evitando bugs comuns em implementações baseadas em `setTimeout`.
* **Geração Balanceada de Estímulos:** O algoritmo garante que as listas de estudo e reconhecimento sejam geradas de forma balanceada e aleatória, um requisito crucial para a validade científica do teste.
* **Métricas Detalhadas:** Calcula e exibe métricas essenciais como o Índice de Discriminação de Lures (LDI), taxas de acerto e alarmes falsos.
* **Exportação de Dados:** Permite que os dados brutos e os resultados sejam exportados nos formatos CSV e JSON para análise posterior.
* **Configuração Flexível:** Parâmetros como duração dos estímulos e número de *trials* podem ser facilmente ajustados no objeto `CONFIG` no código.

---

## Como o Teste Funciona

O teste é dividido em duas fases principais para o participante: **Estudo** e **Reconhecimento**.

### Fase 1: Estudo (Codificação)

1.  **Tarefa:** Ao participante é apresentada uma série de imagens (neste caso, emojis) de objetos comuns, uma de cada vez.
2.  **Objetivo:** Para cada objeto, o participante deve realizar uma tarefa de orientação semântica: decidir se o objeto é tipicamente encontrado **DENTRO** ou **FORA** de casa.
3.  **Propósito:** Esta tarefa força o participante a prestar atenção e processar cada estímulo, garantindo que ele seja codificado na memória. A instrução é para memorizar os objetos, mas a tarefa principal (dentro/fora) ajuda a manter o engajamento.

### Fase 2: Reconhecimento (Recuperação)

1.  **Tarefa:** Após a fase de estudo, o participante vê uma nova série de imagens. Esta lista é composta por três tipos de estímulos:
    * **Iguais (*Targets*):** Objetos que são *exatamente os mesmos* vistos na fase de estudo.
    * **Semelhantes (*Lures*):** Objetos que são *parecidos, mas não idênticos* aos da fase de estudo (ex: uma maçã vermelha 🍎 vs. uma maçã verde 🍏).
    * **Novos (*Foils*):** Objetos completamente novos, que não têm relação com os da fase de estudo.
2.  **Objetivo:** Para cada imagem, o participante deve classificá-la como **IGUAL**, **SEMELHANTE** ou **NOVO**.
3.  **Propósito:** Esta é a fase crítica do teste. A capacidade de identificar corretamente os itens "Semelhantes" (e não confundi-los com os "Iguais") é o que mede a separação de padrões.

---

## Lógica Técnica

### 1. Geração dos Estímulos

O script primeiro gera as listas de *trials* de forma controlada:
1.  Uma base de dados principal (`STIMULI_DB`) contém pares de estímulos `target` (alvo) e `lure` (semelhante).
2.  A lista de estudo é criada selecionando aleatoriamente um número de `targets` definido em `CONFIG.encodingTrials`.
3.  A lista de reconhecimento é montada com `1/3` de *targets* (repetidos da fase de estudo), `1/3` de *lures* (os pares correspondentes aos *targets*) e `1/3` de *foils* (estímulos novos, que não apareceram de forma alguma na fase de estudo).
4.  Finalmente, a lista de reconhecimento é embaralhada para garantir que os tipos de estímulo apareçam em ordem aleatória.

### 2. Fluxo do Teste

O controle do teste é gerenciado por uma máquina de estados simples e funções assíncronas (`async/await`):
* A função `runTest()` orquestra a execução sequencial das fases.
* Cada fase (`runEncodingPhase`, `runRecognitionPhase`) executa um loop através de sua lista de *trials*.
* A função `runTrial()` é o coração do processo. Para cada *trial*, ela executa a seguinte sequência:
    1.  Mostra a cruz de fixação (`CONFIG.fixationDuration`).
    2.  Esconde a fixação e mostra o estímulo.
    3.  Inicia um temporizador e aguarda a resposta do usuário (clique ou tecla).
    4.  Registra a resposta, o tempo de reação (RT) e outros dados.
    5.  Esconde o estímulo e aguarda o intervalo inter-estímulo (`CONFIG.isi`).
    6.  Resolve a `Promise`, permitindo que o loop prossiga para o próximo *trial*.

### 3. Métricas Principais Explicadas

* **Índice de Discriminação de Lures (LDI):** A métrica mais importante. É calculada como a proporção de *lures* corretamente identificados como "Semelhante" menos a proporção de *foils* (novos) incorretamente identificados como "Igual" (*Foil False Alarm*). Um LDI alto indica uma boa capacidade de separação de padrões.
* **Taxa de Acertos (*Hit Rate*):** A proporção de itens "Iguais" (*targets*) que foram corretamente identificados como "Igual". Mede a memória de reconhecimento básica.
* **Discriminação de Semelhantes:** A proporção de itens "Semelhantes" (*lures*) corretamente identificados como "Semelhante".
* **Erro com Semelhantes (*Lure False Alarm*):** A proporção de itens "Semelhantes" que foram incorretamente classificados como "Iguais". Um valor alto aqui sugere uma falha na separação de padrões.
* **Falso Alarme (*Foil False Alarm*):** A proporção de itens "Novos" incorretamente classificados como "Iguais". Mede a tendência geral a falsas memórias.

---

## Como Executar

1.  Salve o código fornecido em um arquivo com a extensão `.html` (por exemplo, `index.html`).
2.  Abra este arquivo em qualquer navegador web moderno (Chrome, Firefox, Edge, Safari).
3.  O teste será iniciado automaticamente.

## Como Customizar

* **Parâmetros do Teste:** Para alterar tempos, durações ou o número de *trials*, modifique as propriedades do objeto `CONFIG` no início do `<script>`.
    ```javascript
    const CONFIG = {
        fixationDuration: 500,
        isi: 500,
        encodingTrials: 64,
        recognitionTrials: 96
    };
    ```
* **Estímulos:** Para usar suas próprias imagens, edite os arrays `STIMULI_DB` e `FOILS_DB`. Você pode substituir os emojis por URLs de imagens e modificar o HTML para exibir tags `<img>` em vez de texto.
