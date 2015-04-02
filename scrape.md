# Nokogiri Notes
### Scraping a website with Nokogiri and XPath

* If you're in a rush [skip to example code](/scrape.md#back-to-scraping-/)


To setup your environment to use Nokogiri you need to require both the Nokogiri gem and the OpenURI module from ruby.
Go ahead and start a new pry or irb session in your terminal.
If you already have Nokogiri installed type:

``` ruby
require 'nokogiri'
```
If you don't have Nokogiri yet type:

``` ruby
gem install 'nokogiri'
require 'nokogiri'
```
Now require the OpenURI module by typing:

``` ruby
require 'open-uri'
```
Once both of those are set, we can now use the gem and module
to grab a webpage that we pass it.  You will need the url of a website to get the information off of it.
``` ruby
Nokogiri::HTML(open("https://weworkremotely.com/jobs/search?term=ruby"))
```

That is the url of a website for remote job postings.
While we are at this step, set that document to a variable so you only need to make the url call one time
``` ruby
document = Nokogiri::HTML(open("https://weworkremotely.com/jobs/search?term=ruby"))
```

Now you have a document with a bunch of Nokogiri elements in it.  Check the class of the document
``` ruby
document.class
```
it should return a Nokogiri::HTML::Document

So we have a Nokogiri HTML document, next step is to navigate through the document to find the specific information we need.
To do this we will use a language calld XPath, a language designed to navigate XML documents.  Now, you might be thinking
we don't have any XML document, only HTML.  Well you are right, we'll fix that with the next line of code.

Take that document you saved earlier and pass an XPath onto it.
``` ruby
document.xpath("//div")
```

Now check its class
``` ruby
document.xpath("//div").class
```
That should return a Nokogiri::XML::Document.  Sweet, you have some XML.
The ``` xpath("//div") ``` is the start to how you will be dropping down into the DOM to whatever place has the information you want.

---
<h4>Some notes on XPath:</h4>

1. it navigates XML documents.
2.  every HTML element name you add to the xpath string needs to be separated by ``` // ```.

3. each element can be reached by simply calling the name of it.

    ```
    document.xpath("//div//a//li")
    ```
      * will return all the ``` li ``` elements inside all the ``` a ``` elements

4. when looking for specific elements with an id or class, use contains or position.

    ```
    document.xpath("//div//a//li[contains(@class, "someClassName')]")
    ```
      * will return the ``` li ``` with ``` class='someClassName' ```

    ```
    document.xpath("//div//a//li[contains(@id, "someIDName')]")
    ```
      * will return the ``` li ``` with ``` id='someIDName' ```

    ```
    document.xpath("//div//a//li[position() <= 2]
    ```
      * will return the 1st and 2nd ``` li ``` inside the every ``` a ``` element

---

##### Back to scraping ~

I found starting from the highest point of the DOM and working down into the nested parts works best.  This might mean you will
end up with an enormous xpath string, but its cool... you can always clean up the code later if you're feel crazy.  The more
specific you are the better chance of getting the correct information, until the site you are scraping changes.


So, let's scrape some HTML (continuing from the top section)
I took a website, weworkremotely.com, and ran a search for "ruby".  This returned the url ``` https://weworkremotely.com/jobs/search?term=ruby ```.  We'll be using this as our main document where all our information lies.  Looking at the webpage I can see that the info I want, job titles and links, are all located inside a table in the middle of the page.

Try going to that url and opening up your inspector.  Walk down the DOM with your mouse, try to open up the different elements and see what gets highlighted on the screen.  Keep poking around until that highlighted section is only over the table containing all the jobs.

In this case its once your mouse is hovering over the ``` article ``` element.  Go a little further into the ``` article ```, the highlighting should keep getting more narrow, untill you are inside of a single row in the table, then a single element. Those rows contain the information I want for each job. Lets go get them

    #check if you still have the document saved properly from earlier.
``` ruby
document = Nokogiri::HTML(open("https://weworkremotely.com/jobs/search?term=ruby"))
```

    #use xpath to navigate to to rows of data we want, this will get the whole table of data.
``` ruby
rows = document.xpath("//div[contains(@class, 'container')]//div[contains(@class, 'content')]
//section[contains(@class,'jobs')]//article//ul//li")
```
    #This is the full xpath to get all the rows inside the table with jobs.
    #Those rows are actually all of the "li" elements inside of the job table
    #This will return an Nokogiri::XML::NodeSet, basically and array of nodesets
    #It has a bunch of information you don't care about,
    #so let's narrow it down to only the title and links


    #This next method will return a array of hashes with the title and link - take a look
    #and play around with it, I'll explain in more detail below

``` ruby
rows.collect do |row|
  {
    :title => row.xpath("a//span[contains(@class, 'title')]").text,
    :link  => "https://weworkremotely.com#{row.xpath("a").attribute('href').value}"
  }
end
```

* Try putting a ``` binding.pry ``` under the ``` rows.collect ```, what is ``` row.class ```?
* The row is still a ``` Nokogiri::XML::Document ```, which means we can still call xpath on it - Oh the joy.
* The starting syntax changes a bit if you looked closely at the xpath on the doc and on the row notice
  - There are no ``` // ``` before the first element's name on the xpath for the row
  - Other than that, it is the same idea of walking down the DOM to find elements with the information you want.
* The methods that are called after the last ``` ) ``` in the xpath are Nokogiri methods.
* There are a bunch of them; ``` .text ```, ``` .attribute ``` and ``` .value ``` are what I have found I use the most.
* They pretty much explain themselves
  - ``` .text ``` returns the string of text inside of the ``` Nokogiri::XML::Nodeset ```
  - ``` .attribute() ``` will return the ``` Nokogiri::XML::Nodeset ``` of the attribute
  - ``` .value ``` will return the value of the attribute.

* That's it, the collection returned from the last function will be an array of hashes containing the title and link of the job posting on weworkremotely.com
* As always, please use Nokogiri responsibly
