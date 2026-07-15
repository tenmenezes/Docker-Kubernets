# Referência de Comandos Docker

## Inspeção

| Comando | Uso |
| --- | --- |
| `docker ps` | Lista containers rodando |
| `docker ps -a` | Lista todos os containers, inclusive parados |
| `docker images` ou `docker image ls` | Lista imagens baixadas |
| `docker logs <container>` | Mostra logs do container |
| `docker logs -f <container>` | Acompanha logs em tempo real |
| `docker inspect <container>` | Mostra detalhes completos do container |
| `docker inspect <image>` | Mostra metadados completos de uma imagem |

## Containers

| Comando | Uso |
| --- | --- |
| `docker run hello-world` | Testa a instalação do Docker |
| `docker run ubuntu` | Cria container Ubuntu, mas pode parar rapidamente |
| `docker run -it ubuntu bash` | Cria container Ubuntu e entra no Bash |
| `docker run --name meu-ubuntu -it ubuntu bash` | Cria container com nome definido |
| `docker run -d nginx` | Executa container em segundo plano |
| `docker start <container>` | Inicia container existente |
| `docker stop <container>` | Para container rodando |
| `docker rm <container>` | Remove container parado |
| `docker rm -f <container>` | Força remoção do container |
| `docker exec -it <container> bash` | Entra em container rodando com Bash |
| `docker exec -it <container> sh` | Entra em container rodando com Shell simples |

## Imagens

| Comando | Uso |
| --- | --- |
| `docker pull ubuntu` | Baixa uma imagem |
| `docker build -t minha-api .` | Cria uma imagem a partir do Dockerfile |
| `docker build -t minha-api:1.0 .` | Cria uma imagem com tag de versão |
| `docker history <image>` | Mostra as camadas de uma imagem |
| `docker rmi ubuntu` | Remove uma imagem |
| `docker push usuario/imagem:tag` | Envia imagem para um registry |

## Portas e variáveis

| Comando | Uso |
| --- | --- |
| `docker run -p 8080:80 nginx` | Mapeia porta do host para porta do container |
| `docker run -e POSTGRES_PASSWORD=123456 postgres` | Define variável de ambiente |

## Volumes e mounts

| Comando | Uso |
| --- | --- |
| `docker volume ls` | Lista volumes gerenciados pelo Docker |
| `docker volume create novo-volume` | Cria um volume nomeado |
| `docker run -v dados:/var/lib/postgresql/data postgres` | Usa volume nomeado |
| `docker run --mount type=volume,source=novo-volume,target=/app ubuntu bash` | Usa volume com sintaxe explícita |
| `docker run -v $(pwd):/app node:20` | Usa bind mount com a pasta atual |
| `docker run --mount type=bind,source=/caminho/no/host,target=/app ubuntu bash` | Usa bind mount com sintaxe explícita |
| `docker run --mount type=tmpfs,target=/app ubuntu bash` | Usa tmpfs mount em memória |

## Limpeza

| Comando | Uso |
| --- | --- |
| `docker container prune` | Remove containers parados |
| `docker image prune` | Remove imagens não usadas |
| `docker system prune` | Remove vários recursos não usados |

> Antes de confirmar comandos de limpeza, leia o que o Docker vai remover.

## Docker Compose

| Comando | Uso |
| --- | --- |
| `docker compose up` | Sobe os serviços definidos no Compose |
| `docker compose up -d` | Sobe os serviços em segundo plano |
| `docker compose down` | Para e remove os serviços criados |
| `docker compose logs` | Mostra logs dos serviços |

## Diferenças que mais confundem

| Tema | Lembrete |
| --- | --- |
| `docker ps` | Mostra apenas containers rodando |
| `docker ps -a` | Mostra rodando e parados |
| `docker run` | Cria container novo |
| `docker start` | Inicia container que já existe |
| `docker exec` | Executa comando dentro de container rodando |
| `docker stop` | Para container, não imagem |
| `docker rm` | Remove container |
| `docker rmi` | Remove imagem |

## Sequência de treino

```bash
docker run hello-world
docker ps -a
docker pull ubuntu
docker run -it --name meu-ubuntu ubuntu bash
```

Dentro do container:

```bash
ls
pwd
cat /etc/os-release
exit
```

Depois:

```bash
docker ps -a
docker start meu-ubuntu
docker exec -it meu-ubuntu bash
exit
docker stop meu-ubuntu
docker rm meu-ubuntu
```

## Frase curta

```text
run cria.
start reinicia.
exec entra ou executa.
stop para.
rm remove container.
rmi remove imagem.
```
