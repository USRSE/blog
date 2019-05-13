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

### 2. Generate Posts

The posts are generated automatically - we do this by way of a cron job (scheduled
job) on CircleCI. During the run, each feed is read, and any new posts are generated
as markdown files, with the author tag corresponding to the folder name in [_posts](_posts).
If the post is already included, it is skipped over. This is a reasonable task to do,
because typically feeds only provide the 10 (or a small number) of latest posts.

The reason this is set up to run with continuous integration is so that the site
is regularly updated without human intervention. In the case that there is an error,
the maintainers are notified.

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
