# Comandos Descobertos

Anotações de comandos e comportamentos que apareceram durante a prática, além do que já está em [Containers na prática](./04-containers-na-pratica.md) e na [Referência de comandos](./06-referencia-de-comandos.md).

## `docker run -d -it ubuntu bash`

```bash
docker run -d -it ubuntu bash
```

Esse formato não entra automaticamente no container nem mantém a sessão interativa da forma esperada. O `-d` coloca o processo em segundo plano, enquanto o `-it` tenta abrir um terminal interativo — combinações que costumam confundir no começo.

Para entrar no Bash de um container Ubuntu, prefira:

```bash
docker run -it ubuntu bash
```

Para manter o container rodando em segundo plano sem entrar nele:

```bash
docker run -d ubuntu sleep 1d
```

## Manter um container rodando com `sleep`

```bash
docker run -d debian sleep 2d
```

A imagem é executada em um container que continua ativo enquanto o processo principal (`sleep`) não termina. No exemplo acima, o container fica rodando por 2 dias.

| Comando | Efeito |
| --- | --- |
| `sleep 1d` | Mantém o container ativo por 1 dia |
| `sleep 2d` | Mantém o container ativo por 2 dias |

> Um container só continua rodando enquanto existir um processo principal ativo. Sem `bash`, `nginx` ou outro processo contínuo, ele pode parar logo após ser criado.

## `docker exec -it`

```bash
docker exec -it nome_container bash
```

Executa um comando dentro de um container que já está rodando. No caso acima, abre uma sessão Bash interativa.

Correção comum na prática:

```text
docker exec -it debian bash   # errado se "debian" for o nome da imagem
docker exec -it busy_nash bash # certo — use o nome ou o ID do container
```

O argumento precisa ser o **nome** ou o **ID** do container, não o nome da imagem.

## `docker stop -t=0`

```bash
docker stop -t=0 nome_container
```

Para o container imediatamente, sem esperar o tempo padrão de encerramento.

Por padrão, o Docker espera cerca de 10 segundos antes de forçar a parada. O `-t=0` reduz esse tempo de espera para zero.

## `docker start`

```bash
docker start nome_container
```

Inicializa um container que já existe, mas está parado. Diferente de `docker run`, que sempre cria um container novo.

## `docker pause` e `docker unpause`

```bash
docker pause nome_container
docker unpause nome_container
```

| Comando | O que faz |
| --- | --- |
| `docker pause` | Congela a execução do container, mantendo os processos em memória |
| `docker unpause` | Retoma a execução do container pausado |

No `docker ps`, um container pausado aparece com status `(Paused)`.

## `docker rm --force`

```bash
docker rm --force nome_container
```

Remove o container mesmo que ele ainda esteja em execução. Equivalente a `docker rm -f`.

## Rodando em segundo plano com `-d`

```bash
docker run -d docker/example-voting-app-vote
```

Cria e executa o container sem travar o terminal. O processo principal roda em segundo plano.

## Mapeamento de portas

### Publicação automática com `-P`

```bash
docker run -d -P docker/example-voting-app-vote
```

O Docker escolhe uma porta aleatória no host e a liga à porta exposta pela imagem.

### Mapeamento manual com `-p`

```bash
docker run -d -p 3000:80 docker/example-voting-app-vote
```

```text
3000:80
```

| Porta | Significado |
| --- | --- |
| `3000` | Porta da máquina hospedeira |
| `80` | Porta dentro do container |

Acesso no navegador:

```text
http://localhost:3000
```

Correção importante: nomes de imagem no Docker Hub devem estar em minúsculas.

```bash
docker run -d -p 3000:80 Nginx   # erro
docker run -d -p 3000:80 nginx   # certo
```

## Parar todos os containers rodando

```bash
docker stop $(docker container ls -q)
```

Para todos os containers em execução na máquina de uma vez. O `docker container ls -q` retorna apenas os IDs dos containers ativos.

## Atividades práticas

1. Crie e mantenha um container com Debian em execução por 2 dias.
2. Acesse e navegue no Bash do Debian em execução no container.
3. Pause e retome a execução do container criado.
4. Inicialize um servidor web Nginx com mapeamento de portas.
5. Crie um container com a imagem `dockersamples/static-site`, mapeie as portas e acesse a aplicação no navegador.

### Resolução

#### 1. Container Debian rodando por 2 dias

```bash
docker run -d debian sleep 2d
```

Saída:

```text
docker ps
CONTAINER ID   IMAGE     COMMAND      CREATED          STATUS          PORTS     NAMES
0d31e6e85d42   debian    "sleep 2d"   13 seconds ago   Up 13 seconds             busy_nash
```

#### 2. Entrar no Bash do container

```bash
docker exec -it busy_nash bash
```

Dentro do container:

```bash
ls
touch text_text
ls
exit
```

#### 3. Pausar e retomar o container

```bash
docker pause busy_nash
docker ps
docker unpause busy_nash
docker ps
```

Status durante a pausa:

```text
0d31e6e85d42   debian   "sleep 2d"   About a minute ago   Up About a minute (Paused)   busy_nash
```

#### 4. Nginx com mapeamento de portas

```bash
docker run -d -p 3000:80 nginx
```

Verificação:

```text
docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                     NAMES
da236d657baa   nginx     "/docker-entrypoint.…"   4 seconds ago   Up 3 seconds   0.0.0.0:3000->80/tcp, [::]:3000->80/tcp   gallant_lamarr
```

Acesso: `http://localhost:3000`

#### 5. Site estático com `dockersamples/static-site`

```bash
docker run -d -p 3333:80 dockersamples/static-site
```

Verificação:

```text
docker ps
CONTAINER ID   IMAGE                       COMMAND                  CREATED              STATUS              PORTS                                              NAMES
30771cfb1524   dockersamples/static-site   "/bin/sh -c 'cd /usr…"   4 seconds ago        Up 2 seconds        443/tcp, 0.0.0.0:3333->80/tcp, [::]:3333->80/tcp   flamboyant_chaplygin
da236d657baa   nginx                       "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:3000->80/tcp, [::]:3000->80/tcp            gallant_lamarr
0d31e6e85d42   debian                      "sleep 2d"               5 minutes ago        Up 5 minutes                                                           busy_nash
```

Acesso: `http://localhost:3333`

## O que ficou claro depois da prática

| Situação | Aprendizado |
| --- | --- |
| Container para sozinho | Precisa de um processo principal contínuo, como `sleep`, `bash` ou `nginx` |
| `-d` com `-it` | Combinação confusa; escolha detached (`-d`) ou interativo (`-it`) conforme o objetivo |
| `docker pause` | Congela o container sem encerrar os processos |
