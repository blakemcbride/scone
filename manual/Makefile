PDFLATEX = pdflatex
PDFLATEX_FLAGS = -interaction=nonstopmode -halt-on-error

DOCS = Scone-tutorial Scone-Users-Guide
PDFS = $(DOCS:=.pdf)
AUX_EXTS = aux log toc out fdb_latexmk fls

.PHONY: all clean distclean
.SUFFIXES:

all: $(PDFS)

%.pdf: %.tex
	$(PDFLATEX) $(PDFLATEX_FLAGS) $<
	$(PDFLATEX) $(PDFLATEX_FLAGS) $<

clean:
	rm -f $(foreach ext,$(AUX_EXTS),$(DOCS:=.$(ext)))

distclean: clean
	rm -f $(PDFS)
