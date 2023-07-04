PROJECT=observability_hw1
DB_MARIA_ROOT_PASS=hm1root
DB_MARIA_USER_LOGIN=hm1_db_user
DB_MARIA_USER_PASS=hm1_user_password
GRAFANA_VERSION=10.0.1
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=hm1_grafana_pass

setup:
	docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions

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

run:
	COMPOSE_PROJECT_NAME="${PROJECT}" PROJECT=${PROJECT} DB_MARIA_ROOT_PASS=${DB_MARIA_ROOT_PASS} \
	DB_MARIA_USER_LOGIN=${DB_MARIA_USER_LOGIN} DB_MARIA_USER_PASS=${DB_MARIA_USER_PASS} \
	GRAFANA_ADMIN_USER=${GRAFANA_ADMIN_USER} GRAFANA_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD} \
	GRAFANA_VERSION=${GRAFANA_VERSION} \
	docker compose -f ./wordpress-compose/docker-compose.yml up -d


# Absolutely awesome: http://marmelab.com/blog/2016/02/29/auto-documented-makefile.html
help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'


