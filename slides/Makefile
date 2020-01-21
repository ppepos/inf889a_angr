all : slides

.PHONY:
slides:
	pandoc -t beamer --slide-level 2 -o slides.pdf slides.md

.PHONY:
clean:
	rm -f $(SLIDES)
	rm -f $(HANDOUTS)

