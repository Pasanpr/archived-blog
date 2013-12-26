public: yes
tags: [python, blog, restructured text]
summary: |
    Setting up my static blog

=======================================================
Setting Up My Static Blog
=======================================================

Roughly about a year ago I moved my blog from Wordpress over to a simpler, static implementation but I never wrote down the process. I did it rather haphazardly, cobbling together from a few different articles without really bothering to make note for the future. Now, wanting to replicate it, and because someone else asked for the details, I figure I should write it down, without having to look back every time.

This blog uses the static blogging engine `rstblog <https://github.com/mitsuhiko/rstblog>`__ developed by `Armin Ronacher <http://lucumr.pocoo.org/about/>`__ and is pretty simple to use. The only issue is that there is very little documentation but given that it's an unmaintained project, you can't really ask for much. 

Rstblog is a static blog generator implemented in Python (one of my main reasons for choosing it), or more simply it is a `reStructured Text <http://docutils.sourceforge.net/rst.html>`__ to HTML converter. 

    reStructuredText is an easy-to-read, what-you-see-is-what-you-get plaintext markup syntax and parser system. It is useful for in-line program documentation (such as Python docstrings), for quickly creating simple web pages, and for standalone documents. 

The generated HTML files can be served where ever you want and you can even use version control systems like Git to track everything. Rstblog includes everything you need for a basic blog including post pages, archive pages, feeds and comments using Disqus. It uses `Jinja <http://jinja.pocoo.org/>`__ (a Python templating engine also developed by Armin R.) under the hood, so you can create custom templates and stylesheets. It also uses `Pygments <http://pygments.org/>`__, a Python syntax highlighter, if you want to include code snippets in your writing. 

Before you can get started using rstblog, there are a few concepts/technology that you need to get familiar with. As someone new to Python there was a lot I didn't know, but setting up the blog provided great practice for later projects.

- `YAML <http://www.yaml.org/>`__: In rstblog you use a YAML file for configuring your blog.
- Virtual environments: Rather than installing everything in one directory on your machine, a virtual environment is *an isolated copy of Python which allows you to work on a specific without worry of affecting other projects. For example, you can work on a project which requires Django 1.3 while also maintaining a project which requires Django 1.0.* You can use `virtualenv <https://pypi.python.org/pypi/virtualenv>`__ to create your Python environments or even easier, use `virtualenvwrapper <http://virtualenvwrapper.readthedocs.org/en/latest/index.html>`__. As the name suggests, it is a wrapper on virtualenv that provides a set of commands to make managing your virtual environments much more pleasant.
- `Fabric <http://docs.fabfile.org/en/1.8/>`__: Fabric is a Python (2.5 or higher) library and command-line tool for streamlining the use of SSH for application deployment or systems administration tasks. It isn't necessary to get the blog up and running, but makes life easier if you want to automate the build and deploy process.
- `pip <https://pypi.python.org/pypi/pip>`__: A tool for installing and managing Python packages

Setting Up the Blog
--------------------

So how do we get this thing up and running:

Clone the rstblog Repo
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. sourcecode:: bash

    $ git clone https://github.com/mitsuhiko/rstblog.git

Create a New Python Environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using virtualenvwrapper create a new Python environment titled 'rstblog_env'.

.. sourcecode:: bash

    $ mkvirtualenv rstblog_env
    $ cd rstblog
    $ pip install .

Create Your Blog
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**1. Start by creating a directory tree** :

This is a separate directory from where we just installed the rstblog (that is our source). Navigate to your preferred directory.

.. sourcecode:: bash

    mkdir -p sample_blog/{static,_templates,2013}

The ``-p`` flag creates the subsequent folders in the sample_blog directory.

rstblog requires a certain directory structure to be able to generate your blog. It creates a blog entry for every file that is in a ``<yyyy>/<mm>/<dd>`` format. We already created our our ``2013`` directory to store our entries, the _templates directory for our custom templates and the static directory for our stylesheets and any other static files like images, js files and so on. Here's what an example directory looks like

.. sourcecode:: bash

        pasanpr.org
    ├── 2013
    │   ├── 11
    │   │   └── 15
    │   │       └── more-stuff.rst
    │   └── 10
    │       └── 01
    │           ├── stuff.rst
    │           └── hello-world.rst
    ├── about.rst
    ├── config.yml
    └── _templates
    │   └── layout.html
    └── static
        └── styles.css

**2. Create a requirements file** :

.. sourcecode:: bash
    
    Pygments==1.5
    Fabric==1.4.3
    cssmin==0.1.4
    lxml==3.2.3

A requirements file contains all of a project's dependencies. Using pip we can batch install all the requirements instead of installing them one by one. The requirements file also allows us to keep track of all the dependencies and version numbers in case we want to recreate the same environment elsewhere. To install the dependencies run:

.. sourcecode:: bash

    $ pip install -r /path/to/requirements.txt 


**3. Create a config.yaml file** :

.. sourcecode:: yaml

    active_modules: [pygments, tags, blog, latex, disqus]
    author: Pasan Premaratne
    canonical_url: http://www.pasanpremaratne.com
    modules:
      pygments:
        style: tango
      disqus:
        shortname: pasanpremaratne

That is my YAML file with all the desired modules specified. Be really careful with whitespace in a YAML file - *it's annoying as hell*.

**4. Include the templates in your directory** : 

Included in our rstblog install is some default templates. Let's copy and edit those to suit our needs. From your blog directory run (replacing *<path_to_rstblog>* with your path) :

.. sourcecode:: bash
    
    $ cp -r <path_to_rstblog>/rstblog/templates/* _templates/

**5. Create a layout.html file** : 

Create ``_templates/layout.html`` and edit as you desire. I've pasted my layout file below so you get an idea. You can also check out Ronacher's `layout file <https://github.com/mitsuhiko/lucumr/blob/master/_templates/layout.html>`__:

.. sourcecode:: jinja

    <!doctype html>
    <html>
      <head>
        <meta charset=utf-8>

      {% block htmlhead %}
        <title>{% block title %}Home{% endblock %}</title>
        <link rel="stylesheet" href="/static/style.css" type="text/css">
        <link href="/feed.atom" rel="alternate" title="Recent Blog Posts" type="application/atom+xml">
        {%- for link in links %}
        <link rel="{{ link.rel }}" href="{{ link.href }}"{%
          if link.media %} media="{{ link.media }}"{% endif %} type="{{ link.type }}">
        {%- endfor %}
      {% endblock %}
        <script type="text/javascript" src="//use.typekit.net/tcx7ena.js"></script>
        <script type="text/javascript">try{Typekit.load();}catch(e){}</script>
      </head>
      <body>
        <header role="banner">
          <div class="container">
            <a class="header-title" href="/">Pasan Premaratne</a>
            <ul class="headlinks">
                <li><a href="/archive/">Archive</a></li>
                <li><a href="/about/">About</a></li>
            </ul>
          </div>
        </header>
        <section role="main" class="container">
          {% block body %}{% endblock %}
        </section>
        <footer id="footer">
           <section class="container">
            <p>
              <a href="https://twitter.com/pasanpr" class="twitter-follow-button" data-show-count="false" data-size="large">Follow @pasanpr</a>
    <script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+'://platform.twitter.com/widgets.js';fjs.parentNode.insertBefore(js,fjs);}}(document, 'script', 'twitter-wjs');</script>
            </p>
            <p>&copy; {{ format_date(format='YYYY') }} Pasan Premaratne</p>
           </section>
        </footer>
        <script type="text/javascript">

      var _gaq = _gaq || [];
      _gaq.push(['_setAccount', 'UA-28086501-1']);
      _gaq.push(['_setDomainName', 'pasanpremaratne.com']);
      _gaq.push(['_trackPageview']);

      (function() {
        var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
        ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
        var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
      })();

        </script>
      </body>
    </html> 

At this point you have created your blog. But how do we create an entry and publish it. 

Creating a Blog Entry
-----------------------

Rstblog creates dates and post urls based on your directory structure. We already created a 2013 folder. Let's go ahead and create an entry under December 26th. Navigate to the 2013 directory and run:

.. sourcecode:: bash

    $ mkdir -p 12/26

Create and edit your first blog post 2013/12/26/first-post.rst using reStructured Text.

.. sourcecode:: ReST

    public: yes
    tags: [thoughts, rstblog] 
    summary: |
        I'm tired of Wordpress, here's where I landed

    ============================
    Good Bye Wordpress
    ============================

    I have nothing against Wordpress; I just don't blog heavily enough for it to be my platform of choice.

To get this entry up on our blog we need to build first:

.. sourcecode:: bash
    
    $ run-rstblog build

This generates a ``_build`` folder. To see the results:

.. sourcecode:: bash

    $ run-rstblog serve

At this point, you're good to go. To publish your blog all you need to do is copy your _build folder to your public html folder.

Workfow Tips
---------------

There's a few more things you can do to make your workflow much easier. 

1. Use Github to track everything (Git fundamentals are outside the scope of this post but there's plenty of resources online).

2. Use `Fabric <http://docs.fabfile.org/en/1.8/>`__ and `rsync <http://rsync.samba.org/>`__ to automate some of the tasks. My Fab file is as follows. Copy and replace with your info to use it. 

.. sourcecode:: python

    import time
    from fabric.api import local, env
    from fabric.contrib.project import rsync_project

    env.hosts = ['<insert ftp host here>']
    env.user = '<insert ftp username here>'
    env.path = '<insert path here>'

    def push():
        local('git push origin master')

    def serve():
        local('run-rstblog serve')

    def build():
        # Build HTML
        local('rm -rf _build/ && run-rstblog build')

        # Generate sitemaps
        local('python gensitemap.py > _build/sitemap.xml')

        # Minify CSS
        local('cssmin < _build/static/style.css > _build/static/style.min.css')
        local('mv _build/static/style.min.css _build/static/style.css')
        local('cssmin < _build/static/_pygments.css > _build/static/_pygments.min.css')
        local('mv _build/static/_pygments.min.css _build/static/_pygments.css')

        # Add timestamp to css files
        local('find _build -type f -exec sed -i "s/\(link.*\)style.css/\\1style.css?%s/g" {} \;' % int(time.time()))
        local('find _build -type f -exec sed -i "s/\(link.*\)_pygments.css/\\1_pygments.css?%s/g" {} \;' % int(time.time()))

    def sync():
        rsync_project(remote_dir=env.path,
                      local_dir='_build/',
                      delete=True,
                      exclude=['*.py', '*.pyc', 'requirements.txt'])

    def deploy():
        build()
        sync()

    def publish():
        deploy()
        push() 

To run:

.. sourcecode:: bash
    
    $ fab publish

When you run that command, here's what happens in order:

- You build the _build directory containing all the necessary files
- Sitemaps are generated, css files are minified and timestamped
- The _build directory is uploaded to your server
- The repo is pushed to Github

If you want to poke around more, check out my `repo <https://github.com/Pasanpr/blog>`__.

Thanks to the following folks for all the information online:

- `Goodbye Wordpress hi rstblog <http://blog.dbrgn.ch/2012/6/11/rstblog/>`__
- `mitsuhikos rstblog on Github <http://eolo999.github.io/2012/9/19/how_to_rstblog_on_github/>`__
- `About using the static blogging engine rstblog <http://nblock.org/2011/08/31/1st-blogpost/>`__












