# GNU Makefile template for user ESMF application

################################################################################
################################################################################
## This Makefile must be able to find the "esmf.mk" Makefile fragment in the  ##
## 'include' line below. Following the ESMF User's Guide, a complete ESMF     ##
## installation should ensure that a single environment variable "ESMFMKFILE" ##
## is made available on the system. This variable should point to the         ##
## "esmf.mk" file.                                                            ##
##                                                                            ##
## This example Makefile uses the "ESMFMKFILE" environment variable.          ##
##                                                                            ##
## If you notice that this Makefile cannot find variable ESMFMKFILE then      ##
## please contact the person responsible for the ESMF installation on your    ##
## system.                                                                    ##
## As a work-around you can simply hardcode the path to "esmf.mk" in the      ##
## include line below. However, doing so will render this Makefile a lot less ##
## flexible and non-portable.                                                 ##
################################################################################

ifneq ($(origin ESMFMKFILE), environment)
$(error Environment variable ESMFMKFILE was not set.)
endif

include $(ESMFMKFILE)

#localFopt = -C
#localFopt = -Wunused
#localFopt = -Wuninitialized
#localFopt = -Wall
#localFopt = -ffpe-trap=invalid,zero,overflow

################################################################################
################################################################################

.SUFFIXES: .f90 .F90 .c .C

%.o : %.f90
	$(ESMF_F90COMPILER) $(localFopt) -c $(ESMF_F90COMPILEOPTS) $(ESMF_F90COMPILEPATHS) $(ESMF_F90COMPILEFREENOCPP) $<

%.o : %.F90
	$(ESMF_F90COMPILER) $(localFopt) -c $(ESMF_F90COMPILEOPTS) $(ESMF_F90COMPILEPATHS) $(ESMF_F90COMPILEFREECPP) $(ESMF_F90COMPILECPPFLAGS) $<
        
%.o : %.c
	$(ESMF_CXXCOMPILER) -c $(ESMF_CXXCOMPILEOPTS) $(ESMF_CXXCOMPILEPATHSLOCAL) $(ESMF_CXXCOMPILEPATHS) $(ESMF_CXXCOMPILECPPFLAGS) $<

%.o : %.C
	$(ESMF_CXXCOMPILER) -c $(ESMF_CXXCOMPILEOPTS) $(ESMF_CXXCOMPILEPATHSLOCAL) $(ESMF_CXXCOMPILEPATHS) $(ESMF_CXXCOMPILECPPFLAGS) $<
# ------------------------------------------------------------------------------
.PHONY: nuopc nuopcinstall
nuopc: datm.mk

nuopcinstall: install

# ------------------------------------------------------------------------------
AtmOBJS=datm.o AtmInternalFields.o AtmFieldUtils.o AtmForce.o AtmModel.o LocalDefs.o AtmBundleCreate.o AtmGridUtils.o

datm.o: $(AtmOBJS)

datm.o: AtmModel.o AtmFieldUtils.o AtmGridUtils.o
AtmModel.o: AtmFieldUtils.o
AtmForce.o: AtmInternalFields.o
AtmBundleCreate.o : AtmInternalFields.o
AtmFieldUtils.o : AtmInternalFields.o
AtmExportFields.o : AtmInternalFields.o
# -----------------------------------------------------------------------------
.PRECIOUS: %.o
%.mk : %.o
	@echo "# ESMF self-describing build dependency makefile fragment" > $@ 
	@echo >> $@
	@echo "ESMF_DEP_FRONT = datm"                >> $@
	@echo "ESMF_DEP_INCPATH = `pwd`"             >> $@
	@echo "ESMF_DEP_CMPL_OBJS = `pwd`/"$<        >> $@
	@echo "ESMF_DEP_LINK_OBJS = `pwd`/libdatm.a" >> $@
	@echo "ESMF_DEP_SHRD_PATH = "                >> $@
	@echo "ESMF_DEP_SHRD_LIBS = "                >> $@

datm.mk: libdatm.a

libdatm.a: datm.o $(AtmOBJS)
	ar cr $@ $^

# -----------------------------------------------------------------------------
.PHONY: clean distclean info install
clean:
	rm -f *.a *.mk *.o *.mod *.nc esmApp PET*
distclean: clean
	rm -f *.o *.mod

PWDIR := `pwd`

info:
	@echo ==================================================================
	@echo ESMFMKFILE=$(ESMFMKFILE)
	@echo ==================================================================
	@cat $(ESMFMKFILE)
	@echo ==================================================================

install: datm.mk
	@mkdir -p $(DESTDIR)/$(INSTDIR)
	@cp libdatm.a datm.mod $(DESTDIR)/$(INSTDIR)
	@sed -e 's;'$(PWDIR)';'$(DESTDIR)/$(INSTDIR)';g' datm.mk > $(DESTDIR)/$(INSTDIR)/datm.mk
	@echo Installation into \"$(DESTDIR)/$(INSTDIR)\" complete!
	@echo
