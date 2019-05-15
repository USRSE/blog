# U.S. Research Software Engineer Blog

This is a syndicated blog to provide content for research software engineers
across the United States. Here they can add an rss/xml feed to share stories
and experiences.

## How does it work?

### 1. Add Metadata

An RSE or related group that has a blog, podcast, or similar feed can add their
metadata to the [authors.yml](_data/authors.yml) file. Here is an example
of what we collect:

```yaml
- name: "US Research Software Engineer Community"
  tag: "usrse"
  url: http://us-rse.org/
  feed: http://us-rse.org/feed.xml
```

The tag must be unique (and this is tested), and the feed should be a format
parseable by [feedparser](https://pythonhosted.org/feedparser/) (most are).

#### Wordpress

WordPress is a common blogging platform, and so we include notes here for how
to find a feed for your wordpress blog. If you want to include all content,
you can usually find a main feed at `https://<yourblog>/feed/`. However, it's
recommended to create a [tag or category](https://wordpress.org/support/article/wordpress-feeds/#categories-and-tags) 
feed, in which case you could find the feed at `https://<yourblog>/category/<category>/feed/`. See the linked
page for more ways that you can generate custom feeds based on tags and categories.
Once you've added your feed, it's recommended to test generate posts to ensure
that it's parsed correctly. This is done during the continuous integration,
but you can also do it locally (see below).

### 2. Generate Posts

The posts are generated automatically - we do this by way of a cron job (scheduled
job) on CircleCI. During the run, each feed is read, and any new posts are generated
as markdown files, with the author tag corresponding to the folder name in [_posts](_posts).
If the post is already included, it is skipped over. This is a reasonable task to do,
because typically feeds only provide the 10 (or a small number) of latest posts.

If you look at the [.circleci/config.yml](.circleci/config.yml) you'll notice that
this works by way of an environment variable, `CIRCLECI_TRIGGER`. This is added as
a build context to the workflow that runs nightly, and indicates that a build
and deploy is desired. Another important note is that the environment during the 
trigger is not the same as when triggered by a user (via pull request). For example,
`CIRCLE_USERNAME`, which is normally set, is undefined for the nightly trigger.
For this reason, we also define this in the context.

The reason this is set up to run with continuous integration is so that the site
is regularly updated without human intervention. In the case that there is an error,
the maintainers are notified. However, you can test this generation locally, without
generating any files!

```bash
$ python script/generate_posts.py _data/authors.yml --output _posts/ --test
```

It will show you any new folders and files generated without actually doing it.

```bash
$ python script/generate_posts.py _data/authors.yml --output _posts/ --test
[TEST] new author folder /home/vanessa/Documents/Dropbox/Code/usrse/blog/_posts/dsk
Preparing new post: /home/vanessa/Documents/Dropbox/Code/usrse/blog/_posts/dsk/2019-4-22-p=1452.md
Preparing new post: /home/vanessa/Documents/Dropbox/Code/usrse/blog/_posts/dsk/2019-2-5-p=1446.md
Preparing new post: /home/vanessa/Documents/Dropbox/Code/usrse/blog/_posts/dsk/2019-1-23-p=1444.md
Preparing new post: /home/vanessa/Documents/Dropbox/Code/usrse/blog/_posts/dsk/2018-9-26-p=1442.md
Preparing new post: /home/vanessa/Documents/Dropbox/Code/usrse/blog/_posts/dsk/2018-6-27-p=1423.md
Preparing new post: /home/vanessa/Documents/Dropbox/Code/usrse/blog/_posts/dsk/2018-6-25-p=1421.md
Preparing new post: /home/vanessa/Documents/Dropbox/Code/usrse/blog/_posts/dsk/2018-2-8-p=1415.md
```

## Development

To develop the site, clone the repository and then build with jekyll:

```bash
bundle exec jekyll serve
```

You can also run the script to generate posts locally, if you choose.

```bash
cd script

python generate_posts.py 
usage: generate_posts.py [-o OUTPUT] authors

Authors Parser

positional arguments:
  authors               the authors.yml file.

optional arguments:
  -o OUTPUT, --output OUTPUT
                        The output folder to write posts.
```

This is how the posts are generated in the continuous integration setup:

```bash
python generate_posts.py ../_data/authors.yml --output ../_posts/
```

### Important Notes

The "posts" page under the [posts](posts) folder is intentionally separate from
the other pages in [pages](pages) because the pagination plugin needs to find an
index.html to parse. Please don't rename or change the location of this file.

## Todo:

 - add email to _config.yml to have emails sent via formspree
 - add scheduled jobs to circleci

## Credit

The original template was from [bootstrap-clean-jekyll](https://github.com/BlackrockDigital/startbootstrap-clean-blog-jekyll) 
and has been simplified and updated for this case.
