# Imagens, Dockerfile e Build

## Fluxo principal

```mermaid
flowchart LR
    D[Dockerfile] -->|docker build| I[Imagem]
    I -->|docker run| C[Container]
```

```text
Dockerfile = receita para criar a imagem
Imagem = ambiente pronto, mas parado
Container = imagem sendo executada
```

## Dockerfile

O `Dockerfile` é um arquivo da aplicação que funciona como receita de criação do ambiente. Ele descreve o que o Docker precisa fazer para montar a imagem.

Ele costuma declarar:

- qual imagem base usar;
- quais dependências instalar;
- quais arquivos copiar;
- qual pasta usar dentro do container;
- qual porta a aplicação usa;
- qual comando iniciar quando o container rodar.

Exemplo para Node.js:

```dockerfile
FROM node:20

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev"]
```

## Instruções mais comuns

| Instrução | Papel |
| --- | --- |
| `FROM` | Define a imagem base |
| `WORKDIR` | Define a pasta principal dentro do container |
| `COPY` | Copia arquivos da máquina para a imagem |
| `RUN` | Executa comandos durante a criação da imagem |
| `EXPOSE` | Documenta a porta usada pelo container |
| `CMD` | Define o comando padrão executado ao iniciar o container |

### `FROM`

Define a imagem base.

```dockerfile
FROM node:20
```

Outros exemplos:

```dockerfile
FROM ubuntu
FROM python:3.12
FROM nginx
FROM postgres
```

### `WORKDIR`

Define a pasta principal dentro do container.

```dockerfile
WORKDIR /app
```

### `COPY`

Copia arquivos da máquina para dentro da imagem.

```dockerfile
COPY . .
```

Também é comum copiar primeiro apenas os arquivos de dependências:

```dockerfile
COPY package*.json ./
```

### `RUN`

Executa comandos durante a criação da imagem.

```dockerfile
RUN npm install
```

Esse comando roda durante o build. Ele não roda toda vez que o container inicia.

### `EXPOSE`

Documenta a porta usada pela aplicação dentro do container.

```dockerfile
EXPOSE 3000
```

`EXPOSE` não publica a porta automaticamente na máquina. Para acessar de fora, é comum mapear a porta:

```bash
docker run -p 3000:3000 minha-api
```

### `CMD`

Define o comando padrão executado quando o container inicia.

```dockerfile
CMD ["npm", "run", "dev"]
```

Se esse processo parar, o container também para.

## Imagens

Uma imagem contém tudo que é necessário para criar um ambiente de execução.

Ela pode conter:

- sistema base;
- dependências;
- bibliotecas;
- arquivos da aplicação;
- configurações;
- comandos padrão;
- variáveis de ambiente;
- informações de porta.

Exemplos de imagens:

```bash
ubuntu
node:20
python:3.12
nginx
postgres
mysql
```

Uma imagem é um ambiente pronto, mas parado. Ela não executa nada sozinha.

## Listar imagens locais

`docker image ls` lista as imagens que já estão baixadas na máquina. Ele mostra o repositório, a tag, o ID da imagem, quando ela foi criada e o tamanho ocupado.

```bash
docker image ls
```

Saída demonstrativa:

```text
REPOSITORY              TAG       IMAGE ID       CREATED        SIZE
tenmenezes/allbooks     1.1       a1b2c3d4e5f6   2 hours ago    438MB
node                    20        9f8e7d6c5b4a   3 weeks ago    1.1GB
nginx                   latest    1a2b3c4d5e6f   1 month ago    192MB
ubuntu                  latest    6f5e4d3c2b1a   2 months ago   78MB
```

| Coluna | Significado |
| --- | --- |
| `REPOSITORY` | Nome da imagem ou repositório |
| `TAG` | Versão, variação ou etiqueta da imagem |
| `IMAGE ID` | Identificador único da imagem local |
| `CREATED` | Quando a imagem foi criada ou publicada |
| `SIZE` | Espaço aproximado ocupado pela imagem |

`docker images` é uma forma equivalente de listar imagens locais.

## Imagem e container

```text
Imagem = molde/base pronta
Container = molde/base em execução
```

Outra analogia útil:

```text
Imagem = classe
Container = objeto criado a partir da classe
```

A mesma imagem pode gerar vários containers diferentes:

```bash
docker run ubuntu
docker run ubuntu
docker run ubuntu
```

Cada execução cria um container separado.

## Camadas e cache

Imagens Docker normalmente são formadas por camadas. Cada instrução do `Dockerfile` pode gerar uma camada reutilizável.

```dockerfile
FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
```

Essa ordem ajuda o Docker a reaproveitar cache. Se o código mudar, mas o `package.json` não mudar, o Docker pode reutilizar a camada de instalação de dependências.

As camadas não são exclusivas de uma única imagem. Se duas imagens usam a mesma base ou repetem uma etapa igual, o Docker pode reaproveitar a camada já baixada ou já construída. Por isso, quando você roda `docker pull` ou `docker run` para uma imagem que ainda não existe localmente, o Docker baixa apenas as camadas que faltam.

### Histórico de camadas

`docker history` mostra as camadas que formam uma imagem.

```bash
docker history tenmenezes/allbooks:1.1
```

Saída demonstrativa:

```text
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
a1b2c3d4e5f6   2 hours ago    /bin/sh -c #(nop) CMD ["npm","run","dev"]       0B
<missing>      2 hours ago    /bin/sh -c #(nop) EXPOSE 3000                   0B
<missing>      2 hours ago    /bin/sh -c #(nop) COPY . .                      12MB
<missing>      2 hours ago    /bin/sh -c npm install                          238MB
<missing>      3 weeks ago    /bin/sh -c #(nop) WORKDIR /app                  0B
<missing>      3 weeks ago    /bin/sh -c #(nop) FROM node:20                  1.1GB
```

| Coluna | Significado |
| --- | --- |
| `IMAGE` | ID da camada ou da imagem gerada naquela etapa |
| `CREATED` | Quando aquela camada foi criada |
| `CREATED BY` | Comando ou instrução que gerou a camada |
| `SIZE` | Quanto aquela camada adicionou ao tamanho da imagem |
| `COMMENT` | Comentário opcional, normalmente vazio |

Se a ordem das instruções muda, a sequência de camadas também muda. Isso pode gerar uma nova imagem e reduzir o reaproveitamento do cache.

### Camada de escrita do container

As camadas da imagem são somente leitura. Quando um container é criado, o Docker adiciona por cima uma camada de leitura e escrita, onde ficam as alterações feitas durante a execução.

```text
Camadas da imagem = read-only
Camada do container = read-write
```

Se você cria vários containers a partir da mesma imagem, eles compartilham as camadas da imagem. Cada container precisa guardar apenas sua própria camada de escrita, o que economiza espaço.

## Inspecionar imagens

`docker inspect` mostra os metadados completos de uma imagem em formato JSON.

```bash
docker inspect tenmenezes/allbooks:1.1
```

Esse comando ajuda a consultar:

- ID completo da imagem;
- tags associadas;
- camadas usadas;
- variáveis de ambiente;
- comando padrão;
- arquitetura;
- data de criação.

## Build

`build` é o processo de criar uma imagem a partir de um `Dockerfile`.

```bash
docker build -t minha-api .
```

| Parte | Significado |
| --- | --- |
| `docker build` | Cria a imagem |
| `-t minha-api` | Define nome ou tag da imagem |
| `.` | Usa a pasta atual como contexto de build |

O `-t` define a tag da imagem. O padrão mais usado é:

```text
[registry/][usuario-ou-organizacao/]nome-da-imagem[:tag]
```

Exemplos:

```bash
docker build -t minha-api .
docker build -t minha-api:1.0 .
docker build -t tenmenezes/allbooks:1.1 .
docker build -t docker.io/tenmenezes/allbooks:1.1 .
```

| Parte | Exemplo | Uso |
| --- | --- | --- |
| `registry` | `docker.io` | Local onde a imagem será publicada ou buscada |
| `usuario-ou-organizacao` | `tenmenezes` | Dono do repositório no registry |
| `nome-da-imagem` | `allbooks` | Nome do projeto ou serviço |
| `tag` | `1.1` | Versão ou variação da imagem |

Depois de alterar o `Dockerfile`, faça um novo build. Containers já criados não mudam sozinhos; é preciso recriá-los usando a imagem atualizada.

Rodando uma imagem criada localmente:

```bash
docker run -d -p 8080:3000 tenmenezes/allbooks:1.1
```

Nesse exemplo, a porta `8080` da máquina aponta para a porta `3000` do container.

## Contexto de build

O contexto de build é a pasta que o Docker usa como base para criar a imagem.

```bash
docker build -t minha-api .
```

O `.` indica que o contexto é a pasta atual. Arquivos desse contexto podem ser copiados no `Dockerfile`:

```dockerfile
COPY . .
```

Evite deixar arquivos desnecessários no contexto:

- `node_modules`;
- `.git`;
- arquivos temporários;
- logs;
- arquivos grandes;
- arquivos sensíveis.

## `.dockerignore`

O `.dockerignore` funciona parecido com o `.gitignore`. Ele diz quais arquivos o Docker deve ignorar no build.

```dockerignore
node_modules
.git
.env
dist
coverage
```

Isso deixa o build mais leve e reduz o risco de copiar arquivos sensíveis para dentro da imagem.

## Correções importantes

| Ideia errada | Ideia correta |
| --- | --- |
| Dockerfile é uma imagem | Dockerfile é a receita da imagem |
| Container é literalmente a imagem | Container é criado a partir da imagem |
| Container roda dentro da imagem | Container roda a partir da imagem |
| Imagem executa processo | Quem executa processo é o container |

Frase final:

```text
Eu escrevo um Dockerfile para construir uma imagem.
Depois uso essa imagem para criar e rodar containers.
```
