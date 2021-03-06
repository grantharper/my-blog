## My Blog Source Control
[Deployed version][blog-website]

## About
This blog uses [jekyll][jekyll-install] to generate the static files to load to the deployed site

## Notes
The [jekyll swiss][jekyll-swiss] template default files can be found at the following location on Windows
`C:\RailsInstaller\Ruby2.3.0\lib\ruby\gems\2.3.0\gems\jekyll-swiss-0.4.0`

## Overall Setup Instructions

* Install [Ruby][ruby-install]
* Install [Jekyll][jekyll-install]: `gem install jekyll bundler`
* Create blog directory: `jekyll new my-blog`
* Move into directory: `cd my-blog`
* Modify your Gemfile to add the jekyll swiss theme: `gem "jekyll-swiss"`
* Install the theme: `bundle install`
* Add the theme to `_config.yml` to activate it: `theme: jekyll-swiss`
* Serve the site: `jekyll serve`

## Deployment
* Build the `_site` directory: `jekyll build`
* Deploy the site to S3: `aws s3 sync _site s3://blog.grantharper.org` 
* View the live site: [blog.grantharper.org][blog-website]

## AWS Cloudfront 
* The site is cached with AWS Cloudfront in order to use a security certificate on top of S3. If the same file is uploaded to S3, you may need to invalidate the cache in the Cloudfront console in order to update the file stored at the edge location. Invalidating the root of the site is often the easiest way to do this.

[blog-website]: https://blog.grantharper.org
[ruby-install]: https://www.ruby-lang.org/en/documentation/installation/
[jekyll-install]: https://jekyllrb.com/
[jekyll-swiss]: https://github.com/broccolini/swiss