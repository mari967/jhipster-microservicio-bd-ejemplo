# jhipster-microservicio-bd-imp

## Levantar el registry
1. Levantar el servicio de docker 
`dockerd`
2. Levantar el registry. Ir a la capreta del gateway
`docker compose -f src/main/docker/jhipster-registry.yml up`


## Levantar la base de datos postgres ocn docker
1. Descargar la imagen de postgres
`docker pull postgres`
2. Levantar el contenedor
`docker run --name postgresql -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=admin -p 5432:5432 -v /data:/var/lib/postgresql/data -d postgres`
