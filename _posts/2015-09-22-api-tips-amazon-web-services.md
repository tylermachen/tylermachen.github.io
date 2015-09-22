---
layout: post
title: "API Tips: Amazon Product Advertising"
tags:
 -
---

As part of our first "project mode" app that we are building here at <a href="http://flatironschool.com/?gclid=CP243O37icgCFQmPHwod008GAA">Flatiron School</a>, my team and I decided to focus on using <a href="http://tylermachen.github.io/2015/09/07/d3-data-driven-documents.html">D3</a> to visualize retail product data based on a search keyword. In order to do this, we needed to find some APIs that would return a large quantity of product data in a format that we could play with. This blog is going to focus on the process I went through to make sense of the Amazon Product Advertising API (part of Amazon Web Services).
<!--more-->

## Zero to 100: Getting Started

Before starting this project I had only been guided through the use of APIs during our classwork and had some experience with the Reddit API through another class project called <a href="http://cloudr.herokuapp.com/">Cloudr</a>. Other than that, I was pretty fresh. When I sat down the first night to start implementing the Amazon API into our application, my desires were basic. ALL I WANTED, was to get a list of products and their reviews based on a search term. Pretty simple right? Well...no. It turns out that the Amazon API is pretty dense and has some hoops it makes you jump through before it opens the gates.

## Step #1: Getting Oriented

When starting to implement an API I have not worked with yet, I first decide what information I need, what format I need it in, and then I head to the source to see if I can make sense of things and decide if this particular API is a viable option for the task. In this case, I came across the Amazon Web Services (AWS) <a href="https://aws.amazon.com/documentation/">documentation</a>. This led me to another dead end due to information overload.

<img src="https://dgosxlrnzhofi.cloudfront.net/custom_page_images/307/page_images/AWS-account.png?1424293283">

Amazon Web Services is HUGE! They offer all kinds of cloud web services from analytics to content delivery and the list keeps getting longer. I started sifting through the AWS docs and was not finding any information related to my simple need for products and reviews.

## Step #2: Google It!

Since I couldn't make sense of things on my own from the firsthand API docs, the next step was to find someone else that had experience with the API. Google usually has the answer, or could at least point me in the right direction. If I could find some StackOverflow, blog, or even video posts on how to integrate Amazon's data into my application, then I would be moving in the right direction. I Googled something along the lines of ("ruby amazon api product reviews") and discovered that I needed to look into Amazon's Product Advertising API.

## Step #3: Get Authorized

Not all APIs require authorization, but a lot of the best ones do. Some APIs allow you to access some data without being authorized but will give you more once you are. So again, it's important to get oriented with the API you are going to be working with so that you are aware of it's capabilities and limitations, especially, if you are relying on it for the main functionality of your app.

Amazon requires two types of authorizations to use their API.

* First, you need to sign up for an AWS API account <a href="https://aws.amazon.com/free/?sc_channel=PS&sc_campaign=acquisition_US&sc_publisher=google&sc_medium=cloud_computing_b&sc_content=aws_core_bmm&sc_detail=%2Bamazon%20%2Bweb%20%2Bservices&sc_category=cloud_computing&sc_segment=73823472762&sc_matchtype=b&sc_country=US&s_kwcid=AL!4422!3!73823472762!b!!g!!%2Bamazon%20%2Bweb%20%2Bservices&ef_id=Vf9@EAAAAAyMIRtJ:20150922071426:s">here</a> This will give you free access to Amazon's free tier of cloud computing services for 12 months and, more importantly, allow you to obtain your API access keys.

* Second, sign up to be an Amazon affiliate <a href="https://affiliate-program.amazon.com/">here</a>. This will give you your tracking (affiliate) id which is required to make API requests when searching for products. Also, this will ensure you get commissions if you end up using Amazon's API to sell products on your own site.

Once you have your keys, make sure to include them in your application <strong>BUT!</strong> make sure you <strong>DO NOT!</strong> upload them to any version control network such as GitHub because there are bots crawling all over the place that will steal your API access keys and potentially do some malicious things that will be tied to your name. Look into the <a href="https://github.com/bkeepers/dotenv">dotenv gem</a> or <a href="https://github.com/laserlemon/figaro">figaro gem</a> for information on how to safely implement API keys into your application without uploading them to the public domain.

## Step #4: API Call & Gems

So...even after getting my API keys all ready to go, I still did not know how to get the data I needed from Amazon. After searching around the docs I found <a href="http://docs.aws.amazon.com/AWSECommerceService/latest/DG/CHAP_OperationListAlphabetical.html">this page</a> which shows the valid operations for building a request to the API. In particular, the <a href="http://docs.aws.amazon.com/AWSECommerceService/latest/DG/ItemSearch.html">ItemSearch</a> page was what we ended up using to get our product listings.

These pages guide you through manually setting up requests and show you what you will receive in response from the API. If the below snippets look confusing, read on to find out about using an API wrapper instead.

### Sample Request

{% highlight ruby %}
http://webservices.amazon.com/onca/xml?
Service=AWSECommerceService&
AWSAccessKeyId=[AWS Access Key ID]&
AssociateTag=[Associate ID]&  
Operation=ItemSearch&
Keywords=Mustang&
SearchIndex=Blended
&Timestamp=[YYYY-MM-DDThh:mm:ssZ]
&Signature=[Request Signature]
{% endhighlight %}

### Sample XML Response

{% highlight xml %}
<TotalResults>372</TotalResults>
<TotalPages>38</TotalPages>
<Item>
  <ASIN>B00021HBN6</ASIN>
  <ItemAttributes>
    <Manufacturer>Radio Flyer</Manufacturer>
    <ProductGroup>Toy</ProductGroup>
    <Title>Radio Flyer Retro Rocket</Title>
  </ItemAttributes>
  </Item>
  <Item>
  <ASIN>B0007MZV3C</ASIN>
  <ItemAttributes>
  <Manufacturer>Razor USA LLC</Manufacturer>
  <ProductGroup>Toy</ProductGroup>
  <Title>Razor Dirt Rocket MX350 Bike</Title>
  </ItemAttributes>
</Item>
{% endhighlight %}

### Using the Sucker Gem

Instead of building the requests in the way outlined by Amazon's API documentation, I Googled around and looked into a few different candidates for gems that could make the process easier. Eventually, I came across the <a href="https://github.com/ryanmarshall/sucker">sucker gem</a> that is a minimal Ruby wrapper to the Amazon Product Advertising API.

Nice! That's exactly what I was looking for. I then went on to build a ProductParser module that utilized the gem to make API requests and called its methods elsewhere in the application which returned a hash that was converted from the original XML response.

{% highlight ruby %}
module ProductParser
  def self.build_product_request(keyword)
    # initialize sucker gem
    request = Sucker.new(
      :key    => ENV['AWS_ACCESS_KEY_ID'],
      :secret => ENV['AWS_SECRET_ACCESS_KEY'],
      :associate_tag => 'tag',
      :locale => :us
    )
    # build request
    request << {
      :operation     => 'ItemSearch',
      :keywords      => keyword,
      :search_index  => 'All',
      :ResponseGroup => 'Large',
      :TruncateReviewsAt => 0
    }
    request
  end

  def self.get_product_response(request)
    response = request.get
    response.to_hash
  end
end
{% endhighlight %}

## Step #5: Fame and Fortune

Just kidding...no fame and fortune here, but hopefully you found this post helpful.

#### Some Extra Advice:

* Make sure you abide by the rules. APIs provide guidelines on the appropriate use of their data and how many requests can be made per minute and so on.

* APIs change. What works today may be broken tomorrow. Stay on top of any new developments with your API especially if it is super important for the main functionality of your application. For example, Amazon used to provide raw product reviews as part of their API response, but now only offer review iFrameURLs. This means that they have tried to block manipulation of their review data.

* Not all APIs give you access right away. Furthermore, this could be after you made it through all the obstacles to get signed up. Our team ran into this issue with eBay's shopping.com API. After submitting all of our information, we received a message (in typical eBay fashion) that said our request would be reviewed within 3-5 business days...sweet. We probably won't be utilizing their data since our project is due in 4 days.

* Lastly, some APIs are too cool for school and require that you pay them a hefty fee to access their data. No free lunch.

---

#### Sources

* <a href="https://aws.amazon.com/">Amazon Web Services Homepage</a>

* <a href="http://docs.aws.amazon.com/AWSECommerceService/latest/DG/ProgrammingGuide.html">Product Advertising API Docuementation</a>
