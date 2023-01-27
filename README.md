# Minicurso de Prisma

Minicurso que fiz no módulo 5 da Driven.

***
## Índice

[1. O que é uma ORM e como configurar o Prisma](#1-o-que-é-uma-orm-e-como-configurar-o-prisma)

[2. Nossa Primeira consulta findMany e findFirst](#2-nossa-primeira-consulta-findmany-e-findfirst)

[3. Inserir dados com create](#3-inserir-dados-com-create)

[4. Inserir ou atualizar - o poderoso upsert](#4-inserir-ou-alterar-com-upsert)

[5. Seed](#5-seed)

[6. Migrations](#6-migrations)

[7. Model](#7-model)

[8. Prisma Studio](#8-prisma-studio)


***
<br>

## **`1. O que é uma ORM e como configurar o Prisma`**

ORM (Object-Relational Mapping) é uma técnica de programação que permite acessar dados armazenados em um banco de dados relacional através de objetos de uma aplicação. Ele fornece uma camada de abstração entre a aplicação e o banco de dados, permitindo que a aplicação acesse os dados de forma mais natural e intuitiva, sem precisar lidar diretamente com comandos SQL.

Assim, ao invés de fazer uma consulta no banco usando uma query SQL, a ORM faz isso por nós. O Prisma é uma ORM. Por exmeplo, para inserir um novo job na tabela "jobs", não precisaríamos fazer a query que está na function "insertUnique", usamos apenas o comando 

`jobs.create({ aqui vão os dados a serem inseridos})`

Como no exemplo abaixo: 

![print de uso do Prisma](./prints/Captura%20de%20tela%20de%202023-01-24%2014-02-47.png)

**As vantagens de usar uma ORM são:**

1. Abstrair as queries SQL, ou seja, ela cria a query por você;
2. Ela otimiza as queries automaticamente;
3. Ela abstrai as diferenças entre bancos, por exemeplo, entre PostgreSQL, MySQL, etc.

<br>

### **`Configurações`**
***

O Prisma pode ser ser configurado de duas formas: 

1. Quando você ainda não possui um banco, você pode modelá-lo através do Prisma. 
2. Quando eu já tenho um banco modelado e vou só trazer o Prisma pra rodar na aplicação. 

Vamos focar na **`segunda forma`**. Para isso:

<br>

**1. No terminal, na raiz do meu projeto, eu executo o comando:**

`npx prisma init `

OBS: se você não tiver o prisma instalado, ele vai perguntar se você quer instalá-lo. 

<br>

**2. Ele vai criar uma pasta chamada prisma e um .env. 
No .env, é preciso configurar o DATABASE_URL, pois ele vem assim:**

![print do database_url padrão do prisma](./prints/Captura%20de%20tela%20de%202023-01-24%2014-31-47.png)

Para configurar:

- johndoe: seu usuário do banco,
- randompassword: senha do seu usuário,
- mydb: nome do seu banco

O restante mantém igual

Na pasta prisma, verifique se há essa configuração:

![config na pasta prisma](./prints/Captura%20de%20tela%20de%202023-01-24%2014-35-47.png)

- provider: é o tipo de banco que estamos usando, no caso, postgresql,
- url: carrega as configs do .env

<br>

**3. Para que esse passo funcione, seu banco precisa ter pelo menos 1 tabela criada. Depois que ele tiver pelo menos 1 tabela, rode o comando:**

``` npx prisma db pull```

Isso faz com o Primas leia o seu banco, escaneie as tabelas e traga elas para o prisma. Se você olhar na pasta prisma, elas vão aparecer no arquivo que está lá. 

<br>

**4. Por fim, rode o comando:**

``` npx prisma generate```

Isso cria o prismaClient e deixa a configuração pronta para ser usada. 

***
<br>

## **`2. Nossa Primeira consulta findMany e findFirst`**

A partir do momento que nós passamos a usar o Prisma, nós não usamos mais a lib pg, que fazia a conexão com o banco antes. Para fazer a nova conexão, no arquivo **`database`**:

`import pkg from '@prisma/client'`

Se não tiver o **@prisma/client** instalado: 

`npm i -D @prisma/client`  

Também verifique se no **tsconfig** há essa config: 

`"moduleResolution": "Node"`.

Depois, acrescenta essa config ao aquivo database:

```
const {PrismaClient} = pkg;
const prisma = new PrismaClient();

export default prisma;
```

No arquivo de **`repository`**:

`import prisma from "../database`

<br>

### **`Para fazer o findMany e o findFirst`**
***

Como o prisma faz a query por nós, só precisamos conectá-lo à tabela que queremos buscar. Então, ainda no arquivo de **`repository`** criamos a conexão com a tabela:

```
async function findClasses() {
    return prisma.aulas.findMany();
}
```

A função **findMany** retorna todo os resultados da tabela.

A função **findFirst** retorna o primeiro resultado da tabela. 

***
<br>

## **`3. Inserir dados com create`**

Para inserir dados usamos a função **```create```**:

![função usando create](./prints/Captura%20de%20tela%20de%202023-01-26%2012-57-02.png)

***
<br>

## **`4. Inserir ou alterar com upsert`**

A função **`upsert`** consegue verificar se nós vamos inserir algo que não existe ou apenas atualizar um dado já existente. 

A upsert precisa de pelo menos **3 parâmetros**:

- where: precisa ser um dado unique;
- create: o que você quer criar caso ainda não exista;
- update: o que você quer atualizar caso já exista;

Por exemplo:

![função usando usert](./prints/Captura%20de%20tela%20de%202023-01-26%2013-43-12.png)

Um detalhe importante é a tipagem dos dados nesse caso. 

![tipos](./prints/Captura%20de%20tela%20de%202023-01-26%2013-45-25.png)

o tipo **JobEntity** é baseado em como essas dados estão armazenados no banco, assim ele conta com a propriedade id. 

Como o id é gerado pelo próprio banco, nós precisamos do tipo **Job**, já que ele é baseado no JobEntity, mas omite o id e assim pode ser usado em funções de inserção de dados no banco, por exmeplo. 

Já o **NewJob** é um tipo que é baseado em JobEntity, mas pode receber os dados de forma parcial, inclusive aqueles que estão definidos como obrigatórios. Esse tipo vai ser usado em funções que usem o upsert, já que eu preciso que receber o id em funções assim para poder editar o objeto que corresponde àquela id. 

Eu não posso usar o tipo JobEntity para o parâmetro de funções que usem o upsert porque ele obrigatoriamente tem todos os campos e a função upsert pode ser usada para criar um dado se ele não existir, nesse caso ela não pode receber um id. 

![função usando usert](./prints/Captura%20de%20tela%20de%202023-01-26%2013-43-12.png)

Outro detalhe importante é que a propriedade usada no **where** precisa ser UNIQUE. Para não dar erro, é preciso colocar:

`id: job.id || 0 `

Porque assim, quando a gente não enviar id, ou seja, quando for criar algo, Ele vai buscar pelo id 0 que não existe, mas nõ dá erro já que não é undefined e nem null. 

***
<br>

## **`5. Seed`**

Muitas vezes quando iniciamos uma aplicação, as tabelas vem sem registro nenhum. Isso dificulta fazer testes, etc. Para evitar criar os registros "na mão", o prisma tem o seed. 

Para criar seeds:

**1. No package.json:**

```
"prisma": {
    "seed": "ts-node prisma/seed.ts"
}
```
Pode ser colocado em qualquer lugar. No exemplo do curso, foi colocado depois dos "scripts". 

**2. Na pasta "prisma":**

Cria um arquivo chamado seeds.ts;

Dentro do arquivo:

1. importa do prisma

` import prisma from "../src/database/database";`

2. Para chamar o prisma, a gente cria uma função assíncrona:

![função para criar seeds](./prints/Captura%20de%20tela%20de%202023-01-27%2009-30-08.png)

Essa função vai usar o createMany, já que vai criar mais de um registro e vai receber um "data" que é um array de objetos. 

3. Depois, vamos executar essa função:

![execução da função](./prints/Captura%20de%20tela%20de%202023-01-27%2009-38-37.png)

Como a função criada anteriormente é uma função assíncrona, ela retorna uma promise. Então, caso os registros sejam inseridos com sucesso, colocamos um console.log no then.

Caso dê erro, damos console.log no erro e encerramos o processo com `process.exit(1)`.

Por fim, para evitar vazamento de memória, é importante encerrar a conexão com o banco depois, então usamos o "finally". Esse comando é executado tanto se o then for executado quanto se o catch. O que ele está fazendo ali é desconextar o banco usando a função "disconnect" do prisma. 

4. Criamos um script no package.json para executar o seed no terminal

`"prisma:seed": "npx prisma db seed"`

5. Agora é só executar esse comando no terminal para popular o banco.

***
<br>

## **`6. Migrations`**

As migrations são semelhantes a commits. Conforme vocẽ vai alterando coisas no banco de dados, você vai criando novos arquivos migration e assim um histórico das mudanças fica disponível para quem for rodar a aplicação. 

- Para começar, digitamos o seguinte comando:

`npx prisma migrate dev`

- Depois disso, ele vai informar que precisa deletar todos os dados que anteriores. Como estamos em ambiente de desenvolvimento, podemos aceitar isso.

- Ele vai pedir para você digitar um nome para a migration (isso é como se fosse um commit, pode ser um descritivo do que você está fazendo)

Isso vai criar a migration e a partir daí outras pessoas conseguem recriar o banco da sua aplicação com facilidade. 

Caso alguma alteração seja feita no banco, se executa novamente o comando no terminal e dá-se um nome para a nova migration. 

***
<br>

## **`7. Model`**

Criamos models quando não temos um banco de dados pronto, ou seja, nós vamos modelar o banco no Prisma e refletir isso no banco. 

![model](./prints/Captura%20de%20tela%20de%202023-01-27%2016-15-50.png)

Em geral, se coloca o nome do model no singular e depois se usa `@@map("nome_da_tabela")` para dizer a qual tabela ele corresponde. 

E relação às colunas:

- id: vai ser **Int** e receber @id e @default(autoincrement()) para mostrar que é um número inteiro, que funcionará como id, ou seja, que é a ligação com outras tabelas, e que será incrementado;

- email: é uma string, mas deve ser único, então recebeu a notação **@unique**

![model com relacionamento](./prints/Captura%20de%20tela%20de%202023-01-27%2016-24-32.png)

No model User, nós temos dois relacionamentos com outras tabelas: 

Credentials Credentials[]
Network Network[]

Isso faz com que o prisma entenda e crie nos outros modelos uma coluna de UserId. Como acontece no model Credentials em:

**`user User @relation(fields: [userId], references: [id])`**

Essa linha mostra que user se relaciona com o model User através da id. 

***
<br>

## **`8. Prisma Studio`**

Essa é uma ferramenta web para ajudar com algumas pesquisas de query, etc.

Para abrir, basta rodar no terminal:

`npx prisma studio`

Ele serve para visualizar as tabelas, incluir registros, deletar registros, etc. 