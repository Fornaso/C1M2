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
include sources.mk

# Platform Overrides    (We override this value anyway when executing ie:
#                       $make PLATFORM=MSP432, right?)
#                       (How do I read that overriden PLATFORM?)
#                       --> This is override from console doing ie:
#                           PlATFORM=MSP432
PLATFORM = HOST         #By deffault as HOST platform.

#The list of included directories
INCLUDES =      \
    -I./CMSIS   \
    -I./common  \
    -I./msp432

#Compile time switches
ifeq($(PLATFORM),MSP432)
    #The list of sources files that will need to be compiled (Platform specific)
    SOURCES =                           \
        ./main.c                        \
        ./memory.c                      \
        ./interrupts_msp432p401r_gcc.c  \
        ./startup_msp432p401r_gcc.c     \
        ./system_msp432p401r.c
    # Architectures Specific Flags
    LINKER_FILE = msp432p401r.lds
    CPU = cortex-m4
    ARCH = armv7e-m
    SPECS = nosys.specs

    # Compiler Flags and Defines
    CC = arm-none-eabi-gcc
    LD = arm-none-eabi-ld
    LDFLAGS = -Wall -Werror -T msp432p401r.lds
    CFLAGS = 
    CPPFLAGs = 
else ifeq($(PLATFORM),HOST)
    #The list of sources files that will need to be compiled (Platform specific)
    SOURCES =       \
        ./main.c    \
        ./memory.c
    # Compiler Flags and Defines
    CC = gcc
    LD = ld
    LDFLAGS = 
    CFLAGS = 
    CPPFLAGS =
endif

#Target name
BASENAME = c1m2
TARGET =$(BASENAME).out

#List of object files that must be produced
OBJS := (SOURCES:.c/.o)

#We need to generate a handful of files for each complete build. These are:
#   c1m2.map - Map file for the full build ( Use the -Map=<FILE> option )
#   *.d - Dependency Files for each source file (main.dep, memory.dep, etc)
#         (Use the -M* options [MP, MF, MD, MM, etc])
#   *.o - Individual object files (main.o, memory.o, etc)
#   c1m2.out - Output Executable file

#Targets/Build Rules

#Generate the preprocessed output of all c-program implementation files
%.i: %.c
    $(CC) -E 




#1.
.PHONY: build
build: all
all: $(TARGET)


#2.

#3.
