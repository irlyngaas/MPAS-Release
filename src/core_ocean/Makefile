.SUFFIXES: .F .c .o

OCEAN_SHARED_INCLUDES=-I../shared -I../analysis_members -I../cvmix -I../../framework -I../../external/esmf_time_f90 -I../../operators

OCEAN_LIBRARIES=cvmix/*.o analysis_members/*.o shared/*.o

ifdef MODE

ifeq ($(wildcard ./mode_$(MODE)), ) # CHECK FOR EXISTENCE OF MODE DIRECTORY
all: exit

core_reg: exit

error_msg: error_head
	@echo "$(MODE) is not a valid build mode for the ocean core"

else # IFEQ ($(wildcard....

all: shared libcvmix analysis_members
	(cd mode_$(MODE); $(MAKE) FCINCLUDES="$(FCINCLUDES) $(OCEAN_SHARED_INCLUDES)" )
	if [ -e libdycore.a ]; then \
		($(RM) libdycore.a) \
	fi
	ar -ru libdycore.a $(OCEAN_LIBRARIES) mode_$(MODE)/*.o

core_reg:
	$(CPP) $(CPPFLAGS) $(CPPINCLUDES) Registry.xml > Registry_processed.xml

core_input_gen:
	if [ ! -e default_inputs ]; then  mkdir default_inputs; fi
	(cd default_inputs; $(NL_GEN) ../Registry_processed.xml namelist.ocean_$(MODE) )
	(cd default_inputs; $(ST_GEN) ../Registry_processed.xml streams.ocean_$(MODE) stream_list.ocean_$(MODE). mutable )

gen_includes:
	$(CPP) $(CPPFLAGS) $(CPPINCLUDES) Registry.xml > Registry_processed.xml
	(if [ ! -d inc ]; then mkdir -p inc; fi) # To generate *.inc files
	(cd inc; $(REG_PARSE) < ../Registry_processed.xml )

post_build:
	if [ ! -e $(ROOT_DIR)/default_inputs ]; then mkdir $(ROOT_DIR)/default_inputs; fi
	cp default_inputs/* $(ROOT_DIR)/default_inputs/.
	( cd $(ROOT_DIR)/default_inputs; for FILE in `ls -1`; do if [ ! -e ../$$FILE ]; then cp $$FILE ../.; fi; done )


endif # IFEQ ($(wildcard....

else # IFDEF MODE

all: exit

core_reg: exit

error_msg: error_head
	@echo "The ocean core requires a build mode."

endif # IFDEF MODE

cvmix_source: get_cvmix.sh
	(chmod a+x get_cvmix.sh; ./get_cvmix.sh)
	(cd cvmix; make clean)

libcvmix: cvmix_source
	if [ -d cvmix ]; then \
		(cd cvmix; make all FC="$(FC)" FCFLAGS="$(FFLAGS)" FINCLUDES="$(FINCLUDES)") \
	else \
		(exit 1) \
	fi

shared: libcvmix
	(cd shared; $(MAKE) FCINCLUDES="$(FCINCLUDES) $(OCEAN_SHARED_INCLUDES)")

analysis_members: libcvmix shared
	( cd analysis_members; $(MAKE) FCINCLUDES="$(FCINCLUDES) $(OCEAN_SHARED_INCLUDES)" CPPFLAGS="$(CPPFLAGS)" CPPINCLUDES="$(CPPINCLUDES)" all ) 

error_head:
	@echo ""
	@echo ""
	@echo "*************************************"
	@echo "ERROR"

error_tail: error_head error_msg
	@echo "Available build modes are:"
	@ls -d mode_* | grep ".*" | sed "s/mode_/    /g"
	@echo ""
	@echo "Please specify at build time as follows:"
	@echo "    make target CORE=ocean MODE=build_mode"
	@echo "*************************************"
	@echo ""
	@echo ""

exit: error_head error_msg error_tail
	@exit 1

clean:
	if [ -d cvmix ]; then \
		(cd cvmix; make clean) \
	fi
	(cd mode_forward; $(MAKE) clean)
	(cd mode_analysis; $(MAKE) clean)
	(cd analysis_members; $(MAKE) clean)
	(cd shared; $(MAKE) clean)
	($(RM) *.mod libdycore.a Registry_processed.xml)
	$(RM) -r default_inputs
