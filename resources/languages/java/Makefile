# Copyright 1996-2019 Cyberbotics Ltd.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

ifeq ($(WEBOTS_HOME),)
WEBOTS_HOME_PATH = ../../..
else
space :=
space +=
WEBOTS_HOME_PATH=$(subst $(space),\ ,$(strip $(subst \,/,$(WEBOTS_HOME))))
endif

include $(WEBOTS_HOME_PATH)/resources/Makefile.include

#directories
SWIG            = swig
SWIG_OUTPUT_PATH= SWIG_generated_files
CONTROLLERPATH  = com/cyberbotics/webots/controller

#files
INTERFACE       = controller.i
WRAPPER         = $(addprefix $(SWIG_OUTPUT_PATH)/,$(INTERFACE:.i=_wrap.cpp))
WRAPPER_OBJECT  = $(WRAPPER:.cpp=.o)
JAR_FILE        = $(WEBOTS_HOME_PATH)/lib/java/Controller.jar
JAVA_FILES      = Accelerometer.java \
                  Brake.java \
                  Camera.java \
                  Compass.java \
                  Connector.java \
                  Device.java \
                  DifferentialWheels.java \
                  Display.java \
                  DistanceSensor.java \
                  Emitter.java \
                  Field.java \
                  GPS.java \
                  Gyro.java \
                  InertialUnit.java \
                  ImageRef.java \
                  Joystick.java \
                  Keyboard.java \
                  LED.java \
                  Lidar.java \
                  LidarPoint.java \
                  LightSensor.java \
                  Motion.java \
                  Motor.java \
                  Node.java \
                  Pen.java \
                  PositionSensor.java \
                  Propeller.java \
                  Radar.java \
                  RadarTarget.java \
                  RangeFinder.java \
                  Receiver.java \
                  Robot.java \
                  Skin.java \
                  Speaker.java \
                  Supervisor.java \
                  TouchSensor.java \
                  wrapper.java \
                  wrapperJNI.java
JAVA_SOURCES    = $(addprefix $(SWIG_OUTPUT_PATH)/,$(JAVA_FILES))
LIBNAME         = JavaController

.PHONY: JAVA_CLASSES

#compilation options
WEBOTS_INCLUDES = -I$(WEBOTS_HOME_PATH)/include/controller/cpp -I$(WEBOTS_HOME_PATH)/include/controller/c

SWIG_EXISTS = $(shell which $(SWIG) 2> /dev/null)

ifeq ($(CC),cc)
CC=gcc
endif

# variables depending on the OS
ifeq ($(OSTYPE),windows)
CFLAGS1         = -I$(WEBOTS_HOME_PATH)/windows/include
PLATFORM_INCLUDE= -I"$(JAVA_HOME)/include/win32"
PLATFORM_LIBS   =
TARGET          = $(WEBOTS_HOME_PATH)/lib/java/$(LIBNAME).dll
CFLAGS2         = -c -Wall -Wno-unused-function -O
JNILIB_FLAGS    = -bundle
JAVA_INCLUDES   = -I"$(JAVA_HOME)/include" -I"$(JAVA_HOME)/include/win32"
LIBCONTROLLER   = $(WEBOTS_HOME_PATH)/msys64/mingw64/bin/Controller.dll
LIBCPPCONTROLLER= $(WEBOTS_HOME_PATH)/msys64/mingw64/bin/CppController.dll
ECHO_N          = -n
else

ifeq ($(OSTYPE),darwin)
JAVA_HEADERS    = $(MACOSX_SDK_PATH)/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers
CFLAGS1         = -isysroot $(MACOSX_SDK_PATH) -mmacosx-version-min=$(MACOSX_MIN_SDK_VERSION)
PLATFORM_INCLUDE= -I"$(JAVA_HEADERS)"
TARGET          = $(WEBOTS_HOME_PATH)/lib/java/lib$(LIBNAME).jnilib
CFLAGS2         = -c -Wall -isysroot $(MACOSX_SDK_PATH) -mmacosx-version-min=$(MACOSX_MIN_SDK_VERSION) -Wno-unused-function
JNILIB_FLAGS    = -dynamiclib -install_name @rpath/lib/java/libJavaController.jnilib -Wl,-rpath,@loader_path/../..
JAVA_INCLUDES   = -I$(JAVA_HEADERS)
LIBCONTROLLER   = $(WEBOTS_HOME_PATH)/lib/libController.dylib
LIBCPPCONTROLLER= $(WEBOTS_HOME_PATH)/lib/libCppController.dylib
ECHO_N          = '\c'ifdef USE_STD_CXX

else #Linux
CFLAGS1         = -fPIC
PLATFORM_INCLUDE= -I"$(JAVA_HOME)/include/linux" -I"/usr/X11R6/include"
PLATFORM_LIBS   =
TARGET          = $(WEBOTS_HOME_PATH)/lib/java/lib$(LIBNAME).so
CFLAGS2         = -c -Wall -Wno-unused-function -fPIC
JNILIB_FLAGS    = -bundle
JAVA_INCLUDES   = -I"$(JAVA_HOME)/include" -I"$(JAVA_HOME)/include/linux"
LIBCONTROLLER   = $(WEBOTS_HOME_PATH)/lib/libController.so
LIBCPPCONTROLLER= $(WEBOTS_HOME_PATH)/lib/libCppController.so
ECHO_N          = -n
endif
endif

# rules
ifeq ($(SWIG_EXISTS),)
release debug profile:
	@echo -e "# \033[0;33mSWIG not installed, skipping Java API\033[0m"
else ifeq ($(JAVA_HOME),)
release debug profile:
	@echo -e "# \033[0;33mJava not installed or 'JAVA_HOME' not set, skipping Java API\033[0m"
else
release debug profile: $(JAR_FILE) $(TARGET)

JAVA_CLASSES: $(JAVA_SOURCES)
	javac -target 1.8 $(SWIG_OUTPUT_PATH)/*.java -d .

$(WRAPPER_OBJECT): $(WRAPPER)
	$(CXX) $(CFLAGS2) $(WEBOTS_INCLUDES) $(JAVA_INCLUDES) $< -o $@

$(WEBOTS_HOME_PATH)/lib/java/lib$(LIBNAME).so: $(WRAPPER_OBJECT)
	@mkdir -p $(WEBOTS_HOME_PATH)/lib/java
	$(CXX) -shared $^ -L$(WEBOTS_HOME_PATH)/lib -lController -lCppController -o $@
	@chmod a-x $@

$(WEBOTS_HOME_PATH)/lib/java/lib$(LIBNAME).jnilib: $(WRAPPER_OBJECT)
	@mkdir -p $(WEBOTS_HOME_PATH)/lib/java
	$(CXX) $(CFLAGS1) $(JNILIB_FLAGS) -L$(WEBOTS_HOME_PATH)/lib -lController -lCppController $^ -o "$@"

$(WEBOTS_HOME_PATH)/lib/java/$(LIBNAME).dll: $(WRAPPER_OBJECT)
	@mkdir -p $(WEBOTS_HOME_PATH)/lib/java
	$(CXX) -shared -mwindows -Wl,--add-stdcall-alias -Wl,--enable-auto-import -O \
	$^ -L$(WEBOTS_HOME_PATH)/msys64/mingw64/bin -lController -lCppController -lm -o "$@"

$(JAR_FILE): JAVA_CLASSES
	@mkdir -p $(WEBOTS_HOME_PATH)/lib/java
	jar cfv $(JAR_FILE) $(CONTROLLERPATH)/*.class > /dev/null

$(JAVA_SOURCES): $(WRAPPER)

$(WRAPPER): $(INTERFACE) $(LIBCONTROLLER) $(LIBCPPCONTROLLER)
	test -d $(SWIG_OUTPUT_PATH) || mkdir -p $(SWIG_OUTPUT_PATH)
	$(SWIG) -c++ -java -package $(subst /,.,$(CONTROLLERPATH)) -outdir $(SWIG_OUTPUT_PATH) $(WEBOTS_INCLUDES) -o "$@" $<

$(LIBCONTROLLER):
	@echo "$(LIBCONTROLLER) doesn't exist, build it first."

$(LIBCPPCONTROLLER):
	@echo "$(LIBCPPCONTROLLER) doesn't exist, build it first."
endif

clean:
ifneq ($(SWIG_EXISTS),) # otherwise a user may break the distribution
FILES_TO_REMOVE = $(SWIG_OUTPUT_PATH)
endif
FILES_TO_REMOVE += $(TARGET) $(JAR_FILE) *.o com
EMPTY_DIRECTORIES_TO_REMOVE = $(WEBOTS_HOME_PATH)/lib/java
