PROJECT=observability_platform
DB_MARIA_ROOT_PASS=obs_root
DB_MARIA_USER_LOGIN=obs_db_user
DB_MARIA_USER_PASS=obs_user_password
GRAFANA_VERSION=10.0.1
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=obs_grafana_pass

.PHONY : prepare prepare-dir-env prepare-images prepare-swarm prepare-plugins prepare-networks

prepare: prepare-dir-env prepare-images
prepare-first: prepare-swarm prepare-docker-plugins prepare-networks

prepare-dir-env:
	echo "export PROJECT=${PROJECT}" > ./wordpress-compose/.envrc
	echo "export DB_MARIA_ROOT_PASS=${DB_MARIA_ROOT_PASS}" >>  ./wordpress-compose/.envrc
	echo "export DB_MARIA_USER_LOGIN=${DB_MARIA_USER_LOGIN}" >>  ./wordpress-compose/.envrc
	echo "export DB_MARIA_USER_PASS=${DB_MARIA_USER_PASS}" >>  ./wordpress-compose/.envrc
	echo "export COMPOSE_PROJECT_NAME=${PROJECT}" >>  ./wordpress-compose/.envrc
	echo "export GRAFANA_VERSION=${GRAFANA_VERSION}" >>  ./wordpress-compose/.envrc
	echo "export GRAFANA_ADMIN_USER=${GRAFANA_ADMIN_USER}" >>  ./wordpress-compose/.envrc
	echo "export GRAFANA_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD}" >>  ./wordpress-compose/.envrc
	cd ./wordpress-compose/ && direnv allow

prepare-images:
	docker build -f ./wordpress-compose/grafana/Dockerfile \
        --label "project=${PROJECT}" \
        --build-arg GRAFANA_VERSION=${GRAFANA_VERSION} \
        -t "${PROJECT}-grafana:latest" ./wordpress-compose/grafana/
	docker build -f ./wordpress-compose/blackbox_exporter/Dockerfile \
            --label "project=${PROJECT}" \
            -t "${PROJECT}-blackbox_exporter:latest" ./wordpress-compose/blackbox_exporter/

prepare-swarm:
	#bash  -c '[[ "$(docker info --format '{{.Swarm.ControlAvailable}}')" == "true" ]] ||
	docker swarm init

prepare-docker-plugins:
	docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions

prepare-networks:
	docker network create -d overlay --subnet=10.201.0.0/16 --opt encrypted=false --attachable ${PROJECT}_traefik-public
	docker network create -d overlay --opt encrypted=false --attachable ${PROJECT}_efk
	docker network create -d overlay --opt encrypted=false --attachable ${PROJECT}_cms
	docker network create -d overlay --opt encrypted=false --attachable ${PROJECT}_metrics
	docker network create -d overlay --opt encrypted=false --attachable --internal ${PROJECT}_socat

run:
	# efk and logging
	PROJECT=${PROJECT} docker stack deploy -c wordpress-compose/docker-compose-efk.stage.yaml ${PROJECT}_efk
	# reverse proxy
	PROJECT=${PROJECT} docker stack deploy -c wordpress-compose/docker-compose-reverse-proxy.stage.yaml ${PROJECT}_reverse_proxy
	# cms
	#COMPOSE_PROJECT_NAME="${PROJECT}" PROJECT=${PROJECT} DB_MARIA_ROOT_PASS=${DB_MARIA_ROOT_PASS} \
#	DB_MARIA_USER_LOGIN=${DB_MARIA_USER_LOGIN} DB_MARIA_USER_PASS=${DB_MARIA_USER_PASS} \
#	docker stack deploy -c wordpress-compose/docker-compose-cms.stage.yaml ${PROJECT}_cms
	# monitoring
	PROJECT=${PROJECT}  DB_MARIA_ROOT_PASS=${DB_MARIA_ROOT_PASS} DB_MARIA_USER_LOGIN=${DB_MARIA_USER_LOGIN} DB_MARIA_USER_PASS=${DB_MARIA_USER_PASS} \
	GRAFANA_ADMIN_USER=${GRAFANA_ADMIN_USER} GRAFANA_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD} GRAFANA_VERSION=${GRAFANA_VERSION} \
	docker stack deploy -c wordpress-compose/docker-compose-monitoring.stage.yaml ${PROJECT}_monitoring


#	COMPOSE_PROJECT_NAME="${PROJECT}" PROJECT=${PROJECT} DB_MARIA_ROOT_PASS=${DB_MARIA_ROOT_PASS} \
#	DB_MARIA_USER_LOGIN=${DB_MARIA_USER_LOGIN} DB_MARIA_USER_PASS=${DB_MARIA_USER_PASS} \
#	GRAFANA_ADMIN_USER=${GRAFANA_ADMIN_USER} GRAFANA_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD} \
#	GRAFANA_VERSION=${GRAFANA_VERSION} \
#	docker compose -f ./wordpress-compose/docker-compose.yml up -d

stop:
	docker stack rm ${PROJECT}_reverse_proxy ${PROJECT}_efk ${PROJECT}_monitoring

# Absolutely awesome: http://marmelab.com/blog/2016/02/29/auto-documented-makefile.html
help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'


