# 2010.09.30 Jasmine Selenium Meetup

!SLIDE middle small

![Jasmine Logo](images/jasmine_logo.png)

## SF Selenium Meetup, 2010.09.30

### [Davis W. Frank](mailto:dwfrank@pivotallabs.com) | [Pivotal Labs](http://pivotallabs.com) | [@dwfrank](http://twitter.com/dwfrank)

!SLIDE middle

# TATFT

!SLIDE middle

# What about JavaScript?

!SLIDE middle

# <em>UT</em>&nbsp;YJSATFT

!SLIDE middle

# UTYJSATFT

## with Jasmine

!SLIDE

@@@ js
describe("Math", function() {
  describe(".sqrt", function() {
    it("returns the square root of a number", function() {
      expect(Math.sqrt(64)).toEqual(8);
    });
  });
});
@@@
    
!SLIDE middle
       
![Example test run](./images/simple-example-pass.jpg) 

!SLIDE middle

# Expectations

!SLIDE

<pre>
	As a site administrator
    I want to obfuscate offensive language
    So that I don't risk offending my users
</pre>

!SLIDE

@@@ js
describe("String", function() {
  describe(".cleanUp", function() {
    it("replaces restricted words with four asterisks", function() {
      filterWords = ["golly", "heck"];
      dirtyString = "Golly! What the heck?";
      cleanString = dirtyString.cleanUp(filterWords);
      expect(cleanString).toEqual("****! What the ****?");
    });
  });    
});
@@@

!SLIDE middle

![Example test run](./images/string-failure.jpg) 

!SLIDE

@@@ js
String.prototype.cleanUp = function (filterWords) {
  var pattern = filterWords.join("|"),
  regExp  = new RegExp(pattern, "mig");
  return this.replace(regExp, "****");
};
@@@

!SLIDE middle

![Example test run](./images/string-pass.jpg) 

!SLIDE

# Matchers

@@@ js
expect(cleanString).toEqual("****! What the ****?"); 
expect(cleanString).toMatch(new RegExp("\\*{3}")); 
expect(cleanString).toBeDefined(); 
expect(cleanString).not.toBeNull();
expect(cleanString).toBeTruthy();
expect(cleanString).not.toBeFalsy();
expect(cleanString).not.toContain("golly");
@@@

!SLIDE

# Custom Matchers

@@@ js
// top-level beforeEach() call is made before every spec
beforeEach(function() {
  this.addMatchers({
    toBeCleanedUp: function() {
      return this.actual.cleanUp(filterWords) !== this.actual;
    }
  });
});
@@@

!SLIDE

@@@ js
describe(".cleanUp", function() {
  beforeEach(function() {
    filterWords = ["justin", "bieber"];  
  });
      
  it("replaces restricted words with three asterisks", function() {
    expect("Justin Bieber: My World").toBeCleanedUp();
  });

  it("returns strings without restricted language unmodified", function() {
    expect("The National: High Violet").not.toBeCleanedUp();
  });
});    
@@@

!SLIDE middle

![Example test run](./images/custom-matcher-pass.jpg) 

!SLIDE middle

# Spies

!SLIDE

@@@ js
describe(".getTrends", function() {
  it("requests current trends from Twitter's search API", function() {
    spyOn(jQuery, "getJSON")
        
    twitterApi = new TwitterApi();
    twitterApi.getTrends();
        
    expect(jQuery.getJSON).toHaveBeenCalledWith(
      "http://search.twitter.com/trends.json?callback=?", 
      jasmine.any(Function)
    );
  });
});
@@@

!SLIDE middle

![Example test run](./images/spy-not-called.jpg) 

!SLIDE

@@@ js
function TwitterApi() {
  this.trendsUrl = "http://search.twitter.com/trends.json?callback=?"
}

TwitterApi.prototype = {
  getTrends: function() {
    jQuery.getJSON(this.trendsUrl, function(data, textStatus) {});
  }
};
@@@

!SLIDE middle

![Example test run](./images/spy-called.jpg) 

!SLIDE

@@@ js
mockResponse = {
  as_of: "2010-05-18 19:21",
  trends: [
    {name: "Woodford Reserve", url: "http://fake.url/woodford-reserve"},
    {name: "Clojure", url: "http://fake.url/clojure"},
    {name: "iPad", url: "http://fake.url/ipad"},
    {name: "San Bruno Fire", url: "http://fake.url/san-bruno-fire"},
    {name: "Haruki Murakami", url: "http://fake.url/haruki-murakami"}                
  ]
};
@@@

!SLIDE

@@@ js
describe("TwitterApi", function() {
  it("responds with an array of current trends", function() {
    spyOn(jQuery, "getJSON")
       
    twitterApi = new TwitterApi();

    var trends = [];
    twitterApi.getTrends(function(response) { 
      trends = response; 
    });

    jQuery.getJSON.mostRecentCall.args[1](mockResponse);
    expect(trends.length).toEqual(5);
    expect(trends[1]['name']).toEqual('Clojure');
  });
});
@@@

!SLIDE middle

![Example test run](./images/spies-array-failure.jpg) 

!SLIDE

@@@ js
TwitterApi.prototype = {
  getTrends: function(callback) {
    jQuery.getJSON(this.trendsUrl, function(data, textStatus) {
      callback(data["trends"]);
    });
  }
};
@@@

!SLIDE middle

![Example test run](./images/spies-array-pass.jpg) 

!SLIDE

# Spy Matchers

@@@ js
spyOn(jQuery, "getJSON");

expect(jQuery.getJSON).toHaveBeenCalled();
expect(jQuery.getJSON).toHaveBeenCalledWith("http://example.com/service.json");
@@@

!SLIDE

# Spy Definitions

@@@ js
spyOn(Math, "sqrt").andCallThrough(); // Calls the original function
spyOn(Math, "sqrt").andReturn(42); // Return 42
spyOn(Math, "sqrt").andThrow("Exception"); // Throws an exception
spyOn(Math, "sqrt").andCallFake(function(arg) { 
  return arg * Math.random(); 
}); // Substitute with fake anonymous function
@@@

!SLIDE 

# Spy Properties

@@@ js
spyOn(jQuery, "getJSON");
  
jQuery.getJSON.callCount; // Number of calls to spy
jQuery.getJSON.mostRecentCall.args; // Arguments for most recent call to the spy
jQuery.getJSON.argsForCall[1]; // Arguments for a specific call to the spy
@@@

!SLIDE middle

#DOM Fixtures and jQuery

!SLIDE

@@@ js
describe("TwitterApi", function() {
  beforeEach(function() { 
    jQuery('<div id="trends" />').appendTo("#jasmine_content"); 
  });

  afterEach(function() { jQuery("#trends").remove(); });    
  
  describe(".updateTrends", function() {
    it("builds a link for each trend and attaches it to #trends", function() {
      twitterApi = new TwitterApi();
      twitterApi.updateTrends(mockResponse.trends);
      href = jQuery('#trends li a:contains(iPad)').attr('href');
      expect(href).toEqual('http://fake.url/ipad');
    });
  });
});
@@@

!SLIDE

@@@ js
TwitterApi.prototype = {
  updateTrends: function(trends) {
    jQuery.each(trends, function(i, trend) {
      li = jQuery("<li />");

      jQuery("<a />")
        .attr("href", trend.url)
        .attr("target", "_blank")
        .html(trend.name)
        .appendTo(li);

      li.appendTo("#trends");
    });
  }
}
@@@

!SLIDE middle

![Example test run](./images/trends-pass.jpg) 

!SLIDE middle

![Screenshot](./images/trends.jpg)

!SLIDE

# Jasmine Gem

@@@ sh
$ gem install jasmine
$ rails _2.3.5_ app
$ cd app
$ script/generate jasmine
   Jasmine has been installed with example specs.
$ rake jasmine
   your tests are here: http://localhost:8888/
@@@

!SLIDE

![Textmate](./images/textmate-after-generate.jpg) 
![Example test run](./images/jasmine-ruby-sample.jpg) 

!SLIDE

# No Rails Required

@@@ sh
$ mkdir app
$ jasmine init
@@@

![Textmate](./images/jasmine-init.jpg) 

!SLIDE bulleted

# Continuous Integration

@@@ sh
$ rake jasmine:ci
@@@

  * Uses Selenium to drive the browser and rspec to test expectations
  * Works out of the box with CruiseControl.rb, Hudson, Team City, etc.

!SLIDE bulleted

# Support
<p></p>
  * Homepage: [http://pivotal.github.com/jasmine](http://pivotal.github.com/jasmine)
  * Blog: [http://jasmine.pivotallabs.com/blog](http://jasmine.pivotallabs.com/blog	)
  * Twitter: [@jasminebdd](http://twitter.com/jasminebdd)
  * Mailing List: [http://groups.google.com/group/jasmine-js](http://groups.google.com/group/jasmine-js)
  * Issues: [http://github.com/pivotal/jasmine/issues](http://github.com/pivotal/jasmine/issues)
  * Backlog: [http://pivotaltracker.com/projects/](http://pivotaltracker.com/projects/)

<br/>

# Contribute
<p></p>
  * [http://github.com/pivotal/jasmine](http://github.com/pivotal/jasmine)
  * [http://github.com/pivotal/jasmine-gem](http://github.com/pivotal/jasmine-gem)
  * [http://groups.google.com/group/jasmine-js-dev](http://github.com/pivotal/jasmine-gem)

!SLIDE middle thanks

# Thanks

<p></p>

## expect(jasmine).toBeDefined();

<p></p>

### [@jasminedbdd](http://twitter.com/twitterbdd)