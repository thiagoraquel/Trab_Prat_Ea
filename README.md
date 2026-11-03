# Especificação Formal de Sistema de Biblioteca em Maude

Este repositório contém a especificação algébrica formal de um sistema de gerenciamento de biblioteca desenvolvido em **Maude** (`fmod BIBLIOTECA`). O modelo abstrai o acervo de livros, a base de usuários e o controle de empréstimos ativos como coleções matemáticas estruturadas sobre monóides abelianos (multiconjuntos com propriedades associativas e comutativas).

O objetivo principal da especificação é garantir a integridade das operações e do estado do sistema por meio de reescrita equacional, validações preventivas (`ceq`) e políticas de tolerância a operações inválidas (`[owise]`).

---

## Estrutura Básica do Projeto

- **Módulo Funcional:** `BIBLIOTECA` (utilizando os tipos básicos `NAT`, `STRING` e `BOOL`).
- **Abstração do Estado Global:** Reposicionado sob o construtor `sistema(ConjuntoLivros, ConjuntoUsuarios, ConjuntoEmprestimos)`.
- **Coleções AC:** As estruturas de livros, usuários e empréstimos são declaradas com os atributos `assoc comm`, eliminando a necessidade de gerenciamento manual de ordem ou índices.

---

## Features e Restrições do Sistema

### 1. Gestão de Acervo e Usuários

*   **Adição de Livros (`addLivro`)**
    *   **Feature:** Permite cadastrar novos livros no acervo com status inicial `disponivel`.
    *   **Restrição:** O cadastro exige um identificador (`Nat`) único. Tentativas de adicionar um livro com ID já existente no acervo mantêm o sistema inalterado.
*   **Adição de Usuários (`addUsuario`)**
    *   **Feature:** Cadastra novos usuários na base do sistema.
    *   **Restrição:** Exige ID (`Nat`) único por usuário.IDs duplicados são descartados sem alterar o estado.
*   **Remoção Segura de Livro (`removerLivro`)**
    *   **Feature:** Remove um livro do acervo pelo seu ID.
    *   **Restrição:** A remoção só é efetuada se o livro estiver estritamente com o estado `disponivel`. Livros atualmente emprestados não podem ser removidos.
*   **Remoção Segura de Usuário (`removerUsuario`)**
    *   **Feature:** Remove um usuário da base cadastral.
    *   **Restrição:** O usuário só pode ser removido se não possuir nenhum empréstimo pendente em seu nome (`usuarioPossuiEmprestimo == false`).

---

### 2. Empréstimos e Devoluções

*   **Realização de Empréstimo (`emprestar`)**
    *   **Feature:** Associa a posse de um livro a um usuário registrado e adiciona o par `emprestimo(IdLivro, IdUsuario)` ao conjunto de empréstimos ativos.
    *   **Restrição:** Operação válida somente se:
        1. O livro estiver cadastrado e com estado `disponivel`.
        2. O usuário estiver devidamente cadastrado na base.
        Caso contrário, a operação é bloqueada via fallback `[owise]`.
*   **Devolução de Livro (`devolver`)**
    *   **Feature:** Restaura o estado do livro para `disponivel` e remove o registro do empréstimo ativo.
    *   **Restrição:** Exige que exista a relação exata `emprestimo(L, U)` no sistema. Tentativas de devolver livros não emprestados ou associados a outro usuário falham com estado inalterado.

---

### 3. Predicados e Consultas Sobregregadas

*   **Verificação de Pertinência ao Acervo (`pertenceAcervo`)**
    *   Suporta busca por **ID** (`Nat`): verifica se o ID especificado consta no acervo.
    *   Suporta busca por **Título** (`String` via sobrecarga): checa a presença de determinado título no acervo.
*   **Status de Empréstimo (`estaEmprestado`)**
    *   Busca por **Par Livro-Usuário** (`Nat Nat`): confirma se um livro específico está alocado a um determinado usuário.
    *   Busca simplificada por **ID do Livro** (`Nat` via sobrecarga): checa se um livro consta em qualquer empréstimo ativo, independente de quem o retirou.
*   **Contadores e Métricas Globais**
    *   `totalLivros`: retorna a quantidade total de livros cadastrados.
    *   `totalEmprestados`: retorna a quantidade total de empréstimos em aberto.
    *   `totalDisponiveis`: contabiliza apenas os livros que possuem status `disponivel`.

---

## Comandos Interativos no Interpretador Maude

Para executar reduções de forma interativa dentro do ambiente do Maude, utilize os seguintes comandos:

```maude
--- Carrega o módulo funcional da biblioteca no ambiente interativo
load biblioteca.maude .

--- Insere um novo livro com ID 1 e título "Dom Casmurro" com status disponivel
red addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro") .

--- Adiciona um novo usuário com ID 10 e nome "Thiago" à base do sistema
red addUsuario(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 10, "Thiago") .

--- Associa o livro 1 ao usuário 10, atualizando o livro para emprestado e criando o registro
red emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro"), 10, "Thiago"), 1, 10) .

--- Restaura o estado do livro 1 para disponivel e remove o registro de empréstimo ativo
red devolver(emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro"), 10, "Thiago"), 1, 10), 1, 10) .

--- Retorna a quantidade total de livros atualmente cadastrados no acervo
red totalLivros(addLivro(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Livro A"), 2, "Livro B")) .

--- Verifica via sobrecarga por String se o título especificado existe no acervo
red pertenceAcervo(livro(1, "Dom Casmurro", disponivel), "Dom Casmurro") .

--- Checa via sobrecarga por Nat se o livro informado consta em algum empréstimo ativo
red estaEmprestado(emprestimo(1, 10), 1) .

--- Exclui o usuário da base cadastral após validar a ausência de empréstimos em aberto
red removerUsuario(addUsuario(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 10, "Thiago"), 10) .
```