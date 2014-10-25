DOC = draft-linus-trans-gossip-ct

# Not much user servicable parts below this line.

VER=$(shell grep ^docname: $(DOC).mkd | awk -F- '{print $$NF}')
TEXT=$(DOC)-$(VER).txt
HTML=$(DOC)-$(VER).html
XML=$(DOC).xml

all: $(TEXT) $(HTML) 

$(XML): $(DOC).mkd
	kramdown-rfc2629 $< > $@

$(TEXT): $(XML)
	xml2rfc $<

$(HTML): $(DOC).xml
	xml2rfc $< --html

clean:
	rm -f $(DOC).html $(DOC).txt
