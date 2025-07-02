http-parser
-----------

HTTP request/response parser for Python.
If possible a C parser based on
http-parser_ from Ryan Dahl will be used.

http-parser is under the MIT license.

Project url: https://github.com/adamnovak/http-parser/

Forked from the original ``http-parser``: https://github.com/benoitc/http-parser/

Requirements:
-------------

- Python 3.9+ or pypy.
- Cython (for C extension)

Installation
------------

::

    $ pip install http-parser@git://github.com/adamnovak/http-parser.git

Or install from source::

    $ git clone git://github.com/adamnovak/http-parser.git
    $ cd http-parser && pip install .

Note that if you ``import http_parser`` from within the root of the project,
you will load the source code from the ``http_parser/`` directory and not the
installed module.

Usage
-----

http-parser provide you **parser.HttpParser**, a low-level parser in C that
you can access in your Python program, and **http.HttpStream**, providing
higher-level access to a readable, sequential io.RawIOBase object.

To help you in your daily work, http-parser provides you 3 kind of readers
in the reader module: IterReader to read iterables, StringReader to
reads strings and StringIO objects, and SocketReader to read sockets or
objects with the same API (i.e. those implementing ``recv_into``). You can of
course use any io.RawIOBase object.

Example of HttpStream
+++++++++++++++++++++

ex::

    #!/usr/bin/env python
    import socket

    from http_parser.http import HttpStream
    from http_parser.reader import SocketReader

    def main():
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        try:
            s.connect(('gunicorn.org', 80))
            s.send("GET / HTTP/1.1\r\nHost: gunicorn.org\r\n\r\n")
            r = SocketReader(s)
            p = HttpStream(r)
            print p.headers()
            print p.body_file().read()
        finally:
            s.close()

    if __name__ == "__main__":
        main()

Example of HttpParser:
++++++++++++++++++++++

::

    #!/usr/bin/env python
    import socket

    # try to import C parser then fallback in pure python parser.
    try:
        from http_parser.parser import HttpParser
    except ImportError:
        from http_parser.pyparser import HttpParser


    def main():

        p = HttpParser()
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        body = []
        try:
            s.connect(('gunicorn.org', 80))
            s.send("GET / HTTP/1.1\r\nHost: gunicorn.org\r\n\r\n")

            while True:
                data = s.recv(1024)
                if not data:
                    break

                recved = len(data)
                nparsed = p.execute(data, recved)
                assert nparsed == recved

                if p.is_headers_complete():
                    print p.get_headers()

                if p.is_partial_body():
                    body.append(p.recv_body())

                if p.is_message_complete():
                    break

            print "".join(body)

        finally:
            s.close()

    if __name__ == "__main__":
        main()


You can find more docs in the code (or use a doc generator).


Copyright
---------

2011-2020 (c) Benoît Chesneau <benoitc@e-engura.org>


.. http-parser_ https://github.com/ry/http-parser
