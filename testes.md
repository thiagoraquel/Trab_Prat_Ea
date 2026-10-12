### Teste 1: Adicionar um livro ao acervo (Operação Básica)
* **Descrição**: Verificar se um livro é inserido corretamente com o status de disponivel
* **Termo (Comando no Maude):** `red addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "O Alquimista") .`
* **Resultado Esperado:** O sistema deve inserir o novo livro contendo o ID 1, o título "O Alquimista" e o estado disponivel dentro do conjunto de livros, mantendo usuários e empréstimos vazios.
* **Resultado Real:**
result Sistema: sistema(livro(1, "O Alquimista", disponivel), nenhumUsuario, nenhumEmprestimo)
* **Status:** ✅ Aprovado

### Teste 2: Cadastrar um usuário (Operação Básica)
* **Descrição:** Verificar a inclusão correta de um novo usuário no sistema.
* **Termo (Comando no Maude):** `red addUsuario(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 10, "Thiago") .`
* **Resultado Esperado:** O sistema deve registrar o usuário com o ID 10 e o nome "Thiago" no conjunto de usuários, mantendo o acervo e os empréstimos vazios.
* **Resultado Real:**
result Sistema: sistema(nenhumLivro, usuario(10, "Thiago"), nenhumEmprestimo)
* **Status:** ✅ Aprovado.

### Teste 3: Realizar empréstimo válido (Operação Composta / Regra de Negócio)
* **Descrição:** Adicionar livro, adicionar usuário e realizar o empréstimo com sucesso, mudando o estado do livro para emprestado e gerando o registro.
* **Termo (Comando no Maude):** `red emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro"), 10, "Thiago"), 1, 10) .`
* **Resultado Esperado:** O sistema deve executar a adição do livro, do usuário e em seguida aplicar a equação de empréstimo, alterando o estado do livro de disponivel para emprestado e inserindo o termo emprestimo(1, 10) no conjunto de empréstimos em 7 passos de reescrita.
* **Resultado Real:**
rewrites: 7 in 0ms cpu (0ms real)
result Sistema: sistema(livro(1, "Dom Casmurro", emprestado), usuario(10, "Thiago"), emprestimo(1, 10))
* **Status:** ✅ Aprovado.

### Teste 4: Realizar a devolução de um livro (Regra de Negócio)
* **Descrição:** Emprestar um livro e em seguida realizar a devolução, fazendo o livro voltar ao estado disponivel e limpando o registro de empréstimo.
* **Termo (Comando no Maude):** `red devolver(emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro"), 10, "Thiago"), 1, 10), 1, 10) .`
* **Resultado Esperado:** O sistema deve processar a cadeia completa (adição, empréstimo e devolução), fazendo o livro retornar ao estado disponivel e removendo o registro correspondente do conjunto de empréstimos em 8 passos de reescrita.
* **Resultado Real:**
rewrites: 8 in 0ms cpu (0ms real)
result Sistema: sistema(livro(1, "Dom Casmurro", disponivel), usuario(10, "Thiago"), nenhumEmprestimo)
* **Status:** ✅ Aprovado.

### Teste 5: Consulta de quantidade total de livros (Operação de Consulta)
* **Descrição:** Verificar se a função de contagem retorna corretamente o total de livros no acervo.
* **Termo (Comando no Maude):** `red totalLivros(addLivro(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Livro A"), 2, "Livro B")) .`
* **Resultado Esperado:** O sistema deve adicionar dois livros distintos ao acervo e a função de contagem (totalLivros) deve processar a estrutura recursivamente, retornando o número total de livros cadastrados (2) em 12 passos de reescrita.
* **Resultado Real:**
rewrites: 12 in 0ms cpu (2ms real)
result NzNat: 2
* **Status:** ✅ Aprovado.

### Teste 6: Consulta de livros disponíveis após empréstimo (Propriedade de Cardinalidade)
* **Descrição:** Comprovar a regra de que o total de livros é a soma dos disponíveis com os emprestados após movimentações.
* **Termo (Comando no Maude):** `red totalDisponiveis(emprestar(addUsuario(addLivro(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Livro A"), 2, "Livro B"), 10, "Thiago"), 1, 10)) .`
* **Resultado Esperado:** Após adicionar dois livros, um usuário e realizar o empréstimo de um dos livros, a função totalDisponiveis deve contar apenas os livros que permaneceram com o estado disponivel, retornando 1 em 15 passos de reescrita.
* **Resultado Real:**
rewrites: 15 in 0ms cpu (0ms real)
result NzNat: 1
* **Status:** ✅ Aprovado

### Teste 7: Caso Inválido 1 — Emprestar livro que já está emprestado
* **Descrição:** Garantir que o sistema bloqueie (via [owise]) a tentativa de emprestar o mesmo livro duas vezes, mantendo o estado inalterado.
* **Termo (Comando no Maude):** `red emprestar(emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro"), 10, "Thiago"), 1, 10), 1, 10) .`
* **Resultado Esperado:** O sistema deve executar a adição e o primeiro empréstimo com sucesso. Na segunda tentativa de empréstimo do mesmo livro (que agora está com estado emprestado), a equação válida falhará, caindo na regra de fallback [owise], que bloqueia a ação e mantém o sistema no estado anterior em 8 passos de reescrita.
* **Resultado Real:**
rewrites: 8 in 0ms cpu (1ms real)
result Sistema: sistema(livro(1, "Dom Casmurro", emprestado), usuario(10, "Thiago"), emprestimo(1, 10))
* **Status:** ✅ Aprovado.

### Teste 8: Caso Inválido 2 — Adicionar livro com ID duplicado
* **Descrição:** Testar a equação condicional (ceq) que impede o cadastro de dois livros com o mesmo identificador único.
* **Termo (Comando no Maude):** `red addLivro(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Primeiro Título"), 1, "Título Duplicado") .`
* **Resultado Esperado:** O sistema deve inserir o primeiro livro com sucesso. Ao tentar adicionar um segundo livro com o mesmo ID 1, a condição de unicidade avaliada por pertenceAcervo retornará verdadeira, acionando a equação que descarta a segunda inserção e mantém apenas o livro original no acervo em 8 passos de reescrita.
* **Resultado Real:**
rewrites: 8 in 0ms cpu (0ms real)
result Sistema: sistema(livro(1, "Primeiro Ttulo", disponivel), nenhumUsuario, nenhumEmprestimo)
* **Status:** ✅ Aprovado.

### Teste 9: Verificar se um usuário possui determinado livro (Consulta de Relação)
* **Descrição:** Testar o predicado booleano usuarioPossuiLivro para validar o vínculo entre usuário e livro emprestado.
* **Termo (Comando no Maude):** `red usuarioPossuiLivro(emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro"), 10, "Thiago"), 1, 10), 1, 10) .`
* **Resultado Esperado:** O sistema deve processar a adição do livro, do usuário, efetuar o empréstimo e avaliar o predicado de relação, retornando o valor booleano true em 9 passos de reescrita.
* **Resultado Real:**
rewrites: 9 in 0ms cpu (0ms real)
result Bool: true
* **Status:** ✅ Aprovado.

### Teste 10: Remover um livro disponível do acervo (Operação de Limpeza / Fronteira)
* **Descrição:** Testar a remoção de um livro que cumpre a regra de estar estritamente disponível.
* **Termo (Comando no Maude):** `red removerLivro(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Livro Descartável"), 1) .`
* **Resultado Esperado:** O sistema deve adicionar o livro ao acervo e, em seguida, executar a operação de remoção válida (já que o livro está disponivel), retornando um sistema com o acervo vazio (nenhumLivro) em 4 passos de reescrita.
* **Resultado Real:**
rewrites: 4 in 0ms cpu (0ms real)
result Sistema: sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo)
* **Status:** ✅ Aprovado.