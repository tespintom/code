## ReseekFile

Originally published: 2003-01-09 16:09:13
Last updated: 2003-01-09 16:09:13
Author: Andrew Dalke

Wrap a file handle to allow seeks back to the beginning\n\nSometimes data coming from a socket or other input file handle isn't\nwhat it was supposed to be.  For example, suppose you are reading from\na buggy server which is supposed to return an XML stream but can also\nreturn an unformatted error message.  (This often happens because the\nserver doesn't handle incorrect input very well.)\n\nA ReseekFile helps solve this problem.  It is a wrapper to the\noriginal input stream but provides a buffer.  Read requests to the\nReseekFile get forwarded to the input stream, appended to a buffer,\nthen returned to the caller.  The buffer contains all the data read so\nfar.\n\nThe ReseekFile can be told to reseek to the start position.  The next\nread request will come from the buffer, until the buffer has been\nread, in which case it gets the data from the input stream.  This\nnewly read data is also appended to the buffer.\n\nWhen buffering is no longer needed, use the 'nobuffer()' method.  This\ntells the ReseekFile that once it has read from the buffer it should\nthrow the buffer away.  After nobuffer is called, the behaviour of\n'seek' is no longer defined.\n\nFor example, suppose you have the server as above which either\ngives an error message is of the form:\n\n&nbsp;&nbsp;ERROR: cannot do that\n\nor an XML data stream, starting with "<?xml".\n\n&nbsp;&nbsp;  infile = urllib2.urlopen("http://somewhere/")\n&nbsp;&nbsp;  infile = ReseekFile.ReseekFile(infile)\n&nbsp;&nbsp;  s = infile.readline()\n&nbsp;&nbsp;  if s.startswith("ERROR:"):\n&nbsp;&nbsp;&nbsp;&nbsp;      raise Exception(s[:-1])\n&nbsp;&nbsp;  infile.seek(0)\n&nbsp;&nbsp;  infile.nobuffer()   # Don't buffer the data\n&nbsp;&nbsp;   ... process the XML from infile ...\n\n\nThis module also implements 'prepare_input_source(source)' modeled on\nxml.sax.saxutils.prepare_input_source.  This opens a URL and if the\ninput stream is not already seekable, wraps it in a ReseekFile.