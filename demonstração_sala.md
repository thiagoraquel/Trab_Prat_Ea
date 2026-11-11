No cmd (wsl), coloque:
maude
load bibliote.maude .
Em seguida rode as demonstrações.

1. Adicionar um livro válido ao acervo do zero
red addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Dom Casmurro") .

2. Demonstrar o bloqueio de ID duplicado (Regra inválida / ceq)
red addLivro(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "Livro A"), 1, "Livro B") .

3. Realizar um empréstimo completo ponta a ponta
red emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "O Hobbit"), 10, "Maria"), 1, 10) .

4. Testar a sobrecarga de busca de livro por Título (String)
red pertenceAcervo(livro(42, "O Pequeno Príncipe", disponivel), "O Pequeno Príncipe") .

5. Demonstrar a segurança na remoção de usuários com pendências (ceq)
red removerUsuario(emprestar(addUsuario(addLivro(sistema(nenhumLivro, nenhumUsuario, nenhumEmprestimo), 1, "1984"), 10, "Carlos"), 1, 10), 10) .