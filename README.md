# Comandos Avançados no Git

Este documento apresenta comandos avançados do Git, organizados por grupos e cenários práticos de uso, focados no dia a dia de desenvolvedores web/app (backend/fullstack node).

## Grupo 01: Git Rebase

O rebase é uma das ferramentas mais poderosas do Git, permitindo reescrever o histórico de commits, integrar alterações de outras branches e manter um histórico linear e limpo. Diferente do merge, que cria um commit de mesclagem, o rebase reaplica os commits de uma branch sobre outra, resultando em um histórico mais organizado.

### Subcomandos mais utilizados:
- `git rebase <branch>` - Reaplica os commits da branch atual sobre a branch especificada
- `git rebase -i <commit>` - Modo interativo, permite editar, combinar, reordenar ou remover commits
- `git rebase --continue` - Continua o rebase após resolver conflitos
- `git rebase --abort` - Cancela o rebase e retorna ao estado anterior
- `git rebase --skip` - Pula o commit atual durante o rebase

Para mais opções, consulte a [documentação oficial do git rebase](https://git-scm.com/docs/git-rebase).

### Cenário 01.01: Atualizar sua branch de feature com as mudanças da main

Um dos cenários mais comuns no desenvolvimento é quando você está trabalhando em uma branch de feature por alguns dias e, enquanto isso, a branch principal (main) recebeu novos commits. Para manter sua branch atualizada e evitar conflitos futuros, o rebase é uma excelente opção.

```bash
# Estando na sua branch de feature
git checkout feature/nova-api-usuarios
# Primeiro, atualize sua branch main local
git checkout main
git pull origin main
# Volte para sua branch de feature e faça o rebase
git checkout feature/nova-api-usuarios
git rebase main
```

Exemplo de saída ao executar o rebase:

```
$ git rebase main
First, rewinding head to replay your work on top of it...
Applying: Adiciona estrutura básica da API de usuários
Applying: Implementa endpoint de criação de usuário
Applying: Adiciona validação de dados de entrada
```

Se ocorrerem conflitos, o Git pausará o rebase e mostrará uma mensagem como:

```
$ git rebase main
First, rewinding head to replay your work on top of it...
Applying: Adiciona estrutura básica da API de usuários
Using index info to reconstruct a base tree...
M       src/routes/users.js
Falling back to patching base and 3-way merge...
Auto-merging src/routes/users.js
CONFLICT (content): Merge conflict in src/routes/users.js
error: Failed to merge in the changes.
Patch failed at 0001 Adiciona estrutura básica da API de usuários
The copy of the patch that failed is found in: .git/rebase-apply/patch

When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".
```

Este processo reaplica seus commits da feature sobre o estado atual da main, como se você tivesse começado a desenvolver a partir da versão mais recente. Se ocorrerem conflitos, o Git pausará o rebase para que você os resolva manualmente.

No contexto de um projeto Node.js, isso é particularmente útil quando, por exemplo, dependências no package.json foram atualizadas na main e você precisa dessas atualizações na sua branch de feature.

### Cenário 01.02: Limpando o histórico de commits antes de um pull request

Quando trabalhamos em uma feature, frequentemente fazemos vários commits pequenos e com mensagens pouco descritivas como "fix", "wip" ou "ajustes". Antes de submeter um pull request, é uma boa prática consolidar esses commits em unidades lógicas com mensagens claras.

```bash
# Supondo que você queira reorganizar os últimos 5 commits
git rebase -i HEAD~5
```

Isso abrirá um editor com uma lista dos últimos 5 commits, onde você pode:
- Usar `pick` para manter o commit como está
- Usar `squash` ou `fixup` para combinar com o commit anterior
- Usar `reword` para manter o commit mas editar sua mensagem
- Usar `edit` para pausar o rebase naquele commit e permitir alterações
- Usar `drop` para remover o commit

Por exemplo, em um projeto React, você pode ter vários commits relacionados a um único componente. Com o rebase interativo, você pode consolidá-los em um único commit com uma mensagem clara como "Implementa componente de autenticação com suporte a OAuth".

### Cenário 01.03: Exemplo visual do rebase interativo

Quando você executa `git rebase -i HEAD~5`, o Git abrirá um editor com conteúdo semelhante a este:

```
pick abc1234 Adiciona estrutura inicial do componente de login
pick def5678 Implementa validação de formulário
pick ghi9101 Corrige bug na validação de email
pick jkl1121 Adiciona testes para o formulário
pick mno3141 Ajusta estilos do botão de submit

# Rebase 7890abc..mno3141 onto 7890abc (5 commands)
#
# Commands:
# p, pick <commit> = use commit as is
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# u, update-ref <ref> = track a placeholder for the <ref> to be updated
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
```

Você pode editar este arquivo para reorganizar seus commits. Por exemplo:

```
pick abc1234 Adiciona estrutura inicial do componente de login
squash def5678 Implementa validação de formulário
squash ghi9101 Corrige bug na validação de email
pick jkl1121 Adiciona testes para o formulário
reword mno3141 Ajusta estilos do botão de submit
```

Neste exemplo:
- Mantemos o primeiro commit como está
- Combinamos o segundo e terceiro commits com o primeiro, criando um único commit com todas as implementações de validação
- Mantemos o commit de testes separado
- Mantemos o último commit, mas vamos editar sua mensagem para algo mais descritivo

Após salvar e fechar o editor, o Git processará os commits conforme suas instruções e abrirá novos editores para as mensagens de commit que precisam ser editadas. Para o squash, você verá algo como:

```
# This is a combination of 3 commits.
# This is the 1st commit message:

Adiciona estrutura inicial do componente de login

# This is the commit message #2:

Implementa validação de formulário

# This is the commit message #3:

Corrige bug na validação de email

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
```

Você pode editar esta mensagem para criar uma descrição unificada do commit combinado.

### Cenário 01.04: Removendo um commit específico do meio do histórico

Às vezes, você pode precisar remover um commit específico que está no meio do histórico, como quando acidentalmente commitou credenciais ou arquivos grandes que não deveriam estar no repositório.

```bash
# Identifique o hash do commit anterior ao que você quer remover
git rebase -i <hash-do-commit>
# No editor, localize o commit indesejado e mude "pick" para "drop"
# Salve e feche o editor
```

Por exemplo, se você acidentalmente commitou o arquivo `.env` com credenciais de banco de dados em um projeto NestJS, pode usar o rebase interativo para remover esse commit específico sem afetar os commits posteriores.

É importante notar que reescrever o histórico com rebase deve ser evitado em branches públicas que outros desenvolvedores já baixaram, pois isso pode causar problemas de sincronização para a equipe.

## Grupo 02: Git Cherry-pick

O cherry-pick é uma ferramenta que permite selecionar commits específicos de uma branch e aplicá-los em outra. É como se você pegasse apenas as "cerejas" (commits) que deseja, deixando o resto para trás. Isso é particularmente útil quando você precisa aplicar uma correção específica sem trazer todas as alterações de uma branch.

### Subcomandos mais utilizados:
- `git cherry-pick <commit>` - Aplica as alterações do commit especificado na branch atual
- `git cherry-pick -n <commit>` - Aplica as alterações sem criar um novo commit (--no-commit)
- `git cherry-pick --continue` - Continua o cherry-pick após resolver conflitos
- `git cherry-pick --abort` - Cancela o cherry-pick e retorna ao estado anterior
- `git cherry-pick <commit1> <commit2>` - Aplica múltiplos commits em sequência

Para mais opções, consulte a [documentação oficial do git cherry-pick](https://git-scm.com/docs/git-cherry-pick).

### Cenário 02.01: Adicionar um commit de outra branch remota na sua branch local atual

Imagine que você está trabalhando em uma branch de feature e descobre que um colega implementou uma função em outra branch que seria útil para o seu trabalho. Em vez de mesclar toda a branch, você pode usar cherry-pick para trazer apenas esse commit específico.

```bash
# Primeiro, atualize suas referências remotas
git fetch origin
# Identifique o hash do commit desejado
git log origin/feature/outra-funcionalidade
# Aplique o commit na sua branch atual
git cherry-pick abc1234
```

Exemplo de saída do comando `git log` para identificar o commit:

```
$ git log origin/feature/outra-funcionalidade
commit abc1234def5678abc1234def5678abc1234def5 (origin/feature/outra-funcionalidade)
Author: Colega <colega@empresa.com>
Date:   Mon May 15 14:30:45 2025 -0300

    Implementa função de validação de JWT

commit 567890abcdef567890abcdef567890abcdef5678
Author: Colega <colega@empresa.com>
Date:   Mon May 15 11:20:30 2025 -0300

    Adiciona dependências para autenticação
```

Exemplo de saída ao executar o cherry-pick:

```
$ git cherry-pick abc1234
[feature/sua-feature abc1234] Implementa função de validação de JWT
 1 file changed, 25 insertions(+), 0 deletions(-)
```

Por exemplo, em um projeto Node.js, um colega pode ter implementado uma função de validação de JWT em outra branch que você precisa usar na sua implementação de autenticação. Com cherry-pick, você traz apenas essa implementação específica.

## Grupo 03: Git Stash

O git stash permite salvar temporariamente alterações que ainda não foram commitadas, deixando seu diretório de trabalho limpo. É como guardar seu trabalho em progresso em uma gaveta para poder retomá-lo mais tarde. Isso é especialmente útil quando você precisa mudar rapidamente de contexto, como alternar entre branches ou aplicar um hotfix urgente.

### Subcomandos mais utilizados:
- `git stash` - Salva as alterações não commitadas e limpa o diretório de trabalho
- `git stash save "mensagem"` - Salva as alterações com uma descrição personalizada
- `git stash list` - Lista todos os stashes salvos
- `git stash apply` - Aplica o stash mais recente sem removê-lo da lista
- `git stash pop` - Aplica o stash mais recente e o remove da lista
- `git stash drop` - Remove o stash mais recente sem aplicá-lo
- `git stash show` - Mostra as alterações no stash
- `git stash branch <nome-da-branch>` - Cria uma nova branch a partir do stash

Para mais opções, consulte a [documentação oficial do git stash](https://git-scm.com/docs/git-stash).

### Cenário 03.01: Interromper o trabalho para resolver um bug urgente

Você está no meio da implementação de uma feature quando surge um bug crítico que precisa ser corrigido imediatamente. Você não quer perder seu trabalho atual, mas também não quer commitá-lo incompleto.

```bash
# Salve seu trabalho atual com uma descrição
git stash save "Implementação parcial do sistema de notificações"
# Verifique se o diretório está limpo
git status
# Mude para a branch principal para corrigir o bug
git checkout main
# Crie uma branch para o hotfix
git checkout -b hotfix/erro-login
# Após corrigir e commitar o bug, volte para sua branch original
git checkout feature/notificacoes
# Recupere seu trabalho salvo
git stash pop
```

Exemplo de saída ao salvar um stash:

```
$ git stash save "Implementação parcial do sistema de notificações"
Saved working directory and index state On feature/notificacoes: Implementação parcial do sistema de notificações
```

Exemplo de saída do `git status` após o stash:

```
$ git status
On branch feature/notificacoes
nothing to commit, working tree clean
```

Exemplo de saída ao aplicar o stash com `git stash pop`:

```
$ git stash pop
On branch feature/notificacoes
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   src/services/notification.service.js
        modified:   src/controllers/notification.controller.js

no changes added to commit (use "git add" and/or "git commit -a")
Dropped refs/stash@{0} (a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2)
```

Em um projeto NestJS, por exemplo, você pode estar implementando um novo módulo de notificações quando descobre um bug crítico na autenticação. O stash permite que você pause seu trabalho, corrija o bug e depois retome exatamente de onde parou.

### Cenário 03.02: Gerenciar múltiplos stashes com nomes descritivos

Em projetos complexos, você pode precisar alternar entre várias tarefas e manter múltiplos stashes. Usar nomes descritivos facilita a identificação do conteúdo de cada stash.

```bash
# Salve diferentes conjuntos de alterações
git stash save "Refatoração do componente de login"
git stash save "Novos endpoints para API de produtos"
git stash save "Correções de estilo no dashboard"
# Liste todos os stashes
git stash list
# Aplique um stash específico usando seu índice
git stash apply stash@{1}
# Ou crie uma branch a partir de um stash específico
git stash branch feature/novos-endpoints stash@{1}
```

Exemplo de saída do `git stash list`:

```
$ git stash list
stash@{0}: On feature/dashboard: Correções de estilo no dashboard
stash@{1}: On feature/produtos: Novos endpoints para API de produtos
stash@{2}: On feature/auth: Refatoração do componente de login
```

Exemplo de saída ao aplicar um stash específico:

```
$ git stash apply stash@{1}
On branch feature/dashboard
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   src/controllers/product.controller.js
        modified:   src/routes/product.routes.js
        modified:   src/services/product.service.js

no changes added to commit (use "git add" and/or "git commit -a")
```

Exemplo de saída ao criar uma branch a partir de um stash:

```
$ git stash branch feature/novos-endpoints stash@{1}
Switched to a new branch 'feature/novos-endpoints'
On branch feature/novos-endpoints
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   src/controllers/product.controller.js
        modified:   src/routes/product.routes.js
        modified:   src/services/product.service.js

no changes added to commit (use "git add" and/or "git commit -a")
Dropped refs/stash@{1} (f1e2d3c4b5a6f1e2d3c4b5a6f1e2d3c4b5a6f1e2)
```

Em um projeto fullstack com React e Node.js, você pode estar trabalhando simultaneamente em diferentes partes do sistema. Usar stashes nomeados ajuda a organizar essas diferentes linhas de trabalho e retomá-las quando apropriado.

É importante lembrar que, por padrão, o git stash não inclui arquivos não rastreados. Para incluí-los, use `git stash -u` ou `git stash --include-untracked`.

## Grupo 04: Resolução de Conflitos

Conflitos ocorrem quando o Git não consegue mesclar automaticamente alterações de diferentes branches porque as mesmas linhas foram modificadas de maneiras diferentes. Resolver conflitos é uma habilidade essencial para qualquer desenvolvedor que trabalhe em equipe.

### Técnicas mais utilizadas:
- Identificação de conflitos com `git status`
- Visualização de diferenças com `git diff`
- Edição manual dos arquivos conflitantes
- Uso de ferramentas visuais com `git mergetool`
- Marcação de resolução com `git add`
- Continuação do processo com `git merge --continue` ou `git rebase --continue`
- Abortar o processo com `git merge --abort` ou `git rebase --abort`

### Cenário 04.01: Resolver conflitos durante um merge

O cenário mais comum de conflito ocorre durante a mesclagem de branches, como quando você tenta mesclar sua branch de feature na main ou vice-versa.

```bash
# Tente mesclar a branch main na sua branch de feature
git checkout feature/autenticacao
git merge main
# Se ocorrerem conflitos, o Git informará
# Verifique quais arquivos estão em conflito
git status
# Abra os arquivos conflitantes e edite-os
# Os conflitos são marcados com <<<<<<< HEAD, ======= e >>>>>>> main
# Após resolver os conflitos, marque-os como resolvidos
git add arquivo-com-conflito.js
# Continue o merge
git merge --continue
# Ou crie um commit de merge diretamente
git commit -m "Merge main into feature/autenticacao"
```

Exemplo de saída ao tentar fazer um merge com conflitos:

```
$ git merge main
Auto-merging src/components/auth/Login.js
CONFLICT (content): Merge conflict in src/components/auth/Login.js
Automatic merge failed; fix conflicts and then commit the result.
```

Exemplo de saída do `git status` durante um conflito:

```
$ git status
On branch feature/autenticacao
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   src/components/auth/Login.js

no changes added to commit (use "git add" and/or "git commit -a")
```

Exemplo de como um arquivo com conflito aparece:

```javascript
// src/components/auth/Login.js
import React, { useState } from 'react';

function Login() {
<<<<<<< HEAD
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [rememberMe, setRememberMe] = useState(false);
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    // Lógica de autenticação com remember me
    // ...
  };
=======
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    // Nova lógica de autenticação com OAuth
    // ...
  };
>>>>>>> main

  return (
    // ...
  );
}
```

Em um projeto React, por exemplo, você e outro desenvolvedor podem ter modificado o mesmo componente de autenticação de maneiras diferentes. Durante o merge, você precisará decidir quais alterações manter, combinar ou adaptar.

### Cenário 04.02: Resolver conflitos durante um rebase

Os conflitos durante um rebase são semelhantes aos de um merge, mas ocorrem para cada commit que está sendo reaplicado, o que pode tornar o processo mais complexo.

```bash
# Inicie o rebase
git checkout feature/pagamentos
git rebase main
# Se ocorrerem conflitos, o Git pausará o rebase
# Resolva os conflitos em cada arquivo
# Após resolver, marque como resolvido
git add arquivo-resolvido.js
# Continue o rebase
git rebase --continue
# Se mais conflitos surgirem, repita o processo
# Se quiser cancelar o rebase
git rebase --abort
```

Exemplo de saída ao iniciar um rebase com conflitos:

```
$ git rebase main
First, rewinding head to replay your work on top of it...
Applying: Implementa sistema de pagamentos
Using index info to reconstruct a base tree...
M       src/services/payment.service.js
Falling back to patching base and 3-way merge...
Auto-merging src/services/payment.service.js
CONFLICT (content): Merge conflict in src/services/payment.service.js
error: Failed to merge in the changes.
Patch failed at 0001 Implementa sistema de pagamentos
The copy of the patch that failed is found in: .git/rebase-apply/patch

When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".
```

Exemplo de saída ao continuar o rebase após resolver conflitos:

```
$ git add src/services/payment.service.js
$ git rebase --continue
Applying: Implementa sistema de pagamentos
Applying: Adiciona testes para pagamentos
```

Em um projeto NestJS, por exemplo, você pode estar rebasing sua branch de feature que implementa um novo sistema de pagamentos sobre a main, que recebeu atualizações no modelo de dados. Você precisará resolver os conflitos para garantir que seu código funcione com o novo modelo.

### Cenário 04.03: Usando git mergetool para resolver conflitos visualmente

O Git oferece suporte a ferramentas visuais para resolução de conflitos, que podem tornar o processo mais intuitivo, especialmente para conflitos complexos.

```bash
# Configure sua ferramenta de merge preferida (exemplo com VSCode)
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Quando encontrar conflitos durante um merge ou rebase
git mergetool
```

Exemplo de saída ao executar o mergetool:

```
$ git mergetool
Merging:
src/components/auth/Login.js

Normal merge conflict for 'src/components/auth/Login.js':
  {local}: modified file
  {remote}: modified file
Hit return to start merge resolution tool (vscode):
```

A ferramenta visual mostrará as diferentes versões do arquivo (sua versão, a versão da outra branch e a base comum) e permitirá que você escolha quais partes manter ou como combiná-las.

Após resolver todos os conflitos com a ferramenta visual, continue o processo de merge ou rebase normalmente:

```bash
# Para merge
git merge --continue

# Para rebase
git rebase --continue
```

### Cenário 04.04: Abortando operações de merge ou rebase com conflitos

Às vezes, ao encontrar conflitos complexos, pode ser melhor abortar a operação atual e repensar sua estratégia.

```bash
# Para abortar um merge
git merge --abort

# Para abortar um rebase
git rebase --abort
```

Exemplo de saída ao abortar um merge:

```
$ git merge --abort
$ git status
On branch feature/autenticacao
nothing to commit, working tree clean
```

Exemplo de saída ao abortar um rebase:

```
$ git rebase --abort
$ git status
On branch feature/pagamentos
nothing to commit, working tree clean
```

Isso restaurará seu repositório ao estado anterior à operação, permitindo que você tente uma abordagem diferente, como:
- Fazer commits menores e mais focados
- Coordenar melhor com outros desenvolvedores
- Usar cherry-pick em vez de merge/rebase
- Refatorar o código para reduzir sobreposições

### Cenário 04.05: Usando git diff para entender conflitos

Antes de resolver conflitos, é útil entender exatamente o que mudou em cada versão. O comando `git diff` pode ajudar a visualizar essas diferenças.

```bash
# Durante um conflito, para ver as diferenças entre sua versão e a versão que está sendo mesclada
git diff

# Para ver as diferenças específicas em um arquivo
git diff -- caminho/para/arquivo

# Para comparar com a versão base comum (ancestral)
git diff --base -- caminho/para/arquivo

# Para ver o que você mudou em relação à base
git diff --ours -- caminho/para/arquivo

# Para ver o que a outra branch mudou em relação à base
git diff --theirs -- caminho/para/arquivo
```

Exemplo de saída do `git diff` durante um conflito:

```
$ git diff -- src/components/auth/Login.js
diff --cc src/components/auth/Login.js
index abc1234,def5678..0000000
--- a/src/components/auth/Login.js
+++ b/src/components/auth/Login.js
@@@ -1,13 -1,13 +1,19 @@@
  import React, { useState } from 'react';
  
  function Login() {
++<<<<<<< HEAD
 +  const [email, setEmail] = useState('');
 +  const [password, setPassword] = useState('');
 +  const [rememberMe, setRememberMe] = useState(false);
 +  
 +  const handleSubmit = async (e) => {
 +    e.preventDefault();
 +    // Lógica de autenticação com remember me
 +    // ...
 +  };
++=======
+   const [username, setUsername] = useState('');
+   const [password, setPassword] = useState('');
+   
+   const handleSubmit = async (e) => {
+     e.preventDefault();
+     // Nova lógica de autenticação com OAuth
+     // ...
+   };
++>>>>>>> main
```

Entender essas diferenças ajuda a tomar decisões informadas sobre como resolver os conflitos, especialmente em bases de código complexas como aplicações React ou NestJS com muitas dependências entre componentes.

## Grupo 05: Organização de Remotes

A organização de remotes no Git permite gerenciar múltiplos repositórios remotos, facilitando a colaboração em diferentes ambientes e com diferentes equipes. Isso é especialmente útil em cenários de multi-repo, forks, ou quando você precisa sincronizar seu código com diferentes servidores.

### Subcomandos mais utilizados:
- `git remote add <nome> <url>` - Adiciona um novo repositório remoto
- `git remote -v` - Lista todos os repositórios remotos configurados
- `git remote show <nome>` - Mostra informações detalhadas sobre um remote específico
- `git remote rename <antigo> <novo>` - Renomeia um repositório remoto
- `git remote remove <nome>` - Remove um repositório remoto
- `git remote set-url <nome> <url>` - Altera a URL de um repositório remoto
- `git remote set-url --add --push <nome> <url>` - Adiciona uma URL de push para um remote

Para mais opções, consulte a [documentação oficial do git remote](https://git-scm.com/docs/git-remote).

### Cenário 05.01: Configurar múltiplos remotes para push simultâneo

Em alguns projetos, você pode precisar manter o código sincronizado em diferentes servidores Git, como GitHub e GitLab, ou em um servidor interno da empresa e um serviço de hospedagem externo.

```bash
# Adicione o remote principal (convencionalmente chamado de origin)
git remote add origin git@github.com:empresa/projeto.git
# Adicione um segundo remote
git remote add gitlab git@gitlab.com:empresa/projeto.git
# Configure um remote que envia para ambos os servidores
git remote add all git@github.com:empresa/projeto.git
git remote set-url --add --push all git@github.com:empresa/projeto.git
git remote set-url --add --push all git@gitlab.com:empresa/projeto.git
# Agora você pode enviar para ambos os remotes com um único comando
git push all main
```

Exemplo de saída ao listar os remotes configurados:

```
$ git remote -v
all     git@github.com:empresa/projeto.git (fetch)
all     git@github.com:empresa/projeto.git (push)
all     git@gitlab.com:empresa/projeto.git (push)
gitlab  git@gitlab.com:empresa/projeto.git (fetch)
gitlab  git@gitlab.com:empresa/projeto.git (push)
origin  git@github.com:empresa/projeto.git (fetch)
origin  git@github.com:empresa/projeto.git (push)
```

Exemplo de saída ao fazer push para múltiplos remotes:

```
$ git push all main
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 328 bytes | 328.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To github.com:empresa/projeto.git
   abc1234..def5678  main -> main
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 328 bytes | 328.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To gitlab.com:empresa/projeto.git
   abc1234..def5678  main -> main
```

Isso é útil, por exemplo, quando sua empresa mantém um repositório interno para desenvolvimento, mas também usa o GitHub para colaboração com contribuidores externos ou para integração com serviços de CI/CD.

### Cenário 05.02: Trabalhar com forks e repositórios upstream

Quando você contribui para projetos open source ou usa uma estrutura de forks em sua organização, precisa manter seu fork sincronizado com o repositório original (upstream).

```bash
# Clone seu fork
git clone git@github.com:seu-usuario/projeto-open-source.git
cd projeto-open-source
# Adicione o repositório original como upstream
git remote add upstream git@github.com:projeto-original/projeto-open-source.git
# Verifique os remotes configurados
git remote -v
# Para atualizar seu fork com as mudanças do upstream
git fetch upstream
git checkout main
git merge upstream/main
# Ou use rebase para manter um histórico linear
git rebase upstream/main
# Envie as atualizações para seu fork
git push origin main
```

Exemplo de saída ao adicionar e verificar o remote upstream:

```
$ git remote add upstream git@github.com:projeto-original/projeto-open-source.git
$ git remote -v
origin  git@github.com:seu-usuario/projeto-open-source.git (fetch)
origin  git@github.com:seu-usuario/projeto-open-source.git (push)
upstream        git@github.com:projeto-original/projeto-open-source.git (fetch)
upstream        git@github.com:projeto-original/projeto-open-source.git (push)
```

Exemplo de saída ao atualizar seu fork com as mudanças do upstream:

```
$ git fetch upstream
remote: Enumerating objects: 75, done.
remote: Counting objects: 100% (75/75), done.
remote: Compressing objects: 100% (42/42), done.
remote: Total 75 (delta 35), reused 70 (delta 30), pack-reused 0
Unpacking objects: 100% (75/75), 14.52 KiB | 1.32 MiB/s, done.
From github.com:projeto-original/projeto-open-source
 * [new branch]      feature/nova-funcionalidade -> upstream/feature/nova-funcionalidade
 * [new branch]      hotfix/correcao-bug         -> upstream/hotfix/correcao-bug
   abc1234..def5678  main                        -> upstream/main

$ git checkout main
Already on 'main'
Your branch is up to date with 'origin/main'.

$ git merge upstream/main
Updating abc1234..def5678
Fast-forward
 README.md                 | 15 +++++++++------
 package.json              |  8 +++++---
 src/components/Button.js  | 25 +++++++++++++++++++++++++
 3 files changed, 39 insertions(+), 9 deletions(-)
 create mode 100644 src/components/Button.js
```

Esse fluxo é comum em projetos Node.js open source, onde você faz um fork do repositório original, implementa suas alterações em uma branch e depois cria um pull request para o projeto original.

### Cenário 05.03: Gerenciar diferentes ambientes com diferentes remotes

Em alguns projetos, você pode ter diferentes ambientes (desenvolvimento, staging, produção) hospedados em diferentes servidores ou organizações Git.

```bash
# Configure remotes para diferentes ambientes
git remote add dev git@github.com:empresa/projeto-dev.git
git remote add staging git@github.com:empresa/projeto-staging.git
git remote add production git@github.com:empresa/projeto-prod.git
# Envie código para um ambiente específico
git push dev feature/nova-funcionalidade
# Promova código de um ambiente para outro
git fetch staging
git checkout main
git merge staging/main
git push production main
```

Exemplo de saída ao configurar e listar múltiplos remotes para diferentes ambientes:

```
$ git remote add dev git@github.com:empresa/projeto-dev.git
$ git remote add staging git@github.com:empresa/projeto-staging.git
$ git remote add production git@github.com:empresa/projeto-prod.git
$ git remote -v
dev         git@github.com:empresa/projeto-dev.git (fetch)
dev         git@github.com:empresa/projeto-dev.git (push)
production  git@github.com:empresa/projeto-prod.git (fetch)
production  git@github.com:empresa/projeto-prod.git (push)
staging     git@github.com:empresa/projeto-staging.git (fetch)
staging     git@github.com:empresa/projeto-staging.git (push)
```

Em um projeto React/Node.js, por exemplo, você pode ter diferentes configurações de build e deploy para cada ambiente. Usar remotes separados facilita o gerenciamento dessas diferentes configurações e o controle de quais alterações vão para cada ambiente.

## Grupo 06: Git Tag

Tags no Git são referências que apontam para pontos específicos no histórico do Git, geralmente usadas para marcar versões de lançamento (v1.0.0, v2.0.0, etc.). Diferente das branches, as tags não mudam, tornando-as ideais para marcar commits importantes como releases.

### Subcomandos mais utilizados:
- `git tag` - Lista todas as tags disponíveis
- `git tag <nome>` - Cria uma tag leve no commit atual
- `git tag -a <nome> -m "mensagem"` - Cria uma tag anotada com mensagem
- `git tag -a <nome> <commit>` - Cria uma tag anotada em um commit específico
- `git show <nome-da-tag>` - Mostra informações sobre a tag
- `git push origin <nome-da-tag>` - Envia uma tag específica para o remote
- `git push origin --tags` - Envia todas as tags para o remote
- `git tag -d <nome-da-tag>` - Remove uma tag local
- `git push origin :refs/tags/<nome-da-tag>` - Remove uma tag remota

Para mais opções, consulte a [documentação oficial do git tag](https://git-scm.com/docs/git-tag).

### Cenário 06.01: Criando tags para releases de versão

Quando você finaliza uma versão do seu software, é uma boa prática marcar esse ponto no histórico com uma tag.

```bash
# Certifique-se de estar no commit que deseja marcar
git checkout main
# Crie uma tag anotada (recomendado para releases)
git tag -a v1.0.0 -m "Versão 1.0.0 - Primeiro release estável"
# Envie a tag para o repositório remoto
git push origin v1.0.0
```

Exemplo de saída ao criar uma tag anotada:

```
$ git tag -a v1.0.0 -m "Versão 1.0.0 - Primeiro release estável"
```

Exemplo de saída ao listar as tags:

```
$ git tag
v0.9.0
v1.0.0
```

Exemplo de saída ao enviar a tag para o repositório remoto:

```
$ git push origin v1.0.0
Enumerating objects: 1, done.
Counting objects: 100% (1/1), done.
Writing objects: 100% (1/1), 184 bytes | 184.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:empresa/projeto.git
 * [new tag]         v1.0.0 -> v1.0.0
```

As tags anotadas armazenam informações adicionais como o autor, data e mensagem, tornando-as mais adequadas para releases do que tags leves.

Em um projeto Node.js, por exemplo, você pode sincronizar as tags do Git com a versão no package.json, garantindo consistência entre o código fonte e a versão publicada no npm.

### Cenário 06.02: Criando tags retroativas para commits anteriores

Às vezes, você pode precisar adicionar uma tag a um commit anterior, como quando percebe que um commit específico representa um marco importante que deveria ter sido marcado.

```bash
# Encontre o hash do commit que deseja marcar
git log --oneline
# Crie uma tag para esse commit específico
git tag -a v0.9.0 abc1234 -m "Versão beta que foi enviada para testes"
# Envie a tag para o repositório remoto
git push origin v0.9.0
```

Exemplo de saída do `git log --oneline`:

```
$ git log --oneline
def5678 (HEAD -> main, tag: v1.0.0) Adiciona documentação completa
abc1234 Implementa todas as funcionalidades principais
9876fed Corrige bugs críticos reportados pelos testadores
5432cba Adiciona testes automatizados
fedcba9 Configuração inicial do projeto
```

Exemplo de saída ao criar uma tag para um commit anterior:

```
$ git tag -a v0.9.0 abc1234 -m "Versão beta que foi enviada para testes"
```

Isso é útil, por exemplo, quando você precisa referenciar uma versão específica que foi enviada para testes ou quando está documentando retroativamente o histórico de desenvolvimento do projeto.

### Cenário 06.03: Usando tags para checkout em versões específicas

As tags facilitam o checkout em versões específicas do código, o que é útil para reproduzir bugs, comparar comportamentos entre versões ou criar branches de hotfix a partir de releases anteriores.

```bash
# Visualize todas as tags disponíveis
git tag
# Faça checkout em uma tag específica
git checkout v1.2.0
# Você estará em um estado "detached HEAD"
# Para fazer alterações, crie uma branch a partir deste ponto
git checkout -b hotfix/v1.2.1
```

Exemplo de saída ao fazer checkout em uma tag:

```
$ git checkout v1.2.0
Note: switching to 'v1.2.0'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at abc1234 Implementa todas as funcionalidades para a versão 1.2.0
```

Exemplo de saída ao criar uma branch a partir de uma tag:

```
$ git checkout -b hotfix/v1.2.1
Switched to a new branch 'hotfix/v1.2.1'
```

Por exemplo, se um cliente reporta um bug na versão 1.2.0 de sua API NestJS, você pode facilmente fazer checkout nessa versão específica, reproduzir o problema e criar uma branch de hotfix a partir desse ponto.

É importante notar que fazer checkout em uma tag coloca o repositório em um estado "detached HEAD", o que significa que quaisquer commits feitos não pertencerão a nenhuma branch. Por isso, é recomendado criar uma nova branch se precisar fazer alterações.

## Exemplos Complexos de Resolução de Problemas

### Exemplo 1: Recuperação e limpeza de uma branch desatualizada

Imagine que você voltou de férias e precisa retomar o trabalho em uma branch de feature que ficou desatualizada por semanas. A main avançou significativamente e há conflitos complexos a serem resolvidos.

```bash
# Salve qualquer trabalho em progresso
git stash save "Trabalho em progresso antes da atualização"
# Atualize todas as referências remotas
git fetch --all
# Verifique quanto sua branch está atrás da main
git checkout feature/sua-feature
git log --oneline feature/sua-feature..origin/main
# Crie uma branch de backup antes de fazer alterações
git branch feature/sua-feature-backup
# Rebase interativo para limpar sua branch antes de integrar com a main
git rebase -i HEAD~5  # Ajuste o número conforme necessário
# Agora faça o rebase com a main
git rebase origin/main
# Se houver muitos conflitos complexos, pode ser melhor abortar
git rebase --abort
# E tentar uma abordagem diferente: cherry-pick seus commits em uma nova branch
git checkout -b feature/sua-feature-nova origin/main
git log --oneline feature/sua-feature-backup
# Cherry-pick seletivamente os commits relevantes
git cherry-pick abc1234 def5678
# Recupere seu trabalho em progresso se necessário
git stash pop
```

Exemplo de saída ao verificar quanto sua branch está atrás da main:

```
$ git log --oneline feature/sua-feature..origin/main
def5678 Atualiza dependências de segurança
abc1234 Implementa novo sistema de autenticação
9876fed Refatora serviço de notificações
5432cba Adiciona suporte para internacionalização
fedcba9 Corrige bugs no sistema de pagamentos
```

Esta abordagem é útil em projetos de longa duração, como uma aplicação NestJS com múltiplos módulos, onde diferentes equipes trabalham em diferentes partes do sistema e a integração pode se tornar complexa.

### Exemplo 2: Refatoração de código com preservação de histórico

Você precisa refatorar uma parte significativa do código, movendo arquivos entre diretórios e renomeando-os, mas quer preservar o histórico de commits para facilitar futuras investigações com `git blame`.

```bash
# Crie uma branch para a refatoração
git checkout -b refactor/reorganizacao-estrutura
# Use git mv para mover/renomear arquivos (preserva histórico)
git mv src/components/auth/Login.js src/features/authentication/LoginForm.js
git mv src/utils/helpers.js src/common/utilities.js
# Faça as alterações necessárias no código
# Commit as mudanças com mensagem descritiva
git commit -m "Refatora estrutura de diretórios para padrão de features"
# Rebase interativo para organizar commits relacionados
git rebase -i HEAD~10
# Resolva conflitos que surgirem
# Teste tudo antes de mesclar na main
git checkout main
git merge refactor/reorganizacao-estrutura
```

Exemplo de saída ao mover arquivos com `git mv`:

```
$ git mv src/components/auth/Login.js src/features/authentication/LoginForm.js
$ git mv src/utils/helpers.js src/common/utilities.js
$ git status
On branch refactor/reorganizacao-estrutura
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        renamed:    src/components/auth/Login.js -> src/features/authentication/LoginForm.js
        renamed:    src/utils/helpers.js -> src/common/utilities.js
```

Esta abordagem é particularmente útil em projetos React que cresceram organicamente e precisam ser reorganizados para seguir padrões mais modernos como Atomic Design ou arquitetura baseada em features.
