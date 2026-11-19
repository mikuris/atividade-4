

## 1. O que é uma função de hash e por que não dá para voltar à senha

Uma função de hash pega um texto (a senha) e devolve uma sequência de tamanho fixo
que funciona como uma "impressão digital" dele. A mesma entrada sempre gera o mesmo
resultado, e qualquer mudança pequena na entrada muda o resultado por completo.

Não dá para desfazer porque a função é de mão única: ela mistura e descarta
informação durante o cálculo, então muitas entradas possíveis são comprimidas num
resultado de tamanho fixo e não existe a conta inversa. A única forma de "descobrir"
a senha é chutar senhas, calcular o hash de cada uma e comparar. Por isso usamos o
PBKDF2 com muitas iterações: ele torna cada chute lento, o que atrapalha quem tenta
adivinhar milhões de senhas.

## 2. O que é o salt e que problema ele resolve

O salt é um valor aleatório (aqui, 16 bytes = 32 caracteres hexadecimais) gerado
para cada usuário e misturado à senha antes do hash.

Sem salt, duas pessoas com a mesma senha teriam o mesmo hash, e um atacante poderia
usar tabelas prontas de hashes de senhas comuns (rainbow tables) ou perceber quem
repete senha. Com salt, cada usuário tem um hash diferente mesmo com senha igual, e
o atacante precisaria refazer o trabalho de adivinhação para cada usuário.

**Prova tirada do banco:** cadastrem dois usuários com a mesma senha (por exemplo
`abc123`) e rodem:

```sql
SELECT login, salt, senha_hash FROM usuarios WHERE login IN ('maria', 'joao');
```

Resultado: os dois `salt` são diferentes e, por isso, os dois `senha_hash`
também são diferentes, apesar de a senha ser a mesma.

Saída do `prova_salt.py`, que cadastra dois usuários com a mesma senha pelo `inserir` e lê o que ficou gravado:

```
Mesma senha nos dois usuários: abc123

login:      joao_prova
salt:       2a442dc2f07fca93df0398e28473de90
senha_hash: 0773f2d361829fdce19599adb1b6f2e7f03962e82022843f68e7fe5b5996fa3c

login:      maria_prova
salt:       c7dcea2ef3ba6071abf5bf82c528194c
senha_hash: ad3677fdb6b0392576463d1943192a70b89bd068951b6df0ec35e5c66cbc9573
```


## 3. Por que o salt pode ficar no banco sem ser segredo

O objetivo do salt não é esconder nada, e sim garantir que cada hash seja único. Para
conferir o login, o sistema precisa do salt para recalcular o hash da senha
digitada, então ele precisa estar guardado junto do hash.

Mesmo que um atacante veja o salt, ele continua sem a senha: ainda precisa chutar
senhas uma a uma, usando o salt de cada usuário, e não pode reaproveitar tabelas
prontas. O que protege a senha é a dificuldade de inverter o hash, não o sigilo do
salt.
