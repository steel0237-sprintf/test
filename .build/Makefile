ifndef PROJECT
	PROJECT=putevkanew
endif

ifndef ENV
	ENV=dev
endif

ifndef APP_ENV
	APP_ENV=dev
endif

ifndef BUILDUID
	BUILDUID=`id -u`
endif

ifndef BUILDGID
	BUILDGID=`id -g`
endif

.PHONY: tests

COMPOSE=PROJECT=$(PROJECT) ENV=$(ENV) BUILDUID=$(BUILDUID) BUILDGID=$(BUILDGID) docker compose -f docker-compose.yaml -f docker-compose.$(ENV).yaml
STACK_DEPLOY=PROJECT=$(PROJECT) ENV=$(ENV) BUILDUID=$(BUILDUID) BUILDGID=$(BUILDGID) docker stack deploy --compose-file docker-compose.yaml --compose-file docker-compose.$(ENV).yaml $(PROJECT)-$(ENV) --prune --with-registry-auth

env:
	@echo "Setting up env vars for $(ENV) ..."
	@echo "PROJECT is $(PROJECT)"
	@echo "ENV is $(ENV)"
	@echo "APP_ENV is $(APP_ENV)"
	@echo "BUILDUID is $(BUILDUID)"
	@echo "BUILDGID is $(BUILDGID)"
	@echo "build.counter is $(CI_PIPELINE_ID)"
	@sed -i -r "s~^ENV=.+~ENV=$(ENV)~g" .env
	@sed -i -r "s~^APP_ENV=.+~APP_ENV=$(APP_ENV)~g" .env
	@sed -i -r "s~^BUILDUID=[0-9]+~BUILDUID=$(BUILDUID)~g" .env
	@sed -i -r "s~^BUILDGID=[0-9]+~BUILDGID=$(BUILDGID)~g" .env
	@echo "Set up of env for $(ENV) is completed!"

build: env
	@echo "Building $(ENV) ..."
	@$(COMPOSE) build  --build-arg PRIVATE="$$(cat ~/.ssh/id_rsa)" || exit 1
	@echo "Build $(ENV) !"

up: build
	@echo "Starting $(ENV) ..."
	@set -e
	@$(COMPOSE) up -d --remove-orphans || exit 1
	@$(COMPOSE) exec -T php php -i
	@echo "Started $(ENV) !"

db: env
	@echo "Create DB $(ENV) ..."
	@set -e
	@cat docker/postgresql/san_structure.sql | docker compose exec -T postgresql psql -h127.0.0.1 -Uputevka -d putevka
	@echo "Done DB $(ENV) !"

push:
	@echo "Pushing $(ENV) ..."
	@set -e
	@docker buildx bake -f docker-compose.yaml -f docker-compose.$(ENV).yaml --file docker-compose-cache.json --pull --push || exit 1
	@echo "Pushed $(ENV) !"

deploy:
	@echo "Deploy $(ENV) ..."
	@set -e
	@$(STACK_DEPLOY) || exit 1
	@echo "Deploy $(ENV) !"

clean:
	@echo "Stopping all containers and cleaning $(ENV) ..."
	@docker stop $$(docker ps -a -q)
	@yes | docker system prune
	@echo "Cleaned $(ENV) !"

clean-all:
	@echo "Stopping all containers and cleaning $(ENV) ..."
	@docker stop $$(docker ps -a -q)
	@yes | docker system prune -a
	@yes | docker volume prune
	@echo "Cleaned $(ENV) !"
