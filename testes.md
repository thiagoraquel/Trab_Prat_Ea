# Documentação da Suíte de Testes Integrada

Esta documentação reúne todos os casos de teste da especificação formal do módulo `BIBLIOTECA`.

## Execução Automatizada

Para rodar toda a bateria de testes de uma só vez no terminal:

```bash
maude biblioteca.maude testes.maude
```

---

## Casos de Teste

### Teste 1: Adicionar um livro ao acervo (Operação Básica)
- **Descrição:** Verificar se um livro é inserido corretamente com o status `disponivel`.
- **Termo:** `red addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "O Alquimista") .`
- **Resultado Esperado:** `result Sistema: sistema(livro(1, "O Alquimista", disponivel), nenhumUsuario, nenhumEmprestimo)`
- **Status:** ✅ Aprovado

### Teste 2: Cadastrar um usuário (Operação Básica)
- **Descrição:** Verificar a inclusão correta de um novo usuário no sistema.
- **Termo:** `red addUsuario(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 10, "Thiago") .`
- **Resultado Esperado:** `result Sistema: sistema(nenhumLivro, usuario(10, "Thiago"), nenhumEmprestimo)`
- **Status:** ✅ Aprovado

### Teste 3: Realizar empréstimo válido (Operação Composta)
- **Descrição:** Adicionar livro, adicionar usuário e realizar o empréstimo com sucesso, mudando o estado do livro para `emprestado` e gerando o registro.
- **Termo:** `red emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro"), 10, "Thiago"), 1, 10) .`
- **Resultado Esperado:** `result Sistema: sistema(livro(1, "Dom Casmurro", emprestado), usuario(10, "Thiago"), emprestimo(1, 10))`
- **Status:** ✅ Aprovado

### Teste 4: Realizar a devolução de um livro (Regra de Negócio)
- **Descrição:** Emprestar um livro e em seguida devolver, fazendo o livro voltar ao estado `disponivel` e removendo o registro.
- **Termo:** `red devolver(emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro"), 10, "Thiago"), 1, 10), 1, 10) .`
- **Resultado Esperado:** `result Sistema: sistema(livro(1, "Dom Casmurro", disponivel), usuario(10, "Thiago"), nenhumEmprestimo)`
- **Status:** ✅ Aprovado

### Teste 5: Consulta de quantidade total de livros (Operação de Consulta)
- **Descrição:** Verificar se a função de contagem retorna corretamente o total de livros no acervo.
- **Termo:** `red totalLivros(addLivro(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Livro A"), 2, "Livro B")) .`
- **Resultado Esperado:** `result NzNat: 2`
- **Status:** ✅ Aprovado

### Teste 6: Consulta de livros disponíveis após empréstimo
- **Descrição:** Comprovar a regra de cardinalidade em movimentações.
- **Termo:** `red totalDisponiveis(emprestar(addUsuario(addLivro(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Livro A"), 2, "Livro B"), 10, "Thiago"), 1, 10)) .`
- **Resultado Esperado:** `result NzNat: 1`
- **Status:** ✅ Aprovado

### Teste 7: Caso Inválido 1 — Emprestar livro que já está emprestado
- **Descrição:** Bloqueio via `[owise]` de tentativa de re-emprestar livro ocupado.
- **Termo:** `red emprestar(emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro"), 10, "Thiago"), 1, 10), 1, 10) .`
- **Resultado Esperado:** Sistema inalterado após falha da primeira regra.
- **Status:** ✅ Aprovado

### Teste 8: Caso Inválido 2 — Adicionar livro com ID duplicado
- **Descrição:** Testar a equação condicional (`ceq`) que impede IDs duplicados.
- **Termo:** `red addLivro(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Primeiro Titulo"), 1, "Titulo Duplicado") .`
- **Resultado Esperado:** Mantém apenas o livro original no acervo.
- **Status:** ✅ Aprovado

### Teste 9: Verificar se usuário possui livro (Predicado)
- **Descrição:** Validar a relação entre usuário e livro emprestado.
- **Termo:** `red usuarioPossuiLivro(emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro"), 10, "Thiago"), 1, 10), 1, 10) .`
- **Resultado Esperado:** `result Bool: true`
- **Status:** ✅ Aprovado

### Teste 10: Remover livro disponível (Limpeza)
- **Descrição:** Testar a remoção de um livro disponível.
- **Termo:** `red removerLivro(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Livro Descartavel"), 1) .`
- **Resultado Esperado:** `result Sistema: sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo)`
- **Status:** ✅ Aprovado

---

## Novos Casos de Teste (Features Recentes)

### Teste 11: Remoção Segura de Usuário sem Pendências (Feature A)
- **Descrição:** Garantir que um usuário sem empréstimos seja removido com sucesso do sistema.
- **Termo:** `red removerUsuario(addUsuario(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 10, "Thiago"), 10) .`
- **Resultado Esperado:** `result Sistema: sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo)`
- **Status:** ✅ Aprovado

### Teste 12: Bloqueio de Remoção de Usuário com Empréstimo Ativo (Feature A)
- **Descrição:** Garantir que um usuário com empréstimo pendente não possa ser removido.
- **Termo:** `red removerUsuario(emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro"), 10, "Thiago"), 1, 10), 10) .`
- **Resultado Esperado:** Retorna o sistema inalterado (com o usuário mantido).
- **Status:** ✅ Aprovado

### Teste 13: Busca de Pertinência por Título (Discussão 1)
- **Descrição:** Testar a sobrecarga de `pertenceAcervo` buscando por `String`.
- **Termo:** `red pertenceAcervo(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro"), "Dom Casmurro") .`
- **Resultado Esperado:** `result Bool: true`
- **Status:** ✅ Aprovado

### Teste 14: Checagem Simplificada de Empréstimo por ID do Livro (Discussão 2)
- **Descrição:** Testar a sobrecarga de `estaEmprestado` recebendo apenas o ID do livro.
- **Termo:** `red estaEmprestado(emprestimo(1, 10), 1) .`
- **Resultado Esperado:** `result Bool: true`
- **Status:** ✅ Aprovado