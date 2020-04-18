#******************************************************************************
# Copyright (C) 2017 by Alex Fosdick - University of Colorado
#
# Redistribution, modification or use of this software in source or binary
# forms is permitted as long as the files maintain this copyright. Users are 
# permitted to modify this and use it to learn about the field of embedded
# software. Alex Fosdick and the University of Colorado are not liable for any
# misuse of this material. 
#
#******************************************************************************

#------------------------------------------------------------------------------
# Simple Makefile for multitarget build system
#
# Use: make [TARGET] [PLATFORM-OVERRIDES]
#
# Build Targets:
#
#       build 						- Builds and links all source files and genereates:
#
#														c1m2.map - Map file for the full build
#														*.d      - Dependency Files for each source file
#														*.o      - Individual object files
#														c1m2.out - Output Executable file
#
#				<FILE>.i					- Builds <FILE>.i preprocessed file.
#				<FILE>.asm				- Builds <FILE>.i assembly file.
#				<FILE>.o 					- Builds <FILE>.o object file.
#       compile-all 			- Compile all objects but do NOT link them.
#       clean 						- Removes all generated files.
#
# Platform Overrides: Conditionally assign the appropriate compiler flags,
#       							linker flags, and architecture flags. The target platform
#      								must be provided at the command line with the make
#       							command to set the platform you are compiling for.
#       
#       							PLATFORM = MSP432 - The target embedded system will use
#                     										the cross compiler, arm-none-eabi-gcc.
#       							PLATFORM = HOST 	- The host embedded system will use the
#                         								native compiler, gcc.
#
#------------------------------------------------------------------------------

#------------------------------------------------------------------------------
#General Flags (Both Platforms)
#
#   -Wall           Enable All Warning Messages (CFLAGS)
#   -Werror         Treats All Warnings as Errors(CFLAGS)
#   -g              Generate Debugging Info in Executable (CFLAGS)
#   -O0             The level of optimization (-O0, -O1, -O2, -O3)) (LDFLAGS)
#   -std=c99        The C standard set (CFLAGS)
#
#------------------------------------------------------------------------------

#Target name
BASENAME := c1m2
TARGET := $(BASENAME).out

#General Flags
COMMONCFLAGS := -Wall -Werror -g -std=c99
#COMMONCFLAGS := -Wall -Wextra -Wconversion -pedantic -Werror -g -std=c99
COMMONLDFLAGS := -Wl,-O0,-Map=$(BASENAME).map
#No spaces after commas after -Wl option.

CPPFLAGS := -E
# -E flag makes the compiler stop in the preprocessed output

#include msp432p401r.lds

# One of the searchdir in the list of paths that ld will search for archive
# libraries and ld control scripts, using the L flag.
#SEARCHDIR := /home/adrian/Desktop/Coursera/IESSDE/iessde_fornaso_coursera_w2/C1M2-FORNASO/
SEARCHDIR := ../

#Compile time switches
ifeq ($(PLATFORM), MSP432)
	INCLUDES :=	-I../includes/common	\
							-I../includes/msp432	\
							-I../includes/CMSIS
	SOURCES :=  main.c												\
							memory.c											\
							interrupts_msp432p401r_gcc.c	\
							startup_msp432p401r_gcc.c			\
							system_msp432p401r.c

	LINKER_FILE := msp432p401r.lds
	CPU := cortex-m4
	ARCH := armv7e-m
	SPECS := nosys.specs

	CC := arm-none-eabi-gcc
	LD := arm-none-eabi-ld

	LDFLAGS := $(COMMONLDFLAGS),-L$(SEARCHDIR),-T$(LINKER_FILE)

	CFLAGS :=	$(COMMONCFLAGS) -D$(PLATFORM) -mcpu=$(CPU)	\
						-march=$(ARCH) --specs=$(SPECS)
	OBJDUMP := arm-none-eabi-objdump
endif
ifeq ($(PLATFORM), HOST)
	INCLUDES = -I../includes/common

	SOURCES =		main.c	\
							memory.c

	CC = gcc
	LD = ld
	LDFLAGS = $(COMMONLDFLAGS)
	CFLAGS = $(COMMONCFLAGS) -D$(PLATFORM)
	OBJDUMP = objdump
endif

#Listing object files:
OBJECTS = $(SOURCES:%.c=%.o)

# Test target
.PHONY: test1
test1:
	@echo "Searchdir:"
	@echo$(SEARCHDIR)

# 1. --------------------------------------------------------------------------
# Complete build:					c1m2.map - Map file for the full build
#													*.d      - Dependency Files for each source file
#													*.o      - Individual object files
#													c1m2.out - Output Executable file

# LDFLAGS contains the flags for creating the *.map file

.PHONY: build
build: $(TARGET) $(OBJECTS)
$(TARGET):
	$(CC) $(CFLAGS) $(LDFLAGS) $(SOURCES) $(INCLUDES) -o $@


# 2. --------------------------------------------------------------------------
# //// Generates the object files of all c-program implementation files and its
#      dependecies. ///////////////////////////////////////////////////////////

#This implementation places dependency files into a subdirectory named .deps.
DEPDIR := .deps
# DEPFLAGS:
# -M
# Instead of outputting the result of preprocessing, output a rule suitable for
# make describing the dependencies of the main source file.
# -MT $@
# Set the name of the target in the generated dependency file.

# -MMD
# Generate dependency information as a side-effect of compilation, not instead
# of compilation. This version omits system headers from the #generated
# dependencies: if you prefer to preserve system headers as prerequisites,
# use -MD.

# -MP
# Adds a target for each prerequisite in the list, to avoid errors when deleting
# files.

# -MF $(DEPDIR)/$*.d
# Write the generated dependency file $(DEPDIR)/$*.d.
DEPFLAGS := -M -MT $@ -MD -MP -MF $(DEPDIR)/$*.d

# Delete the built-in rules for building object files from .c files, so that
# our rule is used instead.
#%.o: %.c
# Our rule for building object files with its dependency
%.o: %.c $(DEPDIR)/%.d | $(DEPDIR)
	$(CC) $(DEPFLAGS) -c $(CFLAGS) $(INCLUDES) -o $@ $<

# Declare a rule for creating the dependency directory if it doesn’t exist.
$(DEPDIR): ; @mkdir -p $@

# Generate a list of all the dependency files that could exist.
DEPFILES := $(SOURCES:%.c=$(DEPDIR)/%.d)

# Mention each dependency file as a target, so that make won’t fail if the file
# doesn’t exist.
$(DEPFILES):


# 2 bis. ----------------------------------------------------------------------
# /// Generates the object file of all c-program implementation files. ////////
#%.o: %.c
#	$(CC) -c $(CFLAGS) $(INCLUDES) -o $@ $^


# 3. --------------------------------------------------------------------------
# /// Compile all objects but do NOT link them. ///////////////////////////////
.PHONY: compile-all
compile-all: $(SOURCES)
	$(CC) -c $(CFLAGS) $(INCLUDES) $^


# 4. --------------------------------------------------------------------------
# /// Generates the preprocessed output of all c-program implementation files. 
%.i: %.c
	$(CC) $(CPPFLAGS) $(CFLAGS) $(INCLUDES) -o $@ $^	


# 5. --------------------------------------------------------------------------
# /// Create assembler file of a C source. ////////////////////////////////////
%.asm: %.c
	$(CC) -S $(CFLAGS) $(INCLUDES) $< -o $@
# -S flag tells the compiler just generate the assembly file


# 6. --------------------------------------------------------------------------
# /// Removes all compiled objects, preprocessed outputs, assembly outputs,
#     executable files and build output files. ////////////////////////////////
.PHONY: clean
clean:
	rm -f $(TARGET) $(BASENAME).map *.o *.asm *.i
	rm -r $(DEPDIR)

#End of file
