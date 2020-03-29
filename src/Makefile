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



include sources.mk

# Platform Overrides    (We override this value anyway when executing ie:
#                       $make PLATFORM=MSP432, right?)
#                       (How do I read that overriden PLATFORM?)
#                       --> This is override from console doing ie:
#                           PlATFORM=MSP432
PLATFORM = HOST         #By deffault as HOST platform.

#General Flags (Both Platforms)

#   -Wall           Enable All Warning Messages (CFLAGS)
#   -Werror         Treats All Warnings as Errors(CFLAGS)
#   -g              Generate Debugging Info in Executable (CFLAGS)
#   -O0             The level of optimization (-O0, -O1, -O2, -O3)) (LDFLAGS)
#   -std=c99        The C standard set (CFLAGS)



#Compile time switches
ifeq($(PLATFORM),MSP432)
    #The list of included directories
    INCLUDES =               \
        -Iinclude/common     \      
        -Iinclude/msp432     \
        -Iinclude/CMSIS      \

    #The list of sources files that will need to be compiled (Platform specific)
    SOURCES =                           \
        /src./main.c                        \
        /src./memory.c                      \
        /src./interrupts_msp432p401r_gcc.c  \
        /src./startup_msp432p401r_gcc.c     \
        /src./system_msp432p401r.c
    # Architectures Specific Flags
    LINKER_FILE = msp432p401r.lds
    CPU = cortex-m4
    ARCH = armv7e-m
    SPECS = nosys.specs

    # Compiler Flags and Defines
    CC = arm-none-eabi-gcc
    LD = arm-none-eabi-ld
    LDFLAGS = -Wl, -O0, -T=$(LINKER_FILE), -Map=$(BASENAME).map
    CFLAGS = -Wall -Werror -g -O0 -std=c99 -mcpu=$(CPU) -march=$(ARCH) --specs=$(SPECS)
else ifeq($(PLATFORM),HOST)
    #The list of included directories
    INCLUDES =                  \
        -Iinclude/common
    #The list of sources files that will need to be compiled (Platform specific)
    SOURCES =           \
        /src./main.c    \
        /src./memory.c
    # Compiler Flags and Defines
    CC = gcc
    LD = ld
    LDFLAGS = -Wl, -O0 -Map=$(BASENAME).map
    CFLAGS = -Wall -Werror -g -O0 -std=c99
endif

#Preprocesor Flags
CPPFLAGS = -E
#Target name
BASENAME = c1m2
TARGET = $(BASENAME).out

#List of object files that must be produced
OBJS = (SOURCES:.c/.o)

#We need to generate a handful of files for each complete build. These are:
#   c1m2.map - Map file for the full build ( Use the -Map=<FILE> option )
#   *.d - Dependency Files for each source file (main.dep, memory.dep, etc)
#         (Use the -M* options [MP, MF, MD, MM, etc])
#   *.o - Individual object files (main.o, memory.o, etc)
#   c1m2.out - Output Executable file

#Target/Build Rules

#1. Generates the preprocessed output of all c-program implementation files
%.i: %.c
    $(CC) $(CPPFLAGS) $(CFLAGS) $@ $<
#2. Generates the object file of all c-program implementation files
%.O: %.c
    %(CC) -c $(CFLAGS) $@ $< 

#3. Builds target
$(TARGET): $(OBJS)
    $(CC) $(CFLAGS) $(INCLUDES) $(LDFLAGS) -O $(TARGET) $(OBJS)
    
#The previous Target Rule can be re-write as
#$(TARGET): $(OBJS)
#    $(CC) $(CFLAGS) $(INCLUDES) $(LDFLAGS) -O $@ $^

#3. PHONY RULE
.PHONY: build
build: all
all: $(TARGET)

