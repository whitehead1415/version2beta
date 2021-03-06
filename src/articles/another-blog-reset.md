---
pagetitle: Another blog reset
longtitle: I rebuilt my blog again.
tags: personal
author: Rob Martin
published: 2013-01-27 17:45
snippet: Redesign isn't the right word. It seems when I want to freshen my blog, I get down to the nuts and bolts.
---

Last time I reset my blog,[^reset] I chose a static site generator called Hyde[^hyde] because it is based on Python and Django[^django]. It was a good choice at the time, but since then it's become heavy and I got tired of carrying it. In fact, in May 2012 I accidentally deleted my blog dev virtualenv environment and decided then I'd rather move the site to a new platform than rebuild the environment. After all, I keep telling myself I need to learn Jekyll[^jekyll]. Or Sinatra.[^sinatra] Or I could build it in Tako[^tako] on Node.js. In fact, at various times I think I started all of those projects.

So after a mere 8 months of false starts and procrastination, I managed to move my blog. And make it prettier.

## The new setup

The new platform is based on a handful of interesting technologies that I wanted to learn more about:

* Amazon Web Services Route53 and S3 Simple Storage Service
* Vagrant and Chef
* Python Flask microframework
* YAML and Markdown

### Amazon Web Services

Amazon Web Services[^aws] offers a bunch of technologies at affordable pay-per-use prices. I did a largish project on AWS a couple years ago but haven't been using them much until recently, when I decided to take advantage of the "free" tier. I'm using an EC2[^ec2] micro instance virtual private server right now as an online coding server pretty much specifically so I can invite other devs in on tmux[^tmux] for remote pairing, but that's not actually relevant to my blog platform. For that, I'm using Route53[^route53] and S3[^s3], but it's all giving me a good opportunity to come up to speed on a lot of their tech.

S3[^s3] provides file storage in the cloud. With the right permissions on files, you can share them with the world just like a web server would. Since my blogging platform is a "static site generator" (it builds a collection of static files that make up the site), I need someplace to store the files that's accessible via the internet.

Route53[^route53] is the special sauce that makes my static files accessible via my domain. This is a fairly recent addition to the AWS product line. First, it provides Domain Name Services or DNS for any domain pointed there. Second, it let's you associate a host in your DNS with a bucket in your S3. With that, any file I upload to S3 can be served at [http://version2beta.com](http://version2beta.com). Mission accomplished.

### Vagrant and Chef

Chef is a server management tool that let's you associate a server (node) with cookbooks and recipes that install, configure, and maintain the software on the server. More advanced use lets you associate your server with one or more roles (like application server, or database server, load balancer, or super-secret-international-spyperson-hacking-platform) in an environment (like development, QA, production, or maybe destructive-test-grounds).

Chef works well with Vagrant[^vagrant], which is a tool for quickly deploying local virtual machines using VirtualBox.[^vbox] Vagrant deploys the virtual machine and then uses Chef (or Puppet,[^puppet] a similar tool) to provision it.

What I've done this time around is create a Vagrant project (available on my Github[^vagrant-chef-flask]) that sets up my development environment on any machine with Vagrant installed. The blog is also on Github[^blog-github] of course, so in one command, I build everything I need in a development server to work on my blog. It's an on-demand environment. Pretty slick.

I've been killing birds with stones all over the place with Vagrant and Chef. One of my current $dayjob[^tartan] projects is to create a portable development environment that's quick and easy for a dev to deploy. Another is to create QA environments that are disposable and fast. A third is to set up our production systems on Opscode's Hosted Chef environment.[^hostedchef]

I'm also preparing a presentation for the Milwaukee PHP User's Group[^mkepug], one of our excellent local (Milwaukee) tech meetups. My presentation is on creating and using virtual development environments:[^smooth]

> Virtual development environments keep your workspace clean and predictable in dev and production, and creating a virtual development machine with Vagrant is quick and easy once you have all the pieces in place. Rob Martin (@version2beta) is going to put all the pieces together and hand them to you, and then show you how you can use them.

### Python Flask microframework

I've mentioned Flask[^flask] in other blog posts. It's a Python microframework originally inspired by Ruby's Sinatra[^sinatra] and written by Armin Ronacher[^armin]. I'm a big fan of Armin's work, which includes not only Flask but also Werkzeug,[^werkzeug] Jinja2,[^jinja2], and Sphinx.[^sphinx]

I find writing web applications and restful services in Flask pretty easy. But it wasn't until I was looking at my options for replacing Hyde on this blog that I discovered Frozen-Flask,[^fflask] a Flask extension that takes a static snapshot of a Flask application and stores it in a file system. Frozen-Flask makes Flask into a static site generator.

My Flask application is really quite simple.

* If I run it directly (`python sitebuilder.py`) it launches a development server on port 8000 (which Vagrant has forwarded to port 8000 on the host computer.)
* If I run `python sitebuilder.py build` it freezes the website into the ./build directory.
* If I run `python sitebuilder.py deploy` it freezes the website and copies the contents of the ./build directory to the S3 bucket, taking the site live.

### YAML and markdown

Finally it comes down to content, which if I understand correctly is an important part of a blog.

My content is stored in flat files. I like this. It makes it very easy to do version control and store them at Github. In fact with Github's new online editing,[^ghedit] I can even create and edit blog posts through their web interface.

My content is divided into "pages" and "articles". Pages are manually linked to the navigation, or referenced on other pages. Articles are blog posts and always sorted in reverse chronological order (most recent first).

Each page or article is in it's own text file. The raw file is YAML[^yaml] over Markdown.[^markdown] There's a YAML header that contains a short and long title (the short title is used for the H1 tag, the long title is used for the title attribute in links), and an excerpt or summary. Articles also have a published attribute with the date and time to show as the published timestamp. These YAML fields are arbitrary though - what they're called only has relevance if they're referenced in a template or the program logic, so any fields can be added at any time. In fact, I'll probably add tags at some point, which will require updating the controller and the templates.

After the YAML header, there's a blank line. After that, there's markdown defining the main content for the page or article. Easy-peasy.

## What's next

I hadn't intended to use YAML as my metadata transport. I wanted to use JSON for all of the content definitions, and support a variety of backends including CouchDB.[^couchdb] But JSON doesn't support multiline strings (you must embed the carriage returns) and I don't have a JSON+Markdown editor that I can connect to CouchDB. So that's a project I might take on.

Blogging is inherently organized by time, and my platform supports that. I think it'd be nice to also support tags. If I do that, I'll just use Javascript to make a faceted filter - check or uncheck tags or categories and articles appear or disappear.

# Check it out

Check out the code for my blog platform on Github at [https://github.com/Version2beta/version2beta](https://github.com/Version2beta/version2beta).

Check out the Vagrant + Chef setup at [https://github.com/Version2beta/vagrant-chef-flask](https://github.com/Version2beta/vagrant-chef-flask). I'll be building a couple more open source versions of that getup for PHP stuff in preparation for my February presentation.



[^reset]: I blogged about the last blog reset too in [Blog reset][reset]. I blogged about starting this blog in [Instantiate blog][instantiate] almost two years ago.

[reset]: /articles/blog-reset/ "The last blog reset I did"

[instantiate]: http://version2beta.com/articles/instantiate-blog/ "The story of how I ended up blogging. Again."

[^hyde]: [Hyde][hyde] is a static site generator inspired by Ruby's Jekyll[^jekyll].

[hyde]: http://ringce.com/hyde "Hyde is a static website generator powered by Python & Django."

[^django]: [Django][django] is a powerful and monolithic web application platform.

[django]: https://www.djangoproject.com/ "The web framework for perfectionists with deadlines."

[^jekyll]: [Jekyll][jekyll] is a Ruby-based static site generator. I don't consider myself a Ruby programmer, but I suspect that's changing.

[jekyll]: http://jekyllrb.com/ "Jekyll is a blog-aware, static site generator in Ruby."

[^sinatra]: [Sinatra][sinatra] is a brilliantly simple web application microframework in Ruby. It's one of the reasons I'll probably become a Ruby programmer.

[sinatra]: http://www.sinatrarb.com/ "Sinatra is a DSL for quickly creating web applications in Ruby with minimal effort."

[^tako]: [Tako][tako] is a functional web framework in Javascript for Node.js.

[tako]: https://github.com/mikeal/tako "Tako framework."

[^aws]: [AWS][aws] is Amazon Web Services, Amazon's platform for developing "web scale" applications in the cloud.

[aws]: http://aws.amazon.com "Amazon Web Services"

[^ec2]: [EC2, or Elastic Compute Cloud][ec2] is Amazon AWS' virtual private server offering.

[ec2]: http://aws.amazon.com/ec2/ "Elastic Compute Cloud"

[^tmux]: [TMUX][tmux] is a "terminal multiplexer" that I find useful when sharing a coding session with another programmer.

[tmux]: http://tmux.sourceforge.net/ "TMUX terminal multiplexer"

[^route53]: [Route53][route53] is Amazon AWS' domain name service.

[route53]: http://aws.amazon.com/route53/ "Amazon Route 53 is a highly available and scalable Domain Name System (DNS) web service."

[^s3]: [S3, or Simple Storage Service][s3] is Amazon AWS' cloud-based file store.

[s3]: http://aws.amazon.com/s3/ "Amazon internet file store"

[^vagrant]: [Vagrant][vagrant] is a tool for creating lightweight, reproducable, portable development and QA environments.

[vagrant]: http://www.vagrantup.com/ "Development environments made easy"

[^vbox]: [VirtualBox][vbox] makes virtual machines run on a variety of host platforms including at least Linux, MAC OS X, and Windows.

[vbox]: https://www.virtualbox.org/ "Virtualization platform"

[^puppet]: [Puppet][puppet] is a configuration management tool similar in some ways to Chef. I'm using Chef without much review of Puppet, so I can't add a whole lot more here.

[puppet]: http://puppetlabs.com/ "Configuration management."

[^vagrant-chef-flask]: This is my [code repository for my blog dev environment][vagrant-chef-flask] using Vagrant and Chef.

[vagrant-chef-flask]: https://github.com/Version2beta/vagrant-chef-flask "dev environment source code"

[^blog-github]: This is my [code repository for my blog and static site generator][blog-github].

[blog-github]: https://github.com/Version2beta/version2beta "blog source code"

[^tartan]: [Tartan Solutions][tartan] is my $dayjob.

[tartan]: http://tartansolutions.com "Tartan Solutions"

[^hostedchef]: Opscode Chef comes in a [online version][hostedchef] that's pretty slick. I'm using it.

[hostedchef]: http://www.opscode.com/hosted-chef/ "Hosted Chef at Opscode."

[^mkepug]: Milwaukee has a ton of great meetups. The [PHP User's Group][mkepug] is just one of them.

[mkepug]: http://www.mkepug.org/ "Milwaukee PHP User's Group"

[^smooth]: I'll be [presenting][smooth] on February 12, 2013 at 6 PM if you're in the area and available.

[smooth]: http://www.mkepug.org/events/101042652/ "Smooth moves with Vagrant, Chef-solo, and git"

[^flask]: Flask is a Python microframework, originally inspired by Sinatra. It's my framework of choice for most things.

[flask]: http://flask.pocoo.org/ "Flask is a microframework for Python based on Werkzeug, Jinja 2 and good intentions."

[^armin]: Check out [Armin online][armin]. He's pretty damn cool.

[armin]: http://lucumr.pocoo.org/ "Armin's blog."

[^werkzeug]: [Werkzeug][werkzeug] is a whiskey library. No, a WSGI brilary. No, A whiskey a-go-go. I forget, what were we talking about?

[werkzeug]: http://werkzeug.pocoo.org/ "Werkzeug WSGI library"

[^jinja2]: [Jinja2][jinja2] is a Python templating engine with a lot of features.

[jinja2]: http://jinja.pocoo.org/ "Jinja2 templating engine."

[^sphinx]: [Sphinx][sphinx] is a Python-based documentation platform used by lots and lots of projects, including

[http://docs.python.org/](http://docs.python.org/3/).

[sphinx]: http://sphinx-doc.org/ "Sphinx documentation engine."

[^fflask]: [Frozen-Flask][fflask] turns Flask into a static site generator.

[fflask]: http://packages.python.org/Frozen-Flask/ "Frozen-Flask freezes a Flask application into a set of static files"

[^ghedit]: [Github provides online editors now][ghedit]. With these editors you can change existing files or create new ones right from the website. In fact, you can even edit other people's repos online. It forks them for you, and on commit will helpfully send a pull request.

[ghedit]: https://github.com/blog/905-edit-like-an-ace "Github online editing."

[^yaml]: [YAML][yaml] is a way to store data in a way that computers find meaningful and people find comfortable.

[yaml]: http://en.wikipedia.org/wiki/YAML "YAML on Wikipedia"

[^markdown]: [Markdown][markdown] is a markup language, kinda like HTML but way easier to use and much nicer to read.

[markdown]: http://en.wikipedia.org/wiki/Markdown "Markdown on Wikipedia"

[^couchdb]: [CouchDB][couchdb] is a JSON-based NoSQL database.

[couchdb]: http://couchdb.apache.org/ "Relax."
