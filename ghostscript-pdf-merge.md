# Merge PDFs with ghostscript


    gs -dBATCH -dNOPAUSE -q -sDEVICE=pdfwrite -sOutputFile=combined.pdf file-1.pdf file-2.pdf file-3.pdf
