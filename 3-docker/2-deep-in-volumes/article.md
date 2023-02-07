# Docker Volume

Dados que serão compartilhados entre o filesystem da máquina com os containers com finalidade de solucionar o problema de ao reiniciar, matar ou migrar containers ocorra a parca de dados. Assim fica mais fácil e prático armazenar informações que irão serem persistentes entre a máquina de desenvolvimento e os containers do host.

## Por que usar?

- Isolar e reutilizar os dados entre os containers
- Um mesmo volume pode ser compartilhado e usado por vários containers ao mesmo tempo
- Evita perder dados sempre que o container sair do ar

## Comandos

```tsx
docker volume create      Create a volume
docker volume inspect     Display detailed information on one or more volumes
docker volume ls          List volumes
docker volume prune       Remove all unused local volumes
docker volume rm          Remove one or more volumes
```