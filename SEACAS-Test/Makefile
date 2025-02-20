NPROCA = 31
NPROCB = 32
NPROCC = 33

BIN = ../bin
PYTHON  =       python
APREPRO	=	$(BIN)/aprepro
FASTQ	=	$(BIN)/fastq
GJOIN	=	$(BIN)/gjoin
GEN3D	=	$(BIN)/gen3d
GENSHELL=	$(BIN)/genshell
GREPOS	=	$(BIN)/grepos
IO_SHELL = 	$(BIN)/io_shell
EXOTXT  =       $(BIN)/exotxt
TXTEXO  =       $(BIN)/txtexo
EPU     =       $(BIN)/epu
EXODIFF =       $(BIN)/exodiff
DECOMP  =       $(BIN)/decomp
EXO2MAT =       $(BIN)/exo2mat
MAT2EXO =       $(BIN)/mat2exo
CONJOIN =       $(BIN)/conjoin
RM	=	rm -f

.SUFFIXES: .g .fsq .g3

.fsq.g:
	$(RM) $@
	$(FASTQ) -aprepro -mesh=$*.g $*.fsq

test:	baseline.g3 new_tape.g3 new_shell.g3 slice conv_text matlab python conjoin

base_o.g3: base_o.g base_o.g3i
	$(APREPRO) base_o.g3i |	$(GEN3D) base_o.g base_o.g3

base.g3: base.g base.g3i
	$(APREPRO) base.g3i | $(GEN3D)  base.g base.g3

base_sh.g3: base_sh.g base_sh.g3i
	$(APREPRO) base_sh.g3i | $(GENSHELL)  base_sh.g base_sh.g3

cap.g3: cap.g cap.g3i
	$(APREPRO) cap.g3i | $(GEN3D)  cap.g cap.g3

new_tape.g3: base.g3 base_o.g3 rigid.g3 cap.g3 new_tape.gj
	$(GJOIN) <new_tape.gj

new_shell.g3: base_sh.g3 base_o.g3 rigid.g3 cap.g3 new_shell.gj
	$(GJOIN) <new_shell.gj

rigid.g3: rigid.g rigid.g3i
	$(GEN3D) rigid.g rigid.g3 <rigid.g3i

#-----------------------------
# NOTE: Add names to blocks, nsets, ssets
baseline.g: baseline.g3 names.grp
	$(GREPOS) baseline.g3 baseline.g < names.grp

baseline.g3: baseline_o.g3  rigid_0.g3 base.g3 baseline.gj cap.g3
	$(GJOIN) <baseline.gj

baseline_o.g3: baseline_o.g base.g3i
	$(APREPRO) base_o.g3i |	$(GEN3D) baseline_o.g baseline_o.g3

rigid_0.g3: rigid.g3 rigid.grp
	$(APREPRO) rigid.grp | $(GREPOS)  rigid.g3 rigid_0.g3

# NOTE: This conversion should preserve the block, nset, and sset names
#       that are in baseline.g
slice: slicea sliceb slicec

slicea: baseline.g
	$(DECOMP) -p $(NPROCA) --subdir mesh baseline.g
	ls -l mesh/baseline.g.$(NPROCA).*
	$(EPU) --max_open_files 10 -output_extension e -auto mesh/baseline.g
	$(EXODIFF) -stat baseline.g baseline.e

sliceb: baseline.g
	$(DECOMP) -p $(NPROCB) --subdir mesh baseline.g
	ls -l mesh/baseline.g.$(NPROCB).*
	$(EPU) --max_open_files 8 -output_extension e -auto mesh/baseline.g
	$(EXODIFF) -stat baseline.g baseline.e

slicec: baseline.g
	$(DECOMP) -p $(NPROCC) --subdir mesh baseline.g
	ls -l mesh/baseline.g.$(NPROCC).*
	$(EPU) -output_extension e -auto mesh/baseline.g
	$(EXODIFF) -stat baseline.g baseline.e

# test-gap.g has some nodesets with df and some without
conv_text: test-gap.g.et
	cp test-gap.g.et test-gap.g
	$(EXOTXT) test-gap.g test-gap.t
	$(TXTEXO) test-gap.t test-gap.gt
	$(EXODIFF) -stat test-gap.g test-gap.gt

matlab: baseline.g
	$(EXO2MAT) baseline.g
	$(MAT2EXO) baseline.mat
	$(EXODIFF) -stat baseline.g baseline.exo

base_ioshell.g:
	$(IO_SHELL) --in_type generated 10x10x10+nodeset:xyz+sideset:XYZ+times:5+variables:global,10,element,5,nodal,2,surface,3,nodeset,4 base_ioshell.g

python: baseline.g base_ioshell.g
	$(RM) baseline*copy*.e
	$(RM) base_ioshell*copy*.e
	PYTHONPATH=../lib:$$PYTHONPATH $(PYTHON) test_exo.py
	$(EXODIFF) -stat -pedantic baseline.g baseline_numpy_copy.e
	$(EXODIFF) -stat -pedantic baseline.g baseline_ctype_copy.e
	$(EXODIFF) -stat -pedantic base_ioshell.g base_ioshell_copy.e

conjoin:
	$(IO_SHELL) --in_type generated --split_times 1 10x10x10+times:2000+variables:nodal,3,element,6 conjoin/output.g
	$(CONJOIN) --element_status NONE --nodal_status NONE conjoin/output.g_*
	$(IO_SHELL) --in_type generated 10x10x10+times:2000+variables:nodal,3,element,6 gold.g
	$(EXODIFF) -stat -pedantic conjoin-out.e gold.g

exodiff_partial:
	$(EXODIFF)  -p 8-block.es 1-block.es
	$(EXODIFF)  -p 8-block.es 4-block.es
	$(EXODIFF)  -p 1-block.es 8-block.es
	$(EXODIFF)  -p 4-block.es 8-block.es
	$(EXODIFF)  -p 1-block.es 4-block.es
############################# Default Rules#####################
# Dependencies:
base_o.g: common.h
base.g: common.h

clean:
	$(RM) *.g3 *.g *.e *.gt *.t *~ core*
	$(RM) -r mesh/ tmp.g3d
	$(RM) *.decomp.out *.nem *.pex
	$(RM) *.mat *.exo
	$(RM) baseline_*_copy.e baseline_copy.e
	$(RM) -r conjoin
