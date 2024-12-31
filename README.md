# estrutura-repo-nest-sequelize

## 1 - Instale o Nest.js:
```bash
nest new nome-projeto
```

## 2 - Configurar Sequelize e MySQL
Instale as dependências necessárias:
```bash
npm install @nestjs/sequelize sequelize sequelize-typescript mysql2
npm install --save-dev @types/sequelize npm install --save-dev sequelize-cli npm i --save-dev dotenv
```

## 3 - Criar Estrutura da API
3.1 Criar Módulo para Filmes
Gere o módulo, serviço e controlador:
```bash
nest g module movies
nest g service movies
nest g controller movies
```

## 4 - Configurar Sequelize no Projeto
Abra o arquivo app.module.ts e configure o Sequelize para se conectar ao MySQL:
```bash
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { MoviesModule } from './movies/movies.module';
import SequelizeMovies from './movies/SequelizeMovie.model';
import { SequelizeModule } from '@nestjs/sequelize';

@Module({
  imports: [
    SequelizeModule.forRoot({
      dialect: 'mysql',
      username: process.env.DB_USER || 'root',
      password: process.env.DB_PASS || '',
      database: process.env.DB_NAME || 'database_example', // substitua o nome do banco quando for utilizar em seu projeto!
      host: process.env.DB_HOST || 'localhost',
      port: Number(process.env.DB_PORT) || 3306,
      autoLoadModels: true, // Carrega os modelos automaticamente
      synchronize: false, // Cria tabelas automaticamente (desative em produção)
      models: [SequelizeMovies],
    }),
    MoviesModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}

```

## 5 - Definir o Modelo Sequelize
No módulo movies, crie o arquivo sequelize-movies.model.ts dentro da pasta movies:
OBS: O modelo Movie neste caso vai servir apenas para mapear a tabela movies, mas não cria a tabela.
```bash
import { Table, Column, Model, DataType } from 'sequelize-typescript';

@Table({ tableName: 'movies', timestamps: false }) // Configure o nome e comportamento da tabela
export default class SequelizeMovies extends Model {
  @Column({
    type: DataType.INTEGER,
    primaryKey: true,
    autoIncrement: true,
  })
  id: number;

  @Column({
    type: DataType.STRING,
    allowNull: false,
  })
  title: string;

  @Column({
    type: DataType.STRING,
    allowNull: true,
  })
  director: string;

  @Column({
    type: DataType.DATE,
    allowNull: true,
  })
  releaseDate: Date;
}
```

## 6 - Conecte o modelo no módulo movies:
```bash
import { Module } from '@nestjs/common';
import { SequelizeModule } from '@nestjs/sequelize';
import { MoviesService } from './movies.service';
import { MoviesController } from './movies.controller';
import SequelizeMovies from './SequelizeMovie.model';

@Module({
  imports: [SequelizeModule.forFeature([SequelizeMovies])],
  providers: [MoviesService],
  controllers: [MoviesController],
})
export class MoviesModule {}

```
## 7 - Iniciar as configurações do Sequelize para gerar as tabelas:
  7.1 - Crie o arquivo .sequelizerc:
  ```bash:
const path = require('path');

module.exports = {
  'config': path.resolve(__dirname, 'dist', 'database', 'config', 'database.js'),
  'seeders-path': path.resolve(__dirname,'dist','database','seeders'),
  'migrations-path': path.resolve(__dirname,'dist','database','migrations'),
};
```

## 8 Crie essa estrutura de pastas na raiz do seu projeto:
```bash
database |
         |_
         | -- | config |-- database.ts
         |
         | -- | migrations | *.ts
         |
         | -- seeders | *.ts 
```
8.1 - Dentro da pasta config, insira os dados para fazer a conexão com o BD:
```bash
import 'dotenv/config';
import { Options } from 'sequelize';

const config: Options = {
  username: process.env.DB_USER || 'root',
  password: process.env.DB_PASS || '',
  database: process.env.DB_NAME || 'database_example', // substitua o nome do banco quando for utilizar em seu projeto!
  host: process.env.DB_HOST || 'localhost',
  port: Number(process.env.DB_PORT) || 3306,
  dialect: 'mysql',
};

export = config;
```
8.2 - Dentro da pasta migrations faça a estrutura para criar as tabelas:
```bash
import { Model, QueryInterface, DataTypes } from 'sequelize';
import { IMovie } from 'database/types/IMovie';

export default {
  up(queryInterface: QueryInterface) {
    return queryInterface.createTable<Model<IMovie>>('movies', {
      id: {
        type: DataTypes.INTEGER,
        autoIncrement: true,
        primaryKey: true,
      },
      title: {
        type: DataTypes.STRING,
        allowNull: false,
      },
      director: {
        type: DataTypes.STRING,
        allowNull: false,
      },
      realeseDate: {
        type: DataTypes.STRING,
        allowNull: false,
      },
    });
  },
};
```

8.3 - Agora vamos popular nosso banco de dados. Crie um arquivo dentro da pasta seeders com o nome por exemplo: 1-movies.ts:
OBS: Escrever dessa maneira faz com que os dados ou as tabelas do banco de dados vão em ordem. Por exemplo se o nosso banco tenha as tabelas: movies e series. Podemos escrever os seus arquivos com: 1-movies.ts e 2-series.ts dessa maneira eles irão ir em ordem. Isso vale para todos os arquivos que estão dentro da pasta `database`. 
```bash
import { QueryInterface } from 'sequelize';
export default {
  up: async (queryInterface: QueryInterface) => {
    await queryInterface.bulkInsert(
      'movies',
      [
        {
          title: 'Interestelar',
          director: 'Christopher Nolan',
          realeseDate: '2014',
        },
      ],
      {},
    );
  },
  down: async (queryInterface: QueryInterface) => {
    await queryInterface.bulkDelete('movies', {});
  },
};

```
## 9 - Dockerfile:
```bash
# Base image
FROM node:18

# Set working directory
WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Copy source code
COPY . .

# Expose application port
EXPOSE 3000

# Start the application
CMD ["npm", "run", "start:dev"]

```

## 10 - docker-compose.yml:
```bash
version: '3.8'
services:
  app:
    container_name: nest_api
    build: .
    ports:
      - 3000:3000
    volumes: 
      - ./:/app
    depends_on:
      db:
        condition: service_healthy
    environment:
      DB_USER: ${DB_USER}
      DB_PASS: ${DB_PASS}
      DB_PORT: ${DB_PORT}
      DB_HOST: ${DB_HOST}
      DB_NAME: ${DB_NAME}
    healthcheck:
      test: ["CMD", "lsof", "-t", "-i:3000"] # Caso utilize outra porta interna para o back, altere ela aqui também
      timeout: 10s
      retries: 5
  db:
    image: mysql:8.0.32
    container_name: movies_db
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASS}
      MYSQL_DATABASE: ${DB_NAME}
    restart: 'always'
    healthcheck:
      test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"] # Deve aguardar o banco ficar operacional
      timeout: 10s
      retries: 5
    cap_add:
      - SYS_NICE # Deve omitir alertas menores
```

## 11 - Comando no package.json:
Adicione esse comando dentro dos scripts do seu package.json
```bash
"db:reset": "npx sequelize-cli db:drop && npx sequelize-cli db:create && npx sequelize-cli db:migrate && npx sequelize-cli db:seed:all",
```

## 12 - Faça esse comando para levantar o containers da API e Banco de Dados:
```bash
docker-compose up --build -d
```
## 12.1 - Após os containers subirem, verifique se eles estão em funcionamento:
```bash
docker ps
```
Caso veja algo parecido com isso:
```
CONTAINER ID   IMAGE                          COMMAND                  CREATED          STATUS                      PORTS                                                  NAMES
f961617ab482   estrutura-nest-sequelize_app   "docker-entrypoint.s…"   39 minutes ago   Up 39 minutes (unhealthy)   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp              nest_api
2dae42900c3a   mysql:8.0.32                   "docker-entrypoint.s…"   39 minutes ago   Up 39 minutes (healthy)     0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   movies_db
```
Então eles estaram em funcionamento. Pode fazer a verificação da API também para ver se não ocorreu nenhum erro de conexão:
```bash
docker logs -f nest_api
```
nest_api porque é o nome do container que está nossa API.

## 13 - Certo, agora vamos acessar o container da nossa API para criar as tabelas do BD e popular ele.
Faça esse comando para acessar o container:
```bash
docker exec -it nest_api sh
```
E após esse comando digite o comando que adicionamos no package.json o `npm run db:reset`.
FIM.
