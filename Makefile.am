## Process this file with automake to produce Makefile.in

# Include M4 macros
ACLOCAL_AMFLAGS = -I m4

# Subdirectories
SUBDIRS = src pyext test caldb models examples scripts

# Pkgconfig directory
pkgconfigdir = $(libdir)/pkgconfig

# Files to install in Pkgconfig directory
pkgconfig_DATA = ctools.pc

# Manual pages to install
man_MANS = 

# Architecture independent data files installed in the package's doc directory
doc_DATA = \
	   AUTHORS \
	   ChangeLog \
	   COPYING \
	   INSTALL \
	   NEWS \
	   README
docdir = ${datadir}/doc/${PACKAGE}

install-data-local: $(HTML_INSTALL_DATA_TARGETS)

uninstall-local: $(HTML_UNINSTALL_DATA_TARGETS)

#
# Additional rules
#

# Print out all rules
help:
	@echo "make targets:"; \
	echo " all       - build all"; \
    echo " check     - unit test"; \
    echo " clean     - clean build"; \
    echo " install   - install build (may required admin privileges)"; \
    echo " doxygen   - generate Doxygen documentation"; \
    echo " sphinx    - generate Sphinx documentation"; \
    echo " cppcheck  - perform cppcheck"; \
    echo " sloccount - perform Source Lines of Code count"; \
    echo " cppncss   - perform cppncss complexity check"; \
    echo " coverage  - determine code coverage"; \
    echo " valgrind  - search for memory leaks"; \
    echo " doc: doxygen sphinx"; \
    echo " static-analysis: cppcheck sloccount cppncss"; \
    echo " dynamic-analysis: coverage valgrind"

# Add static-analysis rule:
static-analysis: cppcheck sloccount cppncss

# Add dynamic-analysis rule:
dynamic-analysis: coverage valgrind

# Add cppcheck rule
cppcheck:
	@if type cppcheck >/dev/null 2>&1; then \
		cppcheck -v --enable=all --xml \
		src 2> cppcheck-result.xml; \
	else \
		echo "cppcheck not found in PATH."; \
	fi

# Add sloccount rule
sloccount:
	@if type sloccount >/dev/null 2>&1; then \
		sloccount --duplicates --wide --details \
		src test > sloccount.sc; \
	else \
		echo "sloccount not found in PATH."; \
	fi

# Add cppncss rule
cppncss:
	@if type cppncss >/dev/null 2>&1; then \
		cppncss -r -v -x -k -f=cppncss-raw-report.xml \
		src inst/*/src; \
	else \
		echo "cppncss not found in PATH."; \
	fi

# Add coverage rule
coverage:
	@if type gcovr >/dev/null 2>&1; then \
		gcovr -r . -x -o gcovr-reports-data.xml; \
	else \
		echo "gcovr script not found in PATH."; \
	fi

# Add Valgrind rule
valgrind:
	@if type valgrind >/dev/null 2>&1; then \
		$(MAKE) -C test valgrind; \
	else \
		echo "valgrind not found in PATH."; \
	fi

# Add Sphinx rule
sphinx:
	@if type sphinx-build >/dev/null 2>&1; then \
		sphinx-build -b html doc/source doc/html; \
	else \
		echo "sphinx-build not found in PATH."; \
	fi

# Install Sphinx files
sphinx-install:
	@if test -f $(srcdir)/doc/html/index.html ; then \
	    $(mkinstalldirs) $(DESTDIR)$(docdir)/html && \
	    for file in $(srcdir)/doc/html/*.* ; do \
	        echo "$(INSTALL_DATA) $$file $(DESTDIR)$(docdir)/html" ; \
	        $(INSTALL_DATA) "$$file" $(DESTDIR)$(docdir)/html ; \
	    done ; \
	    cp -r $(srcdir)/doc/html/_sources $(DESTDIR)$(docdir)/html/ ; \
	    cp -r $(srcdir)/doc/html/_static $(DESTDIR)$(docdir)/html/ ; \
	fi

# Add documentation rule:
doc: doxygen sphinx

# Non-Automake subdirectories to distribute
DISTDIRS = doc/html doc/html/images
dist-hook:
	( \
	  builddir=`pwd` ; \
	  cd $(srcdir) && \
	  ( \
	    for dir in $(DISTDIRS) ; do \
	      find $$dir -depth -print | egrep -v '(~$$)|(/CVS)|(/\.#)|(/\.deps)' \
		| cpio -pdum $$builddir/$(distdir) 2> /dev/null ; \
	    done \
	  ) \
	)

# Add Doxygen rules
if DX_COND_doc

# HTML
if DX_COND_html
DX_CLEAN_HTML = @DX_DOCDIR@/html
endif #DX_COND_html

# CHI & CHI
if DX_COND_chm
DX_CLEAN_CHM = @DX_DOCDIR@/chm
if DX_COND_chi
DX_CLEAN_CHI = @DX_DOCDIR@/@PACKAGE@.chi
endif #DX_COND_chi
endif #DX_COND_chm

# MAN
if DX_COND_man
DX_CLEAN_MAN = @DX_DOCDIR@/man
endif #DX_COND_man

# RTF
if DX_COND_rtf
DX_CLEAN_RTF = @DX_DOCDIR@/rtf
endif #DX_COND_rtf

# XML
if DX_COND_xml
DX_CLEAN_XML = @DX_DOCDIR@/xml
endif #DX_COND_xml

# PS
if DX_COND_ps
DX_CLEAN_PS = @DX_DOCDIR@/@PACKAGE@.ps
DX_PS_GOAL = doxygen-ps
doxygen-ps: @DX_DOCDIR@/@PACKAGE@.ps
@DX_DOCDIR@/@PACKAGE@.ps: @DX_DOCDIR@/@PACKAGE@.tag
	   cd @DX_DOCDIR@/latex; \
	   rm -f *.aux *.toc *.idx *.ind *.ilg *.log *.out; \
	   $(DX_LATEX) refman.tex; \
	   $(MAKEINDEX_PATH) refman.idx; \
	   $(DX_LATEX) refman.tex; \
	   countdown=5; \
	   while $(DX_EGREP) 'Rerun (LaTeX|to get cross-references right)' \
	                     refman.log > /dev/null 2>&1 \
	      && test $$countdown -gt 0; do \
	       $(DX_LATEX) refman.tex; \
	       countdown=`expr $$countdown - 1`; \
	   done; \
	   $(DX_DVIPS) -o ../@PACKAGE@.ps refman.dvi
endif #DX_COND_ps

# PDF
if DX_COND_pdf
DX_CLEAN_PDF = @DX_DOCDIR@/@PACKAGE@.pdf
DX_PDF_GOAL = doxygen-pdf
doxygen-pdf: @DX_DOCDIR@/@PACKAGE@.pdf
@DX_DOCDIR@/@PACKAGE@.pdf: @DX_DOCDIR@/@PACKAGE@.tag
	   cd @DX_DOCDIR@/latex; \
	   rm -f *.aux *.toc *.idx *.ind *.ilg *.log *.out; \
	   $(DX_PDFLATEX) refman.tex; \
	   $(DX_MAKEINDEX) refman.idx; \
	   $(DX_PDFLATEX) refman.tex; \
	   countdown=5; \
	   while $(DX_EGREP) 'Rerun (LaTeX|to get cross-references right)' \
	                     refman.log > /dev/null 2>&1 \
	      && test $$countdown -gt 0; do \
	       $(DX_PDFLATEX) refman.tex; \
	       countdown=`expr $$countdown - 1`; \
	   done; \
	   mv refman.pdf ../@PACKAGE@.pdf
endif #DX_COND_pdf

# LaTeX
if DX_COND_latex
DX_CLEAN_LATEX = @DX_DOCDIR@/latex
endif #DX_COND_latex

# Rules
#.PHONY: doxygen-run doxygen-doc $(DX_PS_GOAL) $(DX_PDF_GOAL)

.INTERMEDIATE: doxygen-run $(DX_PS_GOAL) $(DX_PDF_GOAL)

doxygen-run: @DX_DOCDIR@/@PACKAGE@.tag

# Make Doxygen files
doxygen: doxygen-run $(DX_PS_GOAL) $(DX_PDF_GOAL)

# Install Doxygen files
DOCDIRS = html/doxygen man/man3
doxygen-install:
	$(mkinstalldirs) $(DESTDIR)$(docdir)
	@for dir in $(DOCDIRS) ; do \
	  $(mkinstalldirs) $(DESTDIR)$(docdir)/$$dir && \
	  for file in $(srcdir)/doc/$$dir/*.* ; do \
	    echo "$(INSTALL_DATA) $$file $(DESTDIR)$(docdir)/$$dir" ; \
	    $(INSTALL_DATA) "$$file" $(DESTDIR)$(docdir)/$$dir ; \
	  done ; \
	done

# Uninstall Doxygen files
doxygen-uninstall:
	for dir in $(DOCDIRS) ; do \
	  rm -f -r $(DESTDIR)$(docdir)/$$dir ; \
	done

@DX_DOCDIR@/@PACKAGE@.tag: $(DX_CONFIG) $(pkginclude_HEADERS)
	    rm -rf @DX_DOCDIR@
	    $(DX_ENV) $(DX_DOXYGEN) $(srcdir)/$(DX_CONFIG)

DX_CLEANFILES = \
     @DX_DOCDIR@/@PACKAGE@.tag \
     -r \
     $(DX_CLEAN_HTML) \
     $(DX_CLEAN_CHM) \
     $(DX_CLEAN_CHI) \
     $(DX_CLEAN_MAN) \
     $(DX_CLEAN_RTF) \
     $(DX_CLEAN_XML) \
     $(DX_CLEAN_PS) \
     $(DX_CLEAN_PDF) \
     $(DX_CLEAN_LATEX)

endif #DX_COND_doc

#
# Additional install rules
#

# Install HTML files
HTMLDIRS = html html/images
HTML_INSTALL_DATA_TARGETS = install-data-html
install-data-html:
	$(mkinstalldirs) $(DESTDIR)$(docdir)
	@for dir in $(HTMLDIRS) ; do \
	  $(mkinstalldirs) $(DESTDIR)$(docdir)/$$dir && \
	  for file in $(srcdir)/doc/$$dir/*.* ; do \
	    echo "$(INSTALL_DATA) $$file $(DESTDIR)$(docdir)/$$dir" ; \
	    $(INSTALL_DATA) "$$file" $(DESTDIR)$(docdir)/$$dir ; \
	  done ; \
	done

# Uninstall HTML files
HTML_UNINSTALL_DATA_TARGETS = uninstall-data-html
uninstall-data-html:
	for dir in $(HTMLDIRS) ; do \
	  rm -f -r $(DESTDIR)$(docdir)/$$dir ; \
	done

# Automatically reconfigure libtool
LIBTOOL_DEPS = @LIBTOOL_DEPS@
libtool: $(LIBTOOL_DEPS)
	$(SHELL) ./config.status --recheck
