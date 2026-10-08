# Diretrizes de Design e Arquitetura Web: Módulos Interativos de Sistemas Operacionais (IPC)

Este documento estabelece as diretrizes de design, arquitetura e estrutura funcional para a criação e replicação de páginas web educacionais voltadas ao ensino de conceitos de **Sistemas Operacionais**, **Comunicação entre Processos (IPC)** e **Sincronização**.

---

## 1. Identidade Visual e Temática

A interface deve adotar um tema *Dark Mode Industrial / Developer Focus*, priorizando alto contraste para leitura de código, legibilidade em simulações visuais e reduzindo a fadiga visual.

### Paleta de Cores (CSS Variables)

```css
:root {
  /* Cores de Fundo e Estrutura */
  --bg-main: #0f172a;        /* Slate 900 - Fundo principal */
  --bg-card: #1e293b;        /* Slate 800 - Superfície de cards e painéis */
  --bg-card-hover: #334155;  /* Slate 700 - Hover e destaque secundário */
  --border-color: #334155;   /* Bordas de separação */
  
  /* Cores Tipográficas */
  --text-primary: #f8fafc;   /* Slate 50 - Texto principal */
  --text-secondary: #94a3b8; /* Slate 400 - Rótulos, descrições e comentários */
  --text-muted: #64748b;     /* Slate 500 - Texto desabilitado / secundário */

  /* Cores de Destaque por Tópico */
  --accent-ipc: #38bdf8;      /* Cyan / Sky 400 - Troca de Mensagens */
  --accent-barrier: #c084fc;  /* Purple 400 - Sincronização e Barreiras */
  --accent-rcu: #34d399;      /* Emerald 400 - Concorrência Leve / RCU */
  --accent-terminal: #f59e0b; /* Amber 500 - Avisos e Terminal Logs */
}

```

### Tipografia

* **Interface e Texto Base:** `Inter`, `Segoe UI`, `system-ui`, `-apple-system`, `sans-serif`.
* **Código, Métricas e Terminal:** `JetBrains Mono`, `Fira Code`, `Consolas`, `monospace`.

---

## 2. Estrutura Modular da Página

Toda página didática deve ser construída em um único arquivo (`index.html`) e organizada em **3 pilares fundamentais**:

```
+-----------------------------------------------------------------------+
| 1. Cabeçalho & Navegação por Abas (Tabs System)                       |
+-----------------------------------------------------------------------+
| 2. Área de Simulação Interativa (Playground)                          |
|    +----------------------------------+-------------------------------+ |
|    | Controles & Diagrama Dinâmico   | Console de Terminal (Logs)    | |
|    +----------------------------------+-------------------------------+ |
+-----------------------------------------------------------------------+
| 3. Fundamentação Teórica & Código Fonte                               |
|    +----------------------------------+-------------------------------+ |
|    | Conceitos Claves & Características| Snippet de Código (C/C++)     | |
|    +----------------------------------+-------------------------------+ |
+-----------------------------------------------------------------------+

```

### Detalhamento dos Pilares

1. **Header & Navegação:**
* Título claro da disciplina/capítulo.
* Sistema de abas no topo para alternar entre os tópicos sem recarregar a página (`display: none` / `display: block` gerenciado via JavaScript).


2. **Simulação Interativa (Playground):**
* **Painel de Controle:** Botões funcionais para iniciar, pausar ou dar passos na simulação.
* **Canvas / Palco Dinâmico:** Representação visual de filas, threads, ponteiros ou memória compartilhada com transições CSS suaves (`transition: all 0.3s ease`).
* **Terminal de Eventos:** Bloco estilo shell de comando exibindo o histórico de execução em tempo real com timestamps.


3. **Fundamentação & Código:**
* Lista de pontos-chave conceituais (prós, contras, casos de uso).
* Bloco de código de referência formatado com destaque sintático e comentários explicativos.



---

## 3. Padrões de Implementação por Tópico

Ao adicionar novos tópicos de SO/IPC, siga o padrão visual e comportamental abaixo:

| Tópico | Elementos Visuais Obrigatórios | Mecânica da Simulação |
| --- | --- | --- |
| **Troca de Mensagens (Message Passing)** | • Nó Emissor (Processo A)<br>

<br>• Nó Receptor (Processo B)<br>

<br>• Fila/Buffer Central (`Queue`) | Animação de blocos de mensagens transitando até a fila e sendo consumidos pelo receptor de forma síncrona ou assíncrona. |
| **Barreiras (Synchronization Barriers)** | • N Threads paralelas<br>

<br>• Linha física da Barreira<br>

<br>• Contador de Threads retidas | Threads avançam com velocidades aleatórias, ficam no estado `WAITING` na barreira e são liberadas simultaneamente quando o limite é atingido. |
| **RCU (Read-Copy-Update)** | • Ponteiro Ativo (Dado Original)<br>

<br>• Cópia de Leitura (Lock-Free)<br>

<br>• Período de Graça (*Grace Period*) | Leitores leem continuamente do ponteiro ativo sem travar. O escritor duplica a estrutura, modifica a cópia, atualiza o ponteiro e aguarda os leitores antigos antes de liberar a memória anterior. |

---

## 4. Padrão de Animação e Terminal

### Estilo do Terminal de Logs

```css
.terminal-window {
  background-color: #020617;
  border: 1px solid var(--border-color);
  border-radius: 0.5rem;
  font-family: 'JetBrains Mono', monospace;
  padding: 1rem;
  color: #e2e8f0;
  height: 250px;
  overflow-y: auto;
}

.log-entry {
  margin-bottom: 0.25rem;
  font-size: 0.875rem;
}

.log-timestamp {
  color: var(--text-muted);
  margin-right: 0.5rem;
}

```

### Função Auxiliar de Log (JavaScript)

```javascript
function addLog(terminalId, message, type = 'info') {
  const terminal = document.getElementById(terminalId);
  const now = new Date();
  const timeStr = now.toTimeString().split(' ')[0] + '.' + String(now.getMilliseconds()).padStart(3, '0');
  
  const logLine = document.createElement('div');
  logLine.className = `log-entry log-${type}`;
  logLine.innerHTML = `<span class="log-timestamp">[${timeStr}]</span> ${message}`;
  
  terminal.appendChild(logLine);
  terminal.scrollTop = terminal.scrollHeight; // Auto-scroll para o final
}

```

---

## 5. Requisitos de Entrega e Código (Single-File)

1. **Zero Dependências Externas Opcionais:** A aplicação deve rodar nativamente em qualquer navegador moderno abrindo apenas o arquivo `.html` (sem necessidade de servidores Node.js, compiladores ou pacotes `npm`).
2. **HTML Semântico:** Uso apropriado de `<header>`, `<nav>`, `<main>`, `<section>`, e `<code>`.
3. **Responsividade Integrada:**
* Uso de Flexbox e CSS Grid.
* Em telas com largura menor que `768px`, os painéis da simulação e os blocos de código devem empilhar verticalmente.


4. **Desempenho de Animação:** Usar propriedades CSS otimizadas para GPU (`transform: translate()`, `opacity`) para manter as animações a 60 FPS durante as simulações de IPC.

```

<ElicitationsGroup message="O que deseja fazer agora?">
  <Elicitation label="Criar um modelo HTML limpo baseado nessas diretrizes" query="Crie um template starter em HTML/CSS/JS limpo seguindo exatamente essas diretrizes para novos tópicos."/>
  <Elicitation label="Adicionar um novo módulo de IPC (ex: Memória Compartilhada)" query="Crie a estrutura de um novo módulo de simulação para Memória Compartilhada e Semáforos seguindo estas diretrizes."/>
</ElicitationsGroup>

```
