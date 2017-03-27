CONFIG ?= Debug
CMAKE_SOURCE_DIR = $(CURDIR)
OUT = out/$(CMAKE_SOURCE_DIR)

.PHONY: up
up:
	docker-compose up --build -d

.PHONY: down
down:
	docker-compose down

.PHONY: shell
shell:
	docker-compose exec --user `id -u`:`id -g` develop bash

.PHONY: init
init:
	mkdir -p $(OUT)

.PHONY: remove
remove:
	rm -rf $(OUT)

.PHONY: cmake
cmake: init
	cd $(OUT) && cmake -G Ninja -DCMAKE_BUILD_TYPE=$(CONFIG) $(CMAKE_SOURCE_DIR)

.PHONY: clean
clean:
	cmake --build $(OUT) --target clean

.PHONY: build
build: cmake
	cmake --build $(OUT)

.PHONY: test
test: build
	cmake --build $(OUT) --target test
