all: outil.pdf

outil.pdf:
	pandoc -r markdown -w latex --filter pandoc-citeproc -o outil.pdf outil.md

.PHONY: clean
clean:
	rm *.pdf
