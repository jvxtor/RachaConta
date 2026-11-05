# RachaConta — Divisão de despesas compartilhadas

**Disciplina:** Especificação & Planejamento do SDLC
**Autor:** João Victor da Silva
**Data:** 05/11/2026

---

## 1. Termo de Abertura do Projeto (Project Charter)

### Título do Projeto
**RachaConta**: aplicativo web para registrar e dividir despesas entre pessoas que moram ou convivem juntas.

### Problema Real
Estudantes em residências, repúblicas e apartamentos compartilhados dividem contas (mercado, internet, gás, água) de forma informal, por mensagens de WhatsApp ou anotações soltas. Isso gera esquecimentos, erros de cálculo, discussões sobre "quem já pagou" e falta de transparência sobre quanto cada pessoa deve.

### Proposta de Valor
Um sistema simples e rápido, que funciona bem no celular, onde qualquer morador registra uma despesa em poucos toques. O app calcula automaticamente o saldo de cada pessoa e sugere o menor número de transferências para quitar todas as dívidas do grupo.

### Escopo

**Dentro do escopo (IN) — versão 1.0**
- Cadastro e login de usuários (e-mail e senha).
- Criação de grupos e adição de membros por código de convite.
- Registro de despesas com descrição, valor, data, categoria e quem pagou.
- Divisão igualitária ou personalizada entre os participantes.
- Cálculo do saldo individual (quem deve / quem tem a receber).
- Sugestão de acertos de contas (quem paga quanto a quem).
- Listagem e filtro de despesas por categoria e período.
- Exportação do histórico em CSV.

**Fora do escopo (OUT) — versão 1.0**
- Pagamentos reais integrados (Pix, cartão) — apenas registro manual de acertos.
- Aplicativo nativo (Android/iOS); será apenas web responsivo.
- Múltiplas moedas e conversão cambial.
- Leitura de notas fiscais por OCR/IA.
- Notificações push e chat entre membros.

---

## 2. Histórias de Usuário

1. **Como** morador de uma residência estudantil, **quero** registrar uma despesa informando valor, descrição e quem pagou, **para que** todos vejam o gasto e ninguém esqueça de contribuir.
2. **Como** membro de um grupo, **quero** dividir uma despesa apenas entre as pessoas que participaram dela, **para que** ninguém pague por algo que não consumiu.
3. **Como** membro de um grupo, **quero** ver quanto devo e quanto tenho a receber, **para que** eu saiba exatamente a minha situação financeira com o grupo.
4. **Como** administrador do grupo, **quero** gerar um código de convite, **para que** novos moradores entrem sem precisar de cadastro manual feito por mim.
5. **Como** morador, **quero** receber uma sugestão de quem deve pagar quem, **para que** o acerto final use o menor número possível de transferências.
6. **Como** tesoureiro do grupo, **quero** exportar o histórico em CSV, **para que** eu possa prestar contas ou arquivar os gastos do mês.

---

## 3. Requisitos do Sistema

### Requisitos Funcionais (RF)

| ID | Descrição |
|----|-----------|
| RF01 | O sistema deve permitir cadastro, login e logout de usuários com e-mail e senha. |
| RF02 | O sistema deve permitir criar grupos e entrar em um grupo por código de convite. |
| RF03 | O sistema deve permitir cadastrar, editar e excluir despesas (descrição, valor, data, categoria, pagador e participantes). |
| RF04 | O sistema deve dividir o valor da despesa de forma igualitária ou por valores personalizados, garantindo que a soma das partes seja igual ao total. |
| RF05 | O sistema deve calcular o saldo líquido de cada membro do grupo. |
| RF06 | O sistema deve sugerir as transferências mínimas para quitar os saldos. |
| RF07 | O sistema deve permitir registrar um acerto de contas (pagamento entre dois membros), atualizando os saldos. |
| RF08 | O sistema deve filtrar despesas por categoria, membro e período. |
| RF09 | O sistema deve exportar o histórico de despesas do grupo em CSV. |

### Requisitos Não-Funcionais (RNF)

| ID | Descrição |
|----|-----------|
| RNF01 | **Desempenho:** as telas principais devem responder em até 2 segundos em conexão 4G. |
| RNF02 | **Usabilidade:** a interface deve ser responsiva e utilizável em telas a partir de 360 px de largura. |
| RNF03 | **Segurança:** senhas devem ser armazenadas com hash (bcrypt) e as rotas protegidas por token JWT. |
| RNF04 | **Persistência:** todos os dados devem ser armazenados em banco de dados relacional, sem perda após reinício do servidor. |
| RNF05 | **Compatibilidade:** funcionar nas versões recentes de Chrome, Firefox, Edge e Safari. |
| RNF06 | **Precisão:** valores monetários devem ser armazenados em centavos (inteiros) para evitar erros de ponto flutuante. |

---

## 4. Caso de Uso Principal & Tratamento de Erros

### Caso de uso: Registrar uma despesa compartilhada

**Ator:** membro autenticado de um grupo.

- **Passo 1 — Entrada do usuário:** o usuário abre o grupo, toca em "Nova despesa" e preenche descrição, valor, categoria, data, quem pagou e quais membros participam. Em seguida toca em "Salvar".
- **Passo 2 — Processamento:** o sistema valida os campos, converte o valor para centavos, divide o total entre os participantes (igualmente ou conforme valores informados, ajustando o resto de centavos), grava a despesa e recalcula os saldos do grupo.
- **Passo 3 — Retorno:** o sistema exibe uma mensagem de confirmação, adiciona a despesa no topo da lista e atualiza o painel de saldos de cada membro.

### Fluxos de exceção

| Situação | Comportamento do sistema |
|----------|--------------------------|
| Campo obrigatório vazio ou valor ≤ 0 | Bloqueia o envio e mostra mensagem de erro junto ao campo inválido. |
| Soma da divisão personalizada diferente do total | Exibe aviso com a diferença restante e impede salvar até corrigir. |
| Nenhum participante selecionado | Mostra erro pedindo ao menos um participante. |
| Queda de conexão ao salvar | Mostra "Não foi possível salvar. Tente novamente", mantém os dados preenchidos no formulário e oferece botão de nova tentativa. |
| Token expirado | Redireciona para o login e preserva o rascunho da despesa. |
| Despesa ou grupo não encontrado (404) | Exibe página de "Item não encontrado" com link para voltar ao grupo. |
| Usuário sem permissão no grupo (403) | Nega o acesso e retorna à lista de grupos do usuário. |

---

## 5. Arquitetura & Stack Tecnológica

| Camada | Tecnologia |
|--------|-----------|
| **Interface / Frontend** | React + Vite, com CSS responsivo (mobile-first) |
| **Lógica / Backend** | Node.js com Express (API REST) e validação com Zod |
| **Armazenamento** | PostgreSQL, acessado via Prisma ORM |
| **Autenticação** | JWT + bcrypt |

**Arquitetura:** SPA em React consumindo uma API REST. A API segue camadas (rotas → serviços → repositórios), e o algoritmo de cálculo de saldos fica isolado em um módulo puro, facilitando testes unitários.

### Justificativa
JavaScript/TypeScript em todo o projeto reduz a troca de contexto e permite compartilhar tipos e validações entre frontend e backend. O PostgreSQL oferece integridade relacional (grupos, membros, despesas e divisões) e transações, essenciais para manter os saldos consistentes. React com Vite garante uma interface rápida e fácil de adaptar para telas pequenas.

---

## 6. Critérios de Aceite & Definição de Pronto (DoD)

O software será considerado finalizado quando:

1. Um usuário conseguir se cadastrar, fazer login e criar um grupo sem erros.
2. Outro usuário conseguir entrar no grupo usando o código de convite.
3. Despesas puderem ser criadas, editadas e excluídas, com validação de todos os campos obrigatórios.
4. A divisão igualitária e a personalizada sempre somarem exatamente o valor total (testado com valores que geram resto de centavos, como R$ 10,00 entre 3 pessoas).
5. Os saldos exibidos baterem com o cálculo manual em pelo menos 5 cenários de teste.
6. A sugestão de acertos zerar todos os saldos do grupo com o mínimo de transferências.
7. A exportação CSV gerar um arquivo que abre corretamente no Excel/Google Planilhas.
8. A interface funcionar sem quebra de layout em tela de 360 px e nos navegadores listados no RNF05.
9. Os testes unitários do módulo de cálculo de saldos passarem com 100% de sucesso.
10. O sistema estiver publicado e acessível por uma URL, com o código no GitHub e este README atualizado.
