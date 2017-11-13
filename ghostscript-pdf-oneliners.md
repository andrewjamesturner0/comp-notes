# Merge PDFs with ghostscript


    gs -dBATCH -dNOPAUSE -q -sDEVICE=pdfwrite -sOutputFile=combined.pdf file-1.pdf file-2.pdf file-3.pdf



# Split PDFs with ghostscript


    gs -dBATCH -dNOPAUSE -q -sDEVICE=pdfwrite -sOutputFile=split.pdf -dFirstPage=X -dLastPage=X  file.pdf
