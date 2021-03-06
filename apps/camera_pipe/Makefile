include ../support/Makefile.inc

all: $(BIN)/process

TIMING_ITERATIONS ?= 5

$(BIN)/camera_pipe.generator: camera_pipe_generator.cpp $(GENERATOR_DEPS)
	@mkdir -p $(@D)
	$(CXX) $(CXXFLAGS) -g -fno-rtti $(filter-out %.h,$^) -o $@ $(LDFLAGS) $(HALIDE_SYSTEM_LDFLAGS)

$(BIN)/camera_pipe.a: $(BIN)/camera_pipe.generator
	@mkdir -p $(@D)
	$^ -g camera_pipe -o $(BIN) -f camera_pipe target=$(HL_TARGET) auto_schedule=false

$(BIN)/camera_pipe_auto_schedule.a: $(BIN)/camera_pipe.generator
	@mkdir -p $(@D)
	$^ -g camera_pipe -o $(BIN) -f camera_pipe_auto_schedule target=$(HL_TARGET)-no_runtime auto_schedule=true

$(BIN)/viz/camera_pipe.a: $(BIN)/camera_pipe.generator
	@mkdir -p $(@D)
	$^ -g camera_pipe -o $(BIN)/viz target=$(HL_TARGET)-trace_all

$(BIN)/process: process.cpp $(BIN)/camera_pipe.a $(BIN)/camera_pipe_auto_schedule.a
	$(CXX) $(CXXFLAGS) -Wall -O3 -I$(BIN) $^ -o $@ $(IMAGE_IO_FLAGS) $(LDFLAGS)

$(BIN)/viz/process: process.cpp $(BIN)/viz/camera_pipe.a
	$(CXX) $(CXXFLAGS) -DNO_AUTO_SCHEDULE -Wall -O3 -I$(BIN)/viz $^ -o $@ $(IMAGE_IO_FLAGS) $(LDFLAGS)

$(BIN)/out.png: $(BIN)/process
	$(BIN)/process $(IMAGES)/bayer_raw.png 3700 2.0 50 1.0 $(TIMING_ITERATIONS) $@ $(BIN)/h_auto.png

../../bin/HalideTraceViz: ../../util/HalideTraceViz.cpp
	$(MAKE) -C ../../ bin/HalideTraceViz

$(BIN)/camera_pipe.mp4: $(BIN)/viz/process viz.sh $(HALIDE_TRACE_VIZ) ../../bin/HalideTraceViz
	HL_AVCONV=$(HL_AVCONV) bash viz.sh $(BIN)

clean:
	rm -rf $(BIN)

test: $(BIN)/out.png

viz: $(BIN)/camera_pipe.mp4
	$(HL_VIDEOPLAYER) $^

$(BIN)/viz_auto.mp4: $(BIN)/viz/process ../support/viz_auto.sh ../../bin/HalideTraceViz
	@mkdir -p $(@D)
	HL_AVCONV=$(HL_AVCONV) bash ../support/viz_auto.sh "$(BIN)/viz/process $(IMAGES)/bayer_small.png 3700 1.8 50 1 1 $(BIN)/out.png" ../../bin/HalideTraceViz $@

viz_auto: $(BIN)/viz_auto.mp4
	$(HL_VIDEOPLAYER) $^
