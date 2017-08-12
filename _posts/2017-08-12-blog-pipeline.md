---
layout: post
title:  "Blog Pipeline"
date:   2017-08-12
categories: software
---

If you say blogging technology, I say Wordpress. While Wordpress is a great platform, I have chosen to explore the world of the static Content Management System (CMS) to build this particular blog. 

In the post I will discuss the static CMS blog architecture and its benefits. I will also show you how to set up a blog of your own and what it takes to go from writing a post to hosting it online.

## Architecture

In general, the blog uses [Jekyll][jekyll-install] to compile markdown files into static HTML, CSS, and Javascript files that can be directly loaded to a publicly accessible web directory. The web directory in this case is an [S3 bucket][aws-s3] with web hosting enabled that has a bucket name **identical** to the domain where the blog is hosted (e.g. blog.grantharper.org). Routing from the domain to the bucket is done via [Route 53][route-53].

<img src="/img/blog-architecture.png" height="500px">

## Cheap

Using Amazon S3 for a low-traffic blog is highly cost-effective.

S3 and Route 53 Pricing per Month

Storage | $0.023 per GB
GET Requests | $0.004 per 10,000 requests
PUT, COPY, POST, LIST Requests | $0.005 per 1,000 requests
Data Transfer to Internet | First GB free, Up to 10TB $0.090 per GB
Hosted Zone | $0.50

Domain names typically cost $12-14 per year for most any typical domain provider. Using Amazon Route 53 to register your domain is particularly convenient.

Typical hosting costs for a server that runs a wordpress site are about $5 per month at the low end.

The following calculator estimates the cost on a per month basis of using Amazon S3 for hosting static files.

### Cost Calculator

<style type="text/css">
	#totalCost {
		color: #E74727;
		font-weight: bold;
	}

</style>
Metric | User Input | This Blog as of 8/9/2017 (estimate)
Storage in GB | <input id="storage" class="interactive" type="text" name="storage"/> | 0.000250
Number of Requests | <input id="numRequests" class="interactive" type="text" name="numRequests"/> | 200 (optimistic)
Number of Files Uploaded | <input id="numUploads" class="interactive" type="text" name="numUploads"/> | 1000
Data Transfer to Internet | <input id="dataTransfer" class="form-control interactive" type="text" name="dataTransfer"/> | 0.500
**Total Cost per Month** | **$** <span id="totalCost">0</span> | 0.005


<script type="text/javascript">
	var list = document.getElementsByClassName("interactive");
	for (var i = list.length - 1; i >= 0; i--) {
		list[i].addEventListener("keyup", calculate);
	}

	function calculate(){
		var storage, numRequests, numUploads, dataTransfer, 
			storageCost, numRequestsCost, numUploadsCost, dataTransferCost,
			totalCost;

		//pull in field values
		storage = document.getElementById("storage").value;
		numRequests = document.getElementById("numRequests").value;
		console.log("numRequests=" + numRequests);
		numUploads = document.getElementById("numUploads").value;
		dataTransfer = document.getElementById("dataTransfer").value;

		//calculate each value's cost
		storageCost = storage * 0.023
		numRequestsCost = numRequests * 0.004 / 10000;
		numUploadsCost = numUploads * 0.005 / 1000;
		if(dataTransfer < 1){
			dataTransferCost = 0;
		} else{
			dataTransferCost = dataTransfer * 0.09;
		}
		
		totalCost = storageCost + numRequestsCost + numUploadsCost + dataTransferCost;
		totalCost = Math.round(totalCost * 100) / 100;

		document.getElementById("totalCost").innerHTML = totalCost;

	}
</script>

## Secure

Static HTML, CSS, and Javascript. That's all. There is no operating system or database to hack into. There are no patches to install or updates to perform.

Wordpress, on the other hand, is a favorite PHP playground for hackers, especially for sites that fall behind on updates.

## Fast

AWS S3 page load times are in the milliseconds. 

In contrast, some of the cheaper web hosting options require multiple seconds to load a wordpress page.

## Fully customizable

Anything that HTML, CSS, and Javascript can do, the static CMS Jekyll can do to. While Jekyll does not have the extensive library of themes and plugins present in the Wordpress community, it is a great option for those with an understanding of basic web technologies to build and deploy a blog. 

You can start with a pre-built theme and override any file you want by replicating it in your blog's directories that match the theme's directories.

## Setup

* Install [Ruby][ruby-install]
* Install [Jekyll][jekyll-install]: `gem install jekyll bundler`
* Create blog directory: `jekyll new my-blog`
* Move into directory: `cd my-blog`
* Modify your Gemfile to add the jekyll swiss theme: `gem "jekyll-swiss"`
* Install the theme: `bundle install`
* Add the theme to `_config.yml` to activate it: `theme: jekyll-swiss`
* Serve the site: `jekyll serve`

## Deployment Pipeline

Below is a simple process flow that illustrates how to go from authoring a post, to deploying it live on the internet.

<img src="/img/blog-pipeline-2.png" height="500px">


### Specific steps and commands

* Create a file in the `_posts` directory with the format `yyyy-mm-dd-blog-title.md`
* Open a terminal in the root directory and run the command `jekyll serve`
* Preview the new post at `localhost:4000`
* Deploy the new post using the command `aws s3 sync _site s3://blog.grantharper.org`
* View the new post on [blog.grantharper.org][blog-website]

[blog-website]: http://blog.grantharper.org
[ruby-install]: https://www.ruby-lang.org/en/documentation/installation/
[jekyll-install]: https://jekyllrb.com/
[route-53]: https://aws.amazon.com/route53/
[aws-s3]: https://aws.amazon.com/s3/