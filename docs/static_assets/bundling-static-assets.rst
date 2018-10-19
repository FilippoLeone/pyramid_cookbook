===================================================
Bundling static assets via a Pyramid console script
===================================================

Modern applications often require some kind of build step for bundling static assets for either a development or production environment.
This recipe illustrates how to build a console script that can help with this task.
It also tries to satisfy typical requirements:

* Frontend source code can be distributed as a Python package.
* The source code's repository and site-packages are not written to during the build process.
* Make it possible to provide a plug-in architecture within an application through multiple static asset packages.
* The application's home directory is the destination of the build process to facilitate HTTP serving by a web server.
* Flexible - Allows any frontend toolset (Yarn, Webpack, Rollup, etc.) for JavaScript, CSS, and image bundling to compose bigger pipelines.


Demo
----

This recipe includes a demo application.
The source files are located on GitHub:

https://github.com/Pylons/pyramid_cookbook/tree/master/docs/static_assets/bundling

The demo was generated from the `Pyramid starter cookiecutter <https://github.com/Pylons/pyramid-cookiecutter-starter>`_.

Inside the directory ``bundling`` are two directories:

*   ``bundling_example`` is the Pyramid app generated from the cookiecutter with some additional files and modifications as described in this recipe.
*   ``frontend`` contains the frontend source code and files.

You can generate a project from the starter cookiecutter, install it, then follow along with the rest of this recipe.
If you run into any problems, compare your project with the demo project source files to see what might be amiss.


Requirements
------------

This recipe and the demo application both require `Yarn <https://yarnpkg.com/en/docs/install>`_ and `NodeJS 8.x <https://nodejs.org/en/download/>`_ packages to be installed.


Configure Pyramid
-----------------

First we need to tell Pyramid to serve static content from an additional build directory.
This is useful for development.
In production, often this will be handled by Nginx.

In your configuration file, in the ``[app:main]`` section, add locations for the build process:

.. code-block:: ini

    # build result directory
    statics.dir = %(here)s/static
    # intermediate directory for build process
    statics.build_dir = %(here)s/static_build

In your application's routes, add a static asset view and an asset override configuration:

.. code-block:: py3

    import pathlib
    # after default static view add bundled static support
    config.add_static_view(
        "static_bundled", "static_bundled", cache_max_age=1
    )
    path = pathlib.Path(config.registry.settings["statics.dir"])
    # create the directory if missing otherwise pyramid will not start
    path.mkdir(exist_ok=True)
    config.override_asset(
        to_override="yourapp:static_bundled/",
        override_with=config.registry.settings["statics.dir"],
    )

Now in your templates, reference the built and bundled static assets.

.. code-block:: html

    <script src="{{ request.static_url('yourapp:static_bundled/some-package.min.js') }}"></script>


Console script
--------------

Create a directory ``scripts`` at the root of your application.
Add an empty ``__init__.py`` file to this sub-directory so that it becomes a Python package.
Also in this sub-directory, create a file ``build_static_assets.py`` to serve as a console script to compile assets, with the following code.

.. literalinclude:: bundling/bundling_example/bundling_example/scripts/build_static_assets.py 
    :language: python


Edit your application's ``setup.py`` to create a shell script when you install your application that you will use to start the compilation process.

.. code-block:: py3

    setup(
        name='yourapp',
        ....
        install_requires=requires,
        entry_points={
            'paste.app_factory': [
                'main = channelstream_landing:main',
            ],
            'console_scripts': [
                'yourapp_build_statics = yourapp.scripts.build_static_assets:main',
            ]
        },
    )


Install your app
----------------

Run ``pip install -e`` . again to register the console script.

Now you can configure/run your frontend pipeline with webpack/gulp/rollup or other solution.


Compile static assets
---------------------

Finally we can compile static assets from the frontend and write them into our application.

Run the command:

.. code-block:: bash

    yourapp_build_statics development.ini

This starts the build process.
It creates a fresh ``static`` directory in the same location as your application's ``ini`` file.
The directory should contain all the build process files ready to be served on the web.

You can retrieve variables from your Pyramid application in your Node build configuration files:

.. code-block:: javascript

    destinationRootDir = process.env.FRONTEND_ASSSET_ROOT_DIR

You can view a generated ``pyramid_config.json`` file in your Node script for additional information.
