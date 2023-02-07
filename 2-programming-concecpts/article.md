# **Race Condition**
É um comportamento que acontece quando um dispositivo tenta fazer mais de uma operação ao mesmo tempo, mesmo ele só podendo realizar uma operação por vez sequencialmente. 

Geralmente esse tipo de problema é muito mais famoso no mundo da multithread no qual pode acontecer quando mais de um programa no seu dispositivo ou até mesmo threads estão disputando pelo o mesmo recurso ao mesmo tempo e isso irá ocasionar danos no processo em execução.

## Evitar race conditions
- Use **Singleton** para objetos globais
  
  Outro recurso muito interessantes é usar objetos imutáveis, como é muito usado no padrão de projeto Singleton que garante que uma vez criado não poderá mais ser alterado. Desta forma, se estou usando algum objeto de conexão ao banco de dados, configuração do sistema, acesso a métodos que interagem com dados, consigo evitar tal problema de potencialmente sobrescrever os conteúdos de tal objeto e ocasionar o crash na aplicação.

- Evitar usar estados compartilhados
  
  Ao usar uma variável que estará disponível para todo o sistema e ela poderá ser escrita e lida, haverá uma grande chance de ocasionar em uma race condition. Logo é ideal garantir que operações de escrita ocorra em processos independentes e haja um bloqueio evitando que em outras partes do sistema tente realizar tal operação.

- Usar sincronização de threads

## Referências
- [Node.js race conditions](https://www.nodejsdesignpatterns.com/blog/node-js-race-conditions/)
- [Race condition in React](https://betterprogramming.pub/fix-race-conditions-in-react-187afc87d9e)