<img width="652" height="157" alt="image" src="https://github.com/user-attachments/assets/4c1f2843-ef74-44d3-80aa-6e0d0c35a099" />


# Além do JWT: Como o Redis Pode Fortalecer a Segurança das Sessões

 Olá pessoal, tudo bem? Hoje vou falar sobre um estudo que realizei enquanto eu estava estudando sobre sessões e formas de proteger um pouco nossas aplicações quando acontece algum phishing e é capturado nosso token onde contém dados importantes.

## O que seria o banco de dados Redis e o armazenamento em memória

Imagine que você tem uma mochila e um armário. A mochila é muito mais rápida para pegar qualquer coisa, mas cabe menos itens. O armário guarda muito mais coisas, porém leva mais tempo para abrir.

O Redis funciona como essa mochila. Ele guarda informações diretamente na memória RAM do servidor, em vez de armazená-las primeiro no disco. Por isso, consegue responder em poucos milissegundos.

Como tudo fica na memória, o Redis é extremamente rápido. Isso faz dele uma ótima opção para guardar informações que precisam ser acessadas o tempo todo, como sessões de usuários, cache, códigos temporários e contadores.

Na prática, o Redis não foi criado para substituir bancos como MySQL ou PostgreSQL. Seu objetivo é armazenar dados temporários e acelerar aplicações que precisam responder rapidamente.

## Explicação do JWT e por que ele é o mais utilizado

Quando um usuário faz login, o sistema precisa lembrar que ele já foi autenticado. Uma das formas mais populares de fazer isso é utilizando um JWT (JSON Web Token).

O JWT funciona como um crachá. Depois que você entra em uma empresa, recebe um crachá e só precisa mostrá-lo para entrar nas salas. O sistema faz algo parecido com o token.

Esse token possui informações como o identificador do usuário, permissões e uma assinatura digital que impede alterações no seu conteúdo sem invalidá-lo.

Ele se tornou muito utilizado porque é simples, rápido e não precisa consultar o banco de dados a cada requisição. Basta verificar se a assinatura é válida.

## O risco do uso do JWT e o vazamento do token que pode possuir dados importantes

 Agora imagine que alguém encontrou seu crachá da empresa. Enquanto ele continuar válido, essa pessoa poderá entrar em todos os lugares que o crachá permite.

O mesmo acontece com um JWT. Se um invasor conseguir copiar o token por meio de phishing, malware ou outro ataque, ele poderá se passar pelo usuário até que o token expire.

Mesmo quando o JWT não possui informações extremamente sensíveis, ele pode revelar dados como ID do usuário, nome, e-mail, perfil ou permissões. Além disso, ele representa uma credencial válida de acesso.

Outro problema é que o JWT normalmente não pode ser revogado imediatamente. Se ele foi emitido para durar uma hora, por exemplo, continuará funcionando durante esse período, mesmo após o vazamento.

## Onde o Redis entra para ajudar

Em vez de entregar todas as informações da sessão para o navegador, podemos entregar apenas um identificador aleatório, chamado de ID de sessão.

Quando o usuário faz login, o sistema cria um código completamente aleatório, como um UUID, e salva todas as informações da sessão dentro do Redis.

O navegador recebe somente esse identificador. Sempre que fizer uma requisição, enviará apenas esse código para o servidor.

O servidor procura esse ID no Redis. Se existir, recupera todos os dados da sessão. Se não existir, significa que a sessão foi encerrada ou revogada.

## Estratégia utilizando Redis e sessão com HTTP-Only Cookies

Uma estratégia muito utilizada é armazenar o ID da sessão em um cookie configurado como HttpOnly.

Esse tipo de cookie não pode ser acessado por JavaScript, reduzindo bastante os riscos de ataques como Cross-Site Scripting (XSS).

Se um atacante capturar apenas o ID da sessão, ele não conseguirá descriptografar nada, porque esse identificador não contém informações do usuário. Ele é apenas uma chave para localizar os dados armazenados no Redis.

Além disso, como todas as informações ficam centralizadas no Redis, basta remover aquela sessão para desconectar o usuário imediatamente em todos os dispositivos. Isso oferece um controle muito maior do que depender apenas da expiração de um JWT.

Em uma arquitetura moderna, o fluxo normalmente funciona assim:

O usuário realiza o login.
O servidor cria um ID de sessão aleatório.
Todas as informações da sessão são armazenadas no Redis.
O navegador recebe apenas um cookie HttpOnly contendo o ID da sessão.
A cada requisição, o servidor consulta o Redis usando esse identificador.
Caso seja necessário encerrar o acesso, basta apagar a sessão do Redis.

Essa abordagem reduz a exposição de informações, permite revogar sessões instantaneamente e dificulta o uso de credenciais roubadas em ataques de phishing ou vazamentos de dados. Nenhuma solução elimina completamente os riscos, mas manter os dados da sessão no servidor e expor apenas um identificador aleatório é uma estratégia bastante eficaz para aumentar a segurança da aplicação.

