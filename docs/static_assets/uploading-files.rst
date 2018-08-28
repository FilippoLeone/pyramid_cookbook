===============
Uploading Files
===============

There are two parts necessary for handling file uploads.  The first is to
make sure you have a form that's been setup correctly to accept files.  This
means adding ``enctype`` attribute to your ``form`` element with the value of
``multipart/form-data``.  A very simple example would be a form that accepts
an mp3 file.  Notice we've setup the form as previously explained and also
added an ``input`` element of the ``file`` type.

.. code-block:: html
    :linenos:

    <form action="/store_mp3_view" method="post" accept-charset="utf-8"
          enctype="multipart/form-data">

        <label for="mp3">Mp3</label>
        <input id="mp3" name="mp3" type="file" value="" />

        <input type="submit" value="submit" />
    </form>

The second part is handling the file upload in your view callable (above,
assumed to answer on ``/store_mp3_view``).  The uploaded file is added to the
request object as a ``cgi.FieldStorage`` object accessible through the
``request.POST`` multidict.  The two properties we're interested in are the
``file`` and ``filename`` and we'll use those to write the file to disk:

.. code-block:: python

    import os
    import shutil

    from pyramid.response import Response

    def store_mp3_view(request):
        # ``filename`` contains the name of the file in string format.
        #
        # WARNING: Internet Explorer is known to send an absolute file
        # *path* as the filename.  This example is naive; it trusts
        # user input.
        filename = request.POST['mp3'].filename

        # ``input_file`` contains the actual file data which needs to be
        # stored somewhere.
        input_file = request.POST['mp3'].file

        # Using the filename like this without cleaning it is very
        # insecure so please keep that in mind when writing your own
        # file handling.
        file_path = os.path.join('/tmp', filename)
        with open(file_path, 'wb') as output_file:
            shutil.copyfileobj(input_file, output_file)

        return Response('OK')
