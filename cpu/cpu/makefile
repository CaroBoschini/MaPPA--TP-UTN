include settings.mk

################################################################################

filename = $(1).out

define compile_bin
$(CC) $(CFLAGS) -o "$@" $^ $(IDIRS:%=-I%) $(LIBDIRS:%=-L%) $(RUNDIRS:%=-Wl,-rpath,%) $(LIBS:%=-l%)
endef

define compile_objs
$(CC) $(CFLAGS) -c -o "$@" $< $(IDIRS:%=-I%)
endef

################################################################################

# Check if tests folder exists
ifneq ($(wildcard tests/*),)
TESTS_ENABLED=1
endif

# Set compiler and archiver options
CC=gcc
AR=ar
ARFLAGS=rcs

# Set prerrequisites
SRCS_C += $(shell find src/ -iname "*.c")
SRCS_H += $(shell find src/ -iname "*.h")
DEPS = $(foreach SHL,$(SHARED_LIBPATHS),$(SHL:%=%/bin/lib$(notdir $(SHL)).so)) \
	$(foreach STL,$(STATIC_LIBPATHS),$(STL:%=%/bin/lib$(notdir $(STL)).a))

# Set test prerrequisites
ifeq ($(TESTS_ENABLED),1)
TESTS_C += $(shell find tests/ -iname "*.c")
TESTS_H += $(shell find tests/ -iname "*.h")
endif

# Set header paths to (-I)nclude
IDIRS += $(addsuffix /src,$(SHARED_LIBPATHS) $(STATIC_LIBPATHS) .)

# Set library paths to (-L)ook
LIBDIRS = $(addsuffix /bin,$(SHARED_LIBPATHS) $(STATIC_LIBPATHS))

# Set shared library paths to be found in runtime (-rpath)
RUNDIRS = $(SHARED_LIBPATHS:%=$(shell cd . && pwd)/%/bin)

# Set intermediate objects
OBJS = $(patsubst src/%.c,obj/%.o,$(SRCS_C))

# Set test intermediate objects
ifeq ($(TESTS_ENABLED),1)
TEST_OBJS = $(filter-out $(TEST_EXCLUDE), $(TESTS_C)) $(patsubst src/%.c,obj/%.o,$(filter-out $(TEST_EXCLUDE), $(SRCS_C)))
endif

# Set binary targets
BIN = bin/$(call filename,$(shell cd . && pwd | xargs basename))

# Set test binary targets
ifeq ($(TESTS_ENABLED),1)
TEST = bin/$(shell cd . && pwd | xargs basename)_tests.out
endif

.PHONY: all
all: debug $(TEST)

.PHONY: debug
debug: CFLAGS = $(CDEBUG)
debug: $(BIN)

.PHONY: release
release: CFLAGS = $(CRELEASE)
release: $(BIN)

.PHONY: clean
clean:
	-rm -rfv obj bin

.PHONY: watch
watch:
	@test $(shell which entr) || entr
	while sleep 0.1; do \
		find src/ | entr -d make all --no-print-directory; \
	done

$(BIN): $(OBJS) | $(dir $(BIN))
	$(call compile_bin)

obj/%.o: src/%.c $(SRCS_H) $(DEPS) | $(dir $(OBJS))
	@echo "\ncompilando cpu..."
	$(call compile_objs)

ifeq ($(TESTS_ENABLED),1)
$(TEST): CFLAGS = $(CDEBUG)
$(TEST): $(TEST_OBJS) | $(dir $(TEST))
	$(CC) $(CFLAGS) -o "$@" $^ $(IDIRS:%=-I%) $(LIBDIRS:%=-L%) $(RUNDIRS:%=-Wl,-rpath,%) $(LIBS:%=-l%) -lcspecs
endif

.SECONDEXPANSION:
$(DEPS): $$(shell find $$(patsubst %bin/,%src/,$$(dir $$@)) -iname "*.c" -or -iname "*.h")
	make --no-print-directory -C $(patsubst %bin/,%,$(dir $@)) 3>&1 1>&2 2>&3 | sed -E 's,(src/|tests/)[^ ]+\.(c|h)\:,$(patsubst %bin/,%,$(dir $@))&,' 3>&2 2>&1 1>&3

$(sort $(dir $(BIN) $(OBJS))):
	mkdir -pv $@


################################################################################

.PHONY: start
start: debug
	valgrind --tool=none ./$(BIN) $(ARGS)

.PHONY: daemon
daemon:
	@test $(shell which entr) || entr
	while sleep 0.1; do \
		find src/ | entr -d make start --no-print-directory; \
	done

.PHONY: memcheck
memcheck: debug
	valgrind --leak-check=full $(MEMCHECK_FLAGS) ./$(BIN) $(ARGS_ESTRES)

.PHONY: helgrind
helgrind: debug
	valgrind --tool=helgrind $(HELGRIND_FLAGS) ./$(BIN) $(ARGS)

.PHONY: exec
start:	$(BIN)
	./$(BIN) $(ARGS)

################################################################################

.PHONY: prueba_base
prueba_base: $(BIN)
	./$(BIN) $(ARGS_BASE)

.PHONY: prueba_deadlock
prueba_deadlock: $(BIN)
	./$(BIN) $(ARGS_DEADLOCK)

.PHONY: prueba_errores
prueba_errores: $(BIN)
	./$(BIN) $(ARGS_ERRORES)

.PHONY: prueba_estres
prueba_estres: $(BIN)
	./$(BIN) $(ARGS_ESTRES)

.PHONY: prueba_fs
prueba_fs: $(BIN)
	./$(BIN) $(ARGS_FS)

.PHONY: prueba_integral
prueba_integral: $(BIN)
	./$(BIN) $(ARGS_INTEGRAL)

.PHONY: prueba_memoria
prueba_memoria: $(BIN)
	./$(BIN) $(ARGS_MEMORIA)

.PHONY: prueba_recursos
prueba_recursos: $(BIN)
	./$(BIN) $(ARGS_RECURSOS)

################################################################################

ifeq ($(TESTS_ENABLED),1)
.PHONY: test
test: $(TEST)
	valgrind --tool=none ./$(TEST)

.PHONY: test-daemon
test-daemon:
	@test $(shell which entr) || entr
	while sleep 0.1; do \
		find tests/ src/ | entr -d make test --no-print-directory; \
	done

.PHONY: test-memcheck
test-memcheck: $(TEST)
	valgrind --leak-check=full $(MEMCHECK_FLAGS) ./$(TEST)

.PHONY: test-helgrind
test-helgrind: $(TEST)
	valgrind --tool=helgrind $(HELGRIND_FLAGS) ./$(TEST)
endif

