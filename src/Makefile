#******************************************************************************
# Copyright (C) 2017 by Alex Fosdick - University of Colorado
#
# Redistribution, modification or use of this software in source or binary
# forms is permitted as long as the files maintain this copyright. Users are 
# permitted to modify this and use it to learn about the field of embedded
# software. Alex Fosdick and the University of Colorado are not liable for any
# misuse of this material. 
#
#*****************************************************************************

#------------------------------------------------------------------------------
# Simple Makefile for multitarget build system
#
# Use: make [TARGET] [PLATFORM-OVERRIDES]
#
# Build Targets:
#       <FILE>.o - Builds <FILE>.o object file
#       build - Builds and links all source files
#       all - Same as 'buils'
#       clean - Removes all generated files
#
# Platform Overrides: Conditionally assign the appropriate compiler flags,
#       linker flags, and architecture flags. The target platform must be
#       provided at the command line with the make command to set the platform
#       you are compiling for.
#       
#       PLATFORM = MSP432 - The target embedded system will use the cross
#                           compiler, arm-none-eabi-gcc.
#       PLATFORM = HOST - The host embedded system will use the native
#                         compiler, gcc.
#
#------------------------------------------------------------------------------


#   You will need to generate a handful of files for each complete build.
#   These are:
#
#   c1m2.map - Map file for the full build ( Use the -Map=<FILE> option )
#   *.d - Dependency Files for each source file (main.dep, memory.dep, etc)
#   ( Use the -M* options [MP, MF, MD, MM, etc])
#   *.o - Individual object files (main.o, memory.o, etc)
#   1c1m2.out - Output Executable file
#include sources.mk
# Platform Overrides    (We override this value anyway when executing ie:
#                       $make PLATFORM=MSP432, right?)
#                       (How do I read that overriden PLATFORM?)
#                       --> This is override from console doing ie:
#                           PlATFORM=MSP432
#PLATFORM = HOST         #By deffault as HOST platform.
#
#General Flags (Both Platforms)
#
#   -Wall           Enable All Warning Messages (CFLAGS)
#   -Werror         Treats All Warnings as Errors(CFLAGS)
#   -g              Generate Debugging Info in Executable (CFLAGS)
#   -O0             The level of optimization (-O0, -O1, -O2, -O3)) (LDFLAGS)
#   -std=c99        The C standard set (CFLAGS)
#

#Compile time switches
ifeq ($(PLATFORM), MSP432)
	INCLUDES =  -I../includes/common	\
                -I../includes/msp432	\
		        -I../includes/CMSIS
	SOURCES =   main.c						\
		        memory.c						\
		        interrupts_msp432p401r_gcc.c	\
		        startup_msp432p401r_gcc.c     \
                system_msp432p401r.c

	LINKER_FILE = msp432p401r.lds
	CPU = cortex-m4
	ARCH = armv7e-m
	SPECS = nosys.specs

	CC = arm-none-eabi-gcc
	LD = arm-none-eabi-ld

	LDFLAGS = -Wl, -O0, -T=$(LINKER_FILE),-Map=$(BASENAME).map
	#Fixing the map file generation (not working)
	#Try: -Xlinker -Map=output.map 
	CFLAGS = -Wall -Werror -g -std=c99 -D$(PLATFORM) -mcpu=$(CPU) -march=$(ARCH) --specs=$(SPECS)
	OBJDUMP = arm-none-eabi-objdump
endif
ifeq ($(PLATFORM), HOST)
	INCLUDES = -I../includes/common

	SOURCES =   main.c	\
	            memory.c
#./main.c ./memory.c
	CC = gcc
	LD = ld
	LDFLAGS = -Wl,-O0,-Map=$(BASENAME).map
	#No spaces after commas after -Wl option produces less errors
	#---> Still fix gcc: error: c1m2.out: No such file or directory

	CFLAGS = -Wall -Werror -g -std=c99 -D$(PLATFORM)
	OBJDUMP = objdump
endif


CPPFLAGS = -E
BASENAME = c1m2
TARGET = $(BASENAME).out
OBJS = $(SOURCES:.c/.o)

#SOURCES = source_file_outside_if.c          #We write a variable content for testing it
#Rule for testing if SOURCES variable is actually working
.PHONY: test1
test1:
	@echo "Variable INCLUDES:"
	@echo "$(INCLUDES)"
	@echo "Variable SOURCES:"
	@echo "$(SOURCES)"
	@echo "PLATFORM:"
	@echo "$(PLATFORM)"

$(TARGET): $(SOURCES)
	$(CC) $(CFLAGS) $(INCLUDES) $(SOURCES) $(LDFLAGS) -o $@

#5. Compile all object files and link into a final executable.
.PHONY: build
build: $(TARGET)


#4. Compile all objects but do NOT link them
.PHONY: compile-all
compile-all: $(SOURCES)
	$(CC) -c $(CFLAGS) $(INCLUDES) $(SOURCES)


#1. Generates the preprocessed output of all c-program implementation files
%.i: %.c
	$(CC) $(CPPFLAGS) $(CFLAGS) $@ $<


#2. Generate assembly output of c-program implementation files and the final
#   output executable
%.asm: %.s $(TARGET)
	$(CC) -S $(CFLAGS) $< -O $@
	$(OBJDUMP) -S --disassemble $(TARGET) > $(BASENAME).asm


#3. Generates the object file of all c-program implementation files
%.O: %.c
	$(CC) -c $(CFLAGS) $@ $< 


#6. Removes all compiled objects, preprocessed outputs, assembly outputs,
#   executable files and build output files
OBJS = bject1.o
.PHONY: clean
clean:
	-sudo rm -f $(OBJS) $(TARGET) $(BASENAME).map *.asm *.i
	echo "$(OBJS)"
