#**************************************************************************
#*                                                                        *
#*                                OCaml                                   *
#*                                                                        *
#*                 Xavier Clerc, SED, INRIA Rocquencourt                  *
#*                                                                        *
#*   Copyright 2010 Institut National de Recherche en Informatique et     *
#*     en Automatique.                                                    *
#*                                                                        *
#*   All rights reserved.  This file is distributed under the terms of    *
#*   the GNU Lesser General Public License version 2.1, with the          *
#*   special exception on linking described in the file LICENSE.          *
#*                                                                        *
#**************************************************************************

BASEDIR=../..

INCLUDES=\
  -I $(OTOPDIR)/utils \
  -I $(OTOPDIR)/typing \
  -I $(OTOPDIR)/middle_end \
  -I $(OTOPDIR)/bytecomp \
  -I $(OTOPDIR)/asmcomp

OTHEROBJS=\
  $(OTOPDIR)/compilerlibs/ocamlcommon.cma \
  $(OTOPDIR)/compilerlibs/ocamloptcomp.cma

OBJS=parsecmmaux.cmo parsecmm.cmo lexcmm.cmo

ADD_COMPFLAGS=$(INCLUDES) -w -40 -g

default:
	@if $(BYTECODE_ONLY) || $(SKIP) ; then $(MAKE) skips ; else \
	  $(MAKE) all; \
	fi

all:
	@$(MAKE) arch codegen
	@$(MAKE) tests

codegen: parsecmm.ml lexcmm.ml $(OBJS:.cmo=.cmi) $(OBJS) main.cmo
	@$(OCAMLC) $(LINKFLAGS) -o codegen $(OTHEROBJS) $(OBJS) main.cmo

parsecmm.mli parsecmm.ml: parsecmm.mly
	@$(OCAMLYACC) -q parsecmm.mly

lexcmm.ml: lexcmm.mll
	@$(OCAMLLEX) -q lexcmm.mll

MLCASES=optargs staticalloc bind_tuples is_static register_typing \
  register_typing_switch
ARGS_is_static=-I $(OTOPDIR)/byterun is_in_static_data.c
MLCASES_FLAMBDA=is_static_flambda unrolling_flambda unrolling_flambda2 \
  static_float_array_flambda static_float_array_flambda_opaque
ARGS_is_static_flambda=\
  -I $(OTOPDIR)/byterun is_in_static_data.c is_static_flambda_dep.ml
ARGS_static_float_array_flambda=\
  -I $(OTOPDIR)/byterun is_in_static_data.c simple_float_const.ml
ARGS_static_float_array_flambda_opaque=\
  -I $(OTOPDIR)/byterun is_in_static_data.c -opaque simple_float_const_opaque.ml

CASES=fib tak quicksort quicksort2 soli \
      arith checkbound tagged-fib tagged-integr tagged-quicksort tagged-tak
ARGS_fib=-ccopt -DINT_INT -ccopt -DFUN=fib main.c
ARGS_tak=-ccopt -DUNIT_INT -ccopt -DFUN=takmain main.c
ARGS_quicksort=-ccopt -DSORT -ccopt -DFUN=quicksort main.c
ARGS_quicksort2=-ccopt -DSORT -ccopt -DFUN=quicksort main.c
ARGS_soli=-ccopt -DUNIT_INT -ccopt -DFUN=solitaire main.c
ARGS_integr=-ccopt -DINT_FLOAT -ccopt -DFUN=test main.c
ARGS_arith=mainarith.c
ARGS_checkbound=-ccopt -DCHECKBOUND main.c
ARGS_tagged-fib=-ccopt -DINT_INT -ccopt -DFUN=fib main.c
ARGS_tagged-integr=-ccopt -DINT_FLOAT -ccopt -DFUN=test main.c
ARGS_tagged-quicksort=-ccopt -DSORT -ccopt -DFUN=quicksort main.c
ARGS_tagged-tak=-ccopt -DUNIT_INT -ccopt -DFUN=takmain main.c
ARGS_staticalloc=-I $(OTOPDIR)/utils config.cmx

skips:
	@for c in $(CASES) $(MLCASES) $(MLCASES_FLAMBDA); do \
	  echo " ... testing '$$c': => skipped"; \
	done

one_ml:
	@$(OCAMLOPT) $(ARGS_$(NAME)) -o $(NAME).exe $(NAME).ml && \
	./$(NAME).exe && echo " => passed" || echo " => failed"

one_ml_flambda:
	@if $(FLAMBDA); then \
	   $(OCAMLOPT) $(ARGS_$(NAME)) -o $(NAME).exe $(NAME).ml && \
	   ./$(NAME).exe && echo " => passed" || echo " => failed"; \
	 else \
	   echo " => skipped"; \
	 fi

one:
	@$(OCAMLOPT) -o $(NAME).out $(ARGS_$(NAME)) $(NAME).$(O) $(ARCH).$(O) \
	&& echo " => passed" || echo " => failed"

clean: defaultclean
	@rm -f ./codegen *.out *.out.manifest *.$(O) *.exe
	@rm -f parsecmm.ml parsecmm.mli lexcmm.ml
	@rm -f $(CASES:=.s)

include $(BASEDIR)/makefiles/Makefile.common

ifeq "$(CCOMPTYPE)-$(ARCH)" "msvc-amd64"
# these tests are not ported to MSVC64 yet
SKIP=true
else
SKIP=false
endif

ifeq "$(WITH_SPACETIME)" "true"
# These tests have not been ported for Spacetime
SKIP=true
endif

ifeq ($(CCOMPTYPE),msvc)
CC=set -o pipefail ; $(NATIVECC) $(CFLAGS) /Fe$(1) | tail -n +2
CFLAGS=$(NATIVECCCOMPOPTS) -I$(OTOPDIR)/byterun
else
CC=$(NATIVECC) $(CFLAGS) -o $(1)
CFLAGS=$(NATIVECCCOMPOPTS) -g -I$(OTOPDIR)/byterun
endif
tests: $(CASES:=.$(O))
	@for c in $(CASES); do \
	  printf " ... testing '$$c':"; \
	  $(MAKE) one NAME=$$c; \
	done
	@for c in $(MLCASES); do \
	  printf " ... testing '$$c':"; \
          $(MAKE) one_ml NAME=$$c; \
        done
	@for c in $(MLCASES_FLAMBDA); do \
	  printf " ... testing '$$c':"; \
          $(MAKE) one_ml_flambda NAME=$$c; \
        done

promote:

arch: $(ARCH).$(O)

i386.obj: i386nt.asm
	@set -o pipefail ; \
	$(ASM) $@ $^ | tail -n +2
