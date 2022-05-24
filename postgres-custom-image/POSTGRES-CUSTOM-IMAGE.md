------------------------------------------------------------------------------------------------------------------------

![](./static/main.jpeg)

------------------------------------------------------------------------------------------------------------------------

<h6>CHAPTER UNNUMBERED: THEY'LL FIX YOU. THEY FIX EVERYTHING.</h6>

:mechanical_arm: Hi, folks,

The very first step for creating the Postgres docker image with pre init data is preparing it.
You could decrease the size of the database by removing unused data on it and running the `VACUUM FULL`.

After that create a dump using typical command:
`pg_dump --dbname=vts --file="/Users/USER_NAME/Desktop/{data_source}-{timestamp}-dump.sql" --column-inserts --create --if-exists `

Move dump into the same folder with `Dockerfile` and run the next command to create the image:
`docker image build . -t postgres-custom:latest`

<b>Dockerfile</b>

Sample of Dockerfile:
``` yaml
FROM postgres:14.3-alpine as dumper

COPY pgdump.sql /docker-entrypoint-initdb.d/
RUN ["sed", "-i", "s/exec \"$@\"/echo \"skipping...\"/", "/usr/local/bin/docker-entrypoint.sh"]

ENV POSTGRES_USER=postgres
ENV POSTGRES_PASSWORD=postgres
ENV PGDATA=/data

RUN ["/usr/local/bin/docker-entrypoint.sh", "postgres"]

# final build stage
FROM postgres:14.3-alpine
MAINTAINER Vadzim Kavalkou <vadzim.kavalkou@gmail.com>
COPY --from=dumper /data $PGDATA

```

To run the custom image use the next script:
`docker run -p 5432:5432 postgres-custom:latest`

That's it.

If you have any question, feel free to contact me direct in [linkedin](https://www.linkedin.com/in/vadzimkavalkou/).

------------------------------------------------------------------------------------------------------------------------

[BACK TO THE MAIN PAGE](../README.md)
