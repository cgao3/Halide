include ../support/Makefile.inc

AUTOSCHED_SRC = $(realpath ../autoscheduler)
AUTOSCHED_BIN = $(realpath ../autoscheduler/bin)

.PHONY: build clean test

build: $(BIN)/$(HL_TARGET)/process

$(GENERATOR_BIN)/random_pipeline.generator: random_pipeline_generator.cpp $(GENERATOR_DEPS)
	@mkdir -p $(@D)
	$(CXX) $(CXXFLAGS) $(filter-out %.h,$^) -o $@ $(LIBHALIDE_LDFLAGS)

$(BIN)/%/random_pipeline.a: $(GENERATOR_BIN)/random_pipeline.generator
	@mkdir -p $(@D)
	$^ -g random_pipeline -e $(GENERATOR_OUTPUTS) -o $(@D) -f random_pipeline target=$* auto_schedule=false

$(BIN)/%/random_pipeline_auto_schedule.a: $(GENERATOR_BIN)/random_pipeline.generator
	@mkdir -p $(@D)
	$^ -g random_pipeline -e $(GENERATOR_OUTPUTS) -o $(@D) -f random_pipeline_auto_schedule target=$*-no_runtime auto_schedule=true $(HL_CUSTOM_AUTOSCHEDULER)

$(BIN)/%/process: process.cpp $(BIN)/%/random_pipeline.a $(BIN)/%/random_pipeline_auto_schedule.a
	@mkdir -p $(@D)
	$(CXX) $(CXXFLAGS) -I$(BIN)/$* -Wall $^ -o $@ $(LDFLAGS)

run: $(BIN)/$(HL_TARGET)/process
	@mkdir -p $(@D)
	$^

clean:
	rm -rf $(BIN)

test: run

autotune: # We assume here that our scripts have already built the autotuning binaries
	bash $(AUTOSCHED_SRC)/autotune_loop.sh \
		$(BIN)/$(HL_TARGET)/random_pipeline.generator \
		random_pipeline \
		"host" \
		$(HL_WEIGHTS_DIR) \
		$(AUTOSCHED_BIN) \
		$(HALIDE_DISTRIB_PATH) \
		samples/

