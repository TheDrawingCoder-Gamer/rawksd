#---------------------------------------------------------------------------------
# Clear the implicit built in rules
#---------------------------------------------------------------------------------
.SUFFIXES:

#---------------------------------------------------------------------------------
# TARGET is the name of the output
# BUILD is the directory where object files & intermediate files will be placed
# SOURCES is a list of directories containing source code
# INCLUDES is a list of directories containing extra header files
#---------------------------------------------------------------------------------
TARGET		:=	$(notdir $(CURDIR))
BUILD		:=	build
SOURCES		:=	source source/fat
DATA		:=	data  
INCLUDES	:=  include include/fat ../../libios/include

#---------------------------------------------------------------------------------
# the prefix on the compiler executables
#---------------------------------------------------------------------------------
PREFIX		:= $(DEVKITARM)/bin/arm-none-eabi-
CC			:= $(PREFIX)gcc
CXX			:= $(PREFIX)g++
AR			:= $(PREFIX)ar
OBJCOPY		:= $(PREFIX)objcopy
LD			:= $(PREFIX)g++
AS			:= $(PREFIX)g++

#---------------------------------------------------------------------------------
# linker script
#---------------------------------------------------------------------------------
ifeq ($(BUILDING),$(emptystring))

export ROOT	:= $(CURDIR)

all:
	@[ -d $(BUILD) ] || mkdir -p $(BUILD)
	@$(MAKE) -C $(BUILD) --no-print-directory -f $(CURDIR)/Makefile BUILDING=all
clean:
	@echo clean ...
	@rm -fr $(BUILD) 
	@rm -f $(TARGET).a
else

TARGET := $(notdir $(ROOT))
#----------------------------------------------------
# MS Visual Studio Style Fix:
#----------------------------------------------------
STYLEFIX	= 2>&1 | sed -e 's/\([a-zA-Z\.]\+\):\([0-9]\+\):\([0-9]\+:\)\?\(.\+\)/\1(\2):\4/' -e 's/undefined/error: undefined/'

#---------------------------------------------------------------------------------
# automatically build a list of object files for our project
#---------------------------------------------------------------------------------
OUTPUT		:=  $(ROOT)/$(TARGET)
CFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(ROOT)/$(dir)/*.c)))
CPPFILES	:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(ROOT)/$(dir)/*.cpp)))
sFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(ROOT)/$(dir)/*.s)))
SFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(ROOT)/$(dir)/*.S)))
BINFILES	:=	$(foreach dir,$(DATA),$(notdir $(wildcard $(ROOT)/$(dir)/*.*)))

OFILES		:=	$(addsuffix _bin.o,$(BINFILES)) \
					$(CPPFILES:.cpp=_cpp.o) $(CFILES:.c=_c.o) \
					$(sFILES:.s=_s.o) $(SFILES:.S=_S.o)
					
DEPENDS		:= $(OFILES:.o=.d)

VPATH		=  $(foreach dir,$(SOURCES),$(ROOT)/$(dir))


#---------------------------------------------------------------------------------
# options for code generation
#---------------------------------------------------------------------------------
INCLUDE	:=			$(foreach dir,$(INCLUDES),-I$(ROOT)/$(dir)) \
					-I$(ROOT)/$(BUILD) 

#---------------------------------------------------------------------------------
# build a list of library paths
#---------------------------------------------------------------------------------
ARCH	=	-mcpu=arm926ej-s -mtune=arm926ej-s -mthumb -mthumb-interwork -mbig-endian

CFLAGS	=	-g $(ARCH) $(INCLUDE) -fno-strict-aliasing -Wall -O2 -fomit-frame-pointer -ffast-math -fverbose-asm \
			-Wstrict-prototypes -Wpointer-arith -Winline -Wundef -g -ffunction-sections -fdata-sections -fno-exceptions -Wno-strict-prototypes \
			-DHAVE_CONFIG_H -std=gnu99


AFLAGS	=	-g $(ARCH) -x assembler-with-cpp

$(OUTPUT).a: $(OFILES)
	@echo linking $(notdir $@)
	@$(AR) rvs $@ $(OFILES) $(STYLEFIX)
	
%_cpp.o : %.cpp
	@echo $(notdir $<)
	@$(CXX) -MMD -MF $*_cpp.d $(CFLAGS) -c $< -o$@ $(STYLEFIX)

%_c.o : %.c
	@echo $(notdir $<)
	@$(CC) -MMD -MF $*_c.d $(CFLAGS)  -c $< -o$@ $(STYLEFIX)

%_s.o : %.s
	@echo $(notdir $<)
	@$(AS) -MMD -MF $*_s.d $(AFLAGS) -c $< -o$@ $(STYLEFIX)

%_bin.o : %.bin
	@echo $(notdir $<)
	@$(bin2o)

-include $(DEPENDS)
	
define bin2o
	@echo  -e "\t.section .rodata\n\t.align 4\n\t.global $(*)\n\t.global $(*)_end\n$(*):\n\t.incbin \"$(subst /,\\\\\\\\,$(shell echo $< | sed 's=/==;s=/=:/='))\"\n$(*)_end:\n" > $@.s
	@$(CC)  $(ASFLAGS) $(AFLAGS) -c $@.s -o $@
	@rm -rf $@.s
endef

endif
