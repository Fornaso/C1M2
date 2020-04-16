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

#
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
BASENAME = c1m2
TARGET = $(BASENAME).out

#General Flags
COMMONCFLAGS = -Wall -Werror -g -std=c99
COMMONLDFLAGS = -Wl,-O0,-Map=$(BASENAME).map
#No spaces after commas after -Wl option.

CPPFLAGS = -E
# -E flag makes the compiler stop in the preprocessed output

#Compile time switches
ifeq ($(PLATFORM), MSP432)
	INCLUDES =	-I../includes/common	\
							-I../includes/msp432	\
							-I../includes/CMSIS
	SOURCES =   main.c												\
							memory.c											\
							interrupts_msp432p401r_gcc.c	\
							startup_msp432p401r_gcc.c			\
							system_msp432p401r.c

	LINKER_FILE = msp432p401r.lds
	CPU = cortex-m4
	ARCH = armv7e-m
	SPECS = nosys.specs

	CC = arm-none-eabi-gcc
	LD = arm-none-eabi-ld

	LDFLAGS = $(COMMONLDFLAGS), -T=$(LINKER_FILE)
	#Problem: Map file generation is not working.
	#Solution: between the comma "," and -Map=<map_name>.map flag, must be
	#no spaces. ---> Now does work.

	CFLAGS =	$(COMMONCFLAGS) -D$(PLATFORM) -mcpu=$(CPU)	\
						-march=$(ARCH) --specs=$(SPECS)
	OBJDUMP = arm-none-eabi-objdump
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

#Listing files:
OBJECTS = $(SOURCES:.c=.o)

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

DEPFLAGS = -MT $@ -MD -MP -MF $(DEPDIR)/$*.d

# Delete the built-in rules for building object files from .c files, so that
# our rule is used instead.
# Doesn't work, it is NOT disabling the built in rule!!!
%.o: %.c
# Our rule for building object files with its dependency
%.o: %.c $(DEPDIR)/%.d | $(DEPDIR)
	$(CC) $(DEPFLAGS) -c $(CFLAGS) $(INCLUDES) -o $@ $<

# Declare a rule for creating the dependency directory if it doesn’t exist.
$(DEPDIR): ; @mkdir -p $@

# Generate a list of all the dependency files that could exist.
DEPFILES := $(SRCS:%.c=$(DEPDIR)/%.d)

# Mention each dependency file as a target, so that make won’t fail if the file
# doesn’t exist.
$(DEPFILES):

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
# /// Generate assembly output of c-program implementation files and the final
#     output executable. //////////////////////////////////////////////////////
#%.asm: %.s $(TARGET)
#	$(CC) -S $(CFLAGS) $< -O $@
#	$(OBJDUMP) -S --disassemble $(TARGET) > $(BASENAME).asm
#Problem: The previous rule does not work.
#Solution: Next rule for creating the asm files does work.

# 6. --------------------------------------------------------------------------
# /// Create assembler file of a C source. ////////////////////////////////////
%.asm: %.c
	$(CC) -S $(CFLAGS) $(INCLUDES) $< -o $@
# -S flag tells the compiler just generate the assembly file


# 7. --------------------------------------------------------------------------
# /// Generates the object file of all c-program implementation files. ////////
#%.o: %.c
#	$(CC) -c $(CFLAGS) $(INCLUDES) -o $@ $^




# 9. --------------------------------------------------------------------------
# /// Removes all compiled objects, preprocessed outputs, assembly outputs,
#     executable files and build output files. ////////////////////////////////
.PHONY: clean
clean:
	rm -f $(OBJECTS) $(TARGET) $(BASENAME).map *.asm *.i
	rm -r .dep


include $(wildcard $(DEPFILES))

#End of file
