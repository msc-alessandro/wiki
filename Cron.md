## Cron

O Cron é um `daemon` que executa scripts periodicamente em uma máquina Linux. Por padrão, o arquivo de Cron fica em `/var/spool/cron/crontabs` e são comandos do `shell` que são executados em um tempo definido pelo usuário.


## Gitlab-CI Cron

O Gitlab-CI por padrão não faz a limpeza das imagens do `docker` que são utilizadas durante os processos de *build* nos repositórios. Sendo assim, para evitar de lotar o HD e a RAM da máquina em que as builds são executas, um *Cronjob* faz a limpeza de todos os *containers* e *imagens* que estão suspensas pelo `docker`. Uma cópia destes scripts estão no repositório [Cron](https://gitlab.com/electricdreams/cron) e eles são da seguinte forma:

```
00 02,12 * * * /usr/bin/docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -e MINIMUM_IMAGES_TO_SAVE=1 -e REMOVE_VOLUMES=1 -v /etc:/etc:ro node:10
00 02,12 * * * /usr/bin/docker system prune -af
00 02,12 * * * /usr/bin/docker system prune --volumes -f
00 02,12 * * * find /gitlab-runner/builds -type f -ctime +2 -delete && find /gitlab-runner/builds -type d -empty -delete
```
Estes scripts são executados **todos os dias as 12:00 PM e as 02:00 AM**.


