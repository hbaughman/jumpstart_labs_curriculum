---
layout: page
title: EventManager
sidebar: true
alias: [ /eventmanager, /event_manager.html ]
language: ruby
topics: files, CSV, String, Sunlight API, ERB
---

## Background

### Motivator

A friend of yours runs a non-profit org around political activism. A number of
people have registered for an upcoming event. She has asked for your help in
engaging these future attendees.

### Concepts

In this project you will load content from a CSV (Comma Separated Value) file,
transform it into a standardized format, utilize the data to contact a remote
service, and finally populate a template with user data.

The techniques practiced:

* [File](http://rubydoc.info/stdlib/core/File) input and output
* Reading data from [CSV](http://rubydoc.info/stdlib/csv/file/README.rdoc) files
* [String](http://rubydoc.info/stdlib/core/String) manipulation
* Accessing [Sunlight](http://sunlightlabs.github.io/congress/index.html#parameters/api-key)'s Congressional API through
  the [Sunlight Congress gem][repo_sunlight_congress]
* Using [ERB](http://rubydoc.info/stdlib/erb/ERB) for templating

### Requirements

* [Ruby Environment]({% page_url /topics/environment/environment %})
* [Sample Data in CSV Format](event_attendees.csv)
* [Sunlight-Congress Gem](https://rubygems.org/gems/sunlight-congress)

<div class="note">
<p>This tutorial is open source. If you notice errors, typos, or have questions/suggestions,
  please <a href="https://github.com/JumpstartLab/curriculum/blob/master/source/projects/eventmanager.markdown">submit them to the project on GitHub</a>.</p>
</div>

### Initial Setup

Create a folder named `event_manager` wherever you want to store your project.
In that folder, use your text editor to create a plain text file named
`event_manager.rb`.

{% terminal %}
$ mkdir event_manager
$ cd event_manager
$ mkdir lib
$ touch lib/event_manager.rb
{% endterminal %}

Creating and placing your ruby file in 'lib' directory is entirely optional.
Placing the 'event_manager.rb' file in the 'lib' directory adheres to a common
convention within most ruby applications.

Ruby source files are often times written all in lower-case characters and
instead of camel-casing multiple words together they are instead separated by
an underscore (often called *snake-case*).

Open `lib/event_manager.rb` in your text editor and add the line:

```ruby lib/event_manager.rb
puts "EventManager Initialized!"
```
Validate that ruby is installed correctly and you have created the file correctly:

{% terminal %}
$ ruby lib/event_manager.rb
Event Manager Initialized!
{% endterminal %}

If ruby is not installed and available on your environment path then you will be presented with the following message:

{% terminal %}
$ ruby lib/event_manager.rb
-bash: ruby: command not found
{% endterminal %}

If the file was not created then you will be presented with the following error
message

{% terminal %}
$ ruby lib/event_manager.rb
ruby: No such file or directory -- lib/event_manager.rb (LoadError)
{% endterminal %}

For this project we are going to use the following sample data:

* [Small Sample](event_attendees.csv)
* [Large Sample](full_event_attendees.csv)

Download the *[small sample](event_attendees.csv)* csv file and save it in the
root of `event_manager` directory.

{% terminal %}
$ curl -o event_attendees.csv http://tutorials.jumpstartlab.com/projects/event_attendees.csv
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2125  100  2125    0     0   3269      0 --:--:-- --:--:-- --:--:-- 12073
{% endterminal %}


## Iteration 0: Loading a File

A comma-separated values
[(CSV)](http://en.wikipedia.org/wiki/Comma-separated_values) file stores
tabular data (numbers and text) in plain-text form. The CSV format is readable
by a large number of applications (e.g. Excel, Numbers, Calc). Its portability
makes it a popular option when sharing large sets of tabular data from a
database or spreadsheet applications.

### Read the File Contents

[File](http://rubydoc.info/stdlib/core/File) is a core ruby class that allows
you to perform a large number of operations on files on your filesystem. The
most straightforward being `File.read`

```ruby lib/event_manager.rb
puts "EventManager initialized."

contents = File.read "event_attendees.csv"
puts contents
```

Whether you use Single Quotes or Double Quotes does not matter. They are
different in many ways but are essentially the same when representing a string
of characters in this case as the initial greeting or the name of the file.

We are assuming the file is present here. File has the ability to check if a
file exists at the specified filepath on the filesystem through `File.exist?
"event_attendees.csv"`.


### Read the File Line By Line

Reading and displaying the entire contents of the file showed us how to quickly
access the data. Our goal is to display the first names of all the attendees.
There are numerous [String](http://rubydoc.info/stdlib/core/String) methods
that would allow us to manipulate this large string.

Files can also be read in as an array of lines.

```ruby lib/event_manager.rb
puts "EventManager initialized."

lines = File.readlines "event_attendees.csv"
lines.each do |line|
  puts line
end
```

First we read in the entire contents of the file as an array of lines. Second
we iterate over the entire collection of lines, one at a time, and output the
contents of each line.

### Display the First Name of All Attendees

Instead of outputing the entire contents of each line we want to show only the
first name. That requires us to look at the current contents of our Event
Attendees file.

```
 ,RegDate,first_Name,last_Name,Email_Address,HomePhone,Street,City,State,Zipcode
1,11/12/08 10:47,Allison,Nguyen,arannon@jumpstartlab.com,6154385000,3155 19th St NW,Washington,DC,20010
```

The first row contains header information. This row provides descriptional text
for each column of data. It tells us the data columns are laid out as follows
from left-to-right:

* `ID` - the empty column represents a unique identifier or row number of all
  the subsequent rows
* `RegDate` - the date the user registered for the event
* `first_Name` - their first name
* `last_Name` - their last name
* `Email_Address` - their email address
* `HomePhone` - their home phone number
* `Street` - their street address
* `City` - their city
* `State` - their state
* `Zipcode` - their zipcode

The lack of consistent formatting of these headers models is not ideal when
choosing to model your own data. These column names have been our extreme
example of a poorly formed external service. Great applications are often built
on the backs of such services.

We are intersted in the 'first_Name' column. At the moment we have a string of
text that represents the entire row. We need to convert the string into an
array of columns. The separation of the columns can be identified by the comma
',' separator. We want to split the string into pieces wherever we see a comma.

Ruby's
[String#split](http://rubydoc.info/stdlib/core/String#split-instance_method)
allows you to convert a string of text into an Array along a particular
character.

By default when you send the split message to the String without a parameter it
will break the string apart along a space " " character.

```ruby lib/event_manager.rb
puts "EventManager initialized."

lines = File.readlines "event_attendees.csv"
lines.each do |line|
  columns = line.split(",")
  puts columns
end
```

Within our array of columns we want to access our 'first_Name'. This would be
the third column or column at the array's second element `columns[2]`.

Arrays start counting at 0 instead of 1. To get the idea we would access the
array's zeroth element `columns[0]`.


```ruby lib/event_manager.rb
puts "EventManager initialized."

lines = File.readlines "event_attendees.csv"
lines.each do |line|
  columns = line.split(",")
  name = columns[2]
  puts name
end
```

### Skipping the Header Row

The header row was a great help to us in understanding the contents of the CSV
file. However, the row itself does not represent an actual attendee. To ensure
that we only output attendees we could remove the header row from the file, but
that would make it difficult if we later returned to the file and tried to
understand the columns of data.

Another option is to ignore the first row when we display the names. Currently
we handle all the rows exactly the same which makes it difficult to understand
which one is the header row.

One way to solve this problem would be to skip the line when it exactly matches
our current header row.

```ruby lib/event_manager.rb
puts "EventManager initialized."

lines = File.readlines "event_attendees.csv"
lines.each do |line|
  next if line == " ,RegDate,first_Name,last_Name,Email_Address,HomePhone,Street,City,State,Zipcode\n"
  columns = line.split(",")
  name = columns[2]
  puts name
end
```

A problem with this solution is that the content of our header row could change
in the future. Additional columns could be added or the existing columns
updated.

A second way to solve this problem is for us to track the index of the current
line.

```ruby lib/event_manager.rb
puts "EventManager initialized."

lines = File.readlines "event_attendees.csv"
row_index = 0
lines.each do |line|
  row_index = row_index + 1
  next if row_index == 1
  columns = line.split(",")
  name = columns[2]
  puts name
end
```

This is a such a common operation that Array defines
[Array#each_with_index](http://rubydoc.info/stdlib/core/Enumerable#each_with_index-instance_method).

```ruby lib/event_manager.rb
puts "EventManager initialized."

lines = File.readlines "event_attendees.csv"
lines.each_with_index do |line,index|
  next if index == 0
  columns = line.split(",")
  name = columns[2]
  puts name
end
```

This solves the problem if the header row were to change in the future. It does
now assume that the header row is first row within the file.


### Look for a Solution before Building a Solution

Either of these solutions would be a *OK* given our current attendees file.
Problems may arise if we are given a new CSV file that is generated or
manipulated by another source. This is because the CSV parser that we have
started to create does not take into account a number of other features
supported by the CSV file format.

Two important ones:

* CSV files often contain comments, lines which start with a pound (#) character
* Columns are unable to support a value which contain a comma (,) character

Our goal is to get in contact with our event attendees. It is not to define a
CSV parser. This is often a hard concept to let go of when initially solving a
problem with programming. An important rule to abide by while building software
is:

> Look for a Solution before Building a Solution

Ruby actually provides a CSV parser that we will instead use throughout the
remainder of this exercise.

## Iteration 1: Parsing with CSV

It is likely the case that if you want to solve a problem, someone has likely
done it in some capacity. They may have even been kind enough to share their
solution or the tools that they created. This is the kind of goodwill that
pervades the Open Source community and Ruby ecosystem.

In this iteration we are going to convert our current CSV parser to use Ruby's [CSV](http://rubydoc.info/stdlib/csv).
We will then use this new parser to access our attendees' zip codes.

### Switching over to use the CSV Library

Ruby's core language comes with a wealth of great classes. Not all of them are
loaded every single time ruby code is executed. This ensures unneeded
functionality is not loaded unless required, preventing ruby from having
slower start up times.

You can browse the many libraries available through the [documentation](http://rubydoc.info/stdlib).

```ruby
require "csv"
puts "EventManager initialized."

contents = CSV.open "event_attendees.csv", headers: true
contents.each do |row|
  name = row[2]
  puts name
end
```

First we need to tell Ruby that we want it to load the CSV library. This is done
through the `require` method which accepts a parameter of the functionality to
load.

The way [CSV](http://rubydoc.info/stdlib/csv) loads and parses data is very
similar to what we previously defined.

Instead of `read` or `readlines` we use CSV's `open` method to load our file.
The library also supports the concept of headers and so we provide some
additional parameters which state this file has headers.

There are pros and cons to using an external library. A 'pro' is how easy this
library makes it for us to express that our file has headers. A 'con' is that
you have to learn how the library is implemented.

### Accessing Columns by their Names

CSV files with headers have an additional option which allows you to access
the column values by their headers. Our CSV file defines several different
formats for the column names. The CSV library provides an additional option
which allows us to convert the header names to symbols.

Converting the headers to symbols will make our column names more uniform and
easier to remember. The header 'first_Name' will be converted to `:first_name`.

```ruby
require "csv"
puts "EventManager initialized."

contents = CSV.open "event_attendees.csv", headers: true, header_converters: :symbol
contents.each do |row|
  name = row[:first_name]
  puts name
end
```

### Displaying the Zip Codes of All Attendees

Accessing the zipcode is very easy using the header name. 'Zipcode' becomes
`:zipcode`.

```ruby
require "csv"
puts "EventManager initialized."

contents = CSV.open "event_attendees.csv", headers: true, header_converters: :symbol
contents.each do |row|
  name = row[:first_name]
  zipcode = row[:zipcode]
  puts "#{name} #{zipcode}"
end
```

We now are able to output the name of the individual and their zipcode.

Now that we are able to visualize both pieces of data we realize that we
have a problem....

## Iteration 2: Cleaning up our Zip Codes

The zip codes in our small sample show us:

* Most zip codes are correctly expressed as a five-digit number
* Some zip codes are represented with less than a five-digit number
* Some zip codes are missing

Before we are able to figure out our attendees' representatives we need to
solve the second issue and the third issue.

* Some zip codes are represented with less than a five-digit number

If we looked at the [larger sample of data](full_event_attendees.csv) we would
see that the majority of the shorter zip codes are from individuals from states
in the north-eastern part of the United States. Many zip codes there start with
0. This data was likely stored in the database as an integer, and not as text,
which caused the leading zeros to be removed.

So in the case of zip codes less than five-digits we will assume that we can
pad missing zeros to the front.

* Some zip codes are missing

Some of our attendees are missing a zip code. It is likely that they forgot to
enter the data when they filled out the form. The zip code data was not likely
marked as mandatory and so our future attendees were not presented with an
error message.

We could try and figure out the zip code based on the rest of the address
provided. We could be wrong with our guess so instead we will use a default,
bad zip code of "00000".

### Pseudocode for Cleaning Zip Codes

Before we start to explore a solution with Ruby code it is often helpful to
express what we are hoping to accomplish in English words.

```ruby
require "csv"
puts "EventManager initialized."

contents = CSV.open "event_attendees.csv", headers: true, header_converters: :symbol
contents.each do |row|
  name = row[:first_name]
  zipcode = row[:zipcode]

  # if the zip code is exactly five digits, assume that it is ok
  # if the zip code is more than 5 digits, truncate it to the first 5 digits
  # if the zip code is less than 5 digits, add zeros to the front until it becomes five digits

  puts "#{name} #{zipcode}"
 end
```

* if the zip code is exactly five digits, assume that it is ok

In the case when the zip code is five digits in length we have it easy. We
simply want to do nothing.

* if the zip code is more than 5 digits, truncate it to the first 5 digits

While zip codes can be expressed with additional resolution (more digits after
a dash) we are only interested in the first five digits.

* if the zip code is less than 5 digits, add zeros to the front until it
  becomes five digits

There are many possible ways that we can solve this issue. These are a few
paths:

  * Use a `while` or `until` loop to prepend zeros until the length is five
  * Calculate the length of the current zip code and add missing zeros to the front
  * Add five zeros to the front of the current zip code and then trim the last five digits
  * Use [String#rjust](http://rubydoc.info/stdlib/core/String#rjust-instance_method) to append zeros to the front of the string.

### Handling Bad and Good Zip Codes

The following solution employs:

* [String#length](http://rubydoc.info/stdlib/core/String#length-instance_method) - returns the length of the string.
* [String#rjust](http://rubydoc.info/stdlib/core/String#rjust-instance_method) - to pad the string with zeros.
* [String#slice](http://rubydoc.info/stdlib/core/String#slice-instance_method) - to create sub-strings either through
  the `slice` method or the array-like notation `[]`

```ruby
require 'csv'

puts "EventManager initialized."

contents = CSV.open 'event_attendees.csv', headers: true, header_converters: :symbol

contents.each do |row|
  name = row[:first_name]
  zipcode = row[:zipcode]

  if zipcode.length < 5
    zipcode = zipcode.rjust 5, "0"
  elsif zipcode.length > 5
    zipcode = zipcode[0..4]
  end

  puts "#{name} #{zipcode}"
end
```

When we run our application, we see the first few output correctly and then the
application terminates.

{% terminal %}
$ ruby lib/event_manager.rb
EventManager initialized.
Allison 20010
SArah 20009
Sarah 33703
David 07306
lib/event_manager.rb:11:in `block in <main>': undefined method `length' for nil:NilClass (NoMethodError)
	from /Users/burtlo/.rvm/rubies/ruby-1.9.3-p374/lib/ruby/1.9.1/csv.rb:1792:in `each'
	from lib/event_manager.rb:7:in `<main>'
{% endterminal %}

* What is the error mesage "undefined method `length' for nil:NilClass (NoMethodError)" saying?

Reviewing our CSV data we notice that the next row specifies no value. An empty
field translates into a nil instead of an empty string. This is choice made by
the CSV library maintainers. So we now need to handle this situation.

### Handling Missing Zip Codes

Our solution above does not handle the case when the zip code has not been
specified. CSV return a `nil` value when no value has been specified in the
column. All objects in Ruby respond to `#nil?`. All objects will return false
except for a `nil`.

We can update our implementation to handle this new case by simply adding a
check for `nil?`.

```ruby
require 'csv'

puts "EventManager initialized."

contents = CSV.open 'event_attendees.csv', headers: true, header_converters: :symbol

contents.each do |row|
  name = row[:first_name]
  zipcode = row[:zipcode]

  if zipcode.nil?
    zipcode = "00000"
  elsif zipcode.length < 5
    zipcode = zipcode.rjust 5, "0"
  elsif zipcode.length > 5
    zipcode = zipcode[0..4]
  end

  puts "#{name} #{zipcode}"
end
```

{% terminal %}
$ ruby lib/event_manager.rb
EventManager initialized.
Allison 20010
SArah 20009
Sarah 33703
David 07306
Chris 00000
Aya 90210
Mary Kate 21230
Audrey 95667
Shiyu 96734
Eli 92037
Colin 02703
Megan 43201
Meggie 94611
Laura 00924
Paul 14517
Shannon 03082
Shannon 98122
Nash 98122
Amanda 14841
{% endterminal %}

### Moving Clean Zip Codes to a Method

It is important for us to take a look at our implementation. During this
examination we should ask ourselves:

* Does the code clearly express what it is trying to accomplish?

The implementation does a decent job at expressing what it accomplishes. The
biggest problem is that it is expressing it near so many other concepts. To
make this implementation clearer we should move this logic into it's own method
named `clean_zipcode`.

```ruby
require 'csv'

def clean_zipcode(zipcode)
  if zipcode.nil?
    "00000"
  elsif zipcode.length < 5
    zipcode.rjust(5,"0")
  elsif zipcode.length > 5
    zipcode[0..4]
  else
    zipcode
  end
end

puts "EventManager initialized."

contents = CSV.open 'event_attendees.csv', headers: true, header_converters: :symbol

contents.each do |row|
  name = row[:first_name]

  zipcode = clean_zipcode(row[:zipcode])

  puts "#{name} #{zipcode}"
end
```

While this may feel like a very small, inconsequential change. Small changes
like these help make your code cleaner and your intent clearer.

### Refactoring Clean Zip Codes

With our clean zip code logic tucked away in our `clean_zipcode` method we can
examine it further to see if we can make it even more succint.

* Coercion over Questions

A good rule when developing in Ruby is to favor coercing values into similar
values so that they will behave the same. We have a special case to deal
specifically with a `nil` value. It would be much easier if instead of checking
for a nil value we convert the `nil` into a string with
[NilClass#to_s](http://rubydoc.info/stdlib/core/NilClass#to_s-instance_method).

{% irb %}
$ nil.to_s
=> ""
{% endirb %}

Examining
[String#rjust](http://rubydoc.info/stdlib/core/String#rjust-instance_method) in
irb we can see that when we provide values greater than 5 it performs no work.
This means we apply it in both cases as it will have the same intended effect.

{% irb %}
$ "123456".rjust 5, "0"
=> "123456"
{% endirb %}

Lastly, examining
[String#slice](http://rubydoc.info/stdlib/core/String#slice-instance_method) in
irb we can see that a number that is exactly five digits in length it has no
effect. This also means we can apply it in cases when the zip code is five
digits or more than five digits and it will have the same effect.

{% irb %}
$ "12345"[0..4]
=> "12345"
{% endirb %}

Combining all of these steps together we can write a more succint
`clean_zipcode` method:

```ruby
def clean_zipcode(zipcode)
  zipcode.to_s.rjust(5,"0")[0..4]
end
```

## Iteration 3: Using Sunlight

We now have our list of attendees with their valid zip codes (at least for most
of them). Using their zip code and the [Sunlight
Foundation](http://sunlightfoundation.com/) webservice we are able query for
the representatives for a given area.

The Sunlight Foundation exposes an API that allows registered individuals
(registration is free) to use their service. Their goal is to provide tools to
make government more transparent and accessible.

> The Sunlight Labs API provides methods for obtaining basic information on Members of Congress, legislator IDs used
> by various websites, and lookups between places and the politicians that represent them. The primary purpose of the
> API is to facilitate mashups involving politicians and the various other APIs that are out there.

### Accessing the API

[http://congress.api.sunlightfoundation.com/legislators/locate?zip=90201&apikey=e179a6973728c4dd3fb1204283aaccb5](http://congress.api.sunlightfoundation.com/legislators/locate?zip=22182&apikey=e179a6973728c4dd3fb1204283aaccb5)

Take a close look at that address. Here's how it breaks down:

* `http://` : Use the HTTP protocol
* `congress.api.sunlightfoundation.com` : The server address on the internet
* `legislators.` : The object name
* `locate.` : The method called on that object
* `?` : Parameters to the method
  * `&` : The parameter separator
  * `zip=90201` : The zipcode we want to lookup
  * `apikey=e179a6973728c4dd3fb1204283aaccb5` : A registered API Key to authenticate our requests

We're accessing the `legislators.locate` method of their API, we send in an
`apikey` which is the string that identifies JumpstartLab as the accessor of
the API, then at the very end we have a `zip`. Try modifying the address with
your own zipcode and load the page.

If you're familiar with writing HTML then this XML document probably makes some
sense to you. You can see there is a `response` object that has a list of
`legislators`. That list contains five `legislator` objects which each contain
a ton of data about a legislator. Cool!

Let's look for a solution before we attempt to build a solution.

### Installing the Sunlight Gem

Steve Klabnik, an instructor for Jumpstart, created the **sunlight-congress**
[gem](https://rubygems.org/gems/sunlight-congress). We call this a wrapper
library because its job is to hide complexity from us. We can interact with it
as a regular Ruby object, then the library takes care of fetching and parsing
data from the server.

The [source code][repo_sunlight_congress] is
available on GitHub.

Ruby comes packaged with the `gem` command. This tool allows you to download
libraries simply knowing the name of the library you want to install.

{% terminal %}
$ gem install sunlight-congress
Fetching: sunlight-congress-1.0.0.gem (100%)
Successfully installed sunlight-congress-1.0.0
Done installing documentation for sunlight-congress (0 sec).
1 gem installed
{% endterminal %}

### Showing All Legislators in a Zip Code

The gem comes equipped with example documentation. The documentation is also
available online with their [source code][repo_sunlight_congress].

Reading through the documentation on how to set up and use the
sunlight-congress gem we find that we need to perform the following steps:

* Set the API Key
* Perform the [query](https://github.com/steveklabnik/sunlight-congress#usage) with the given zip code

```ruby
require 'csv'
require 'sunlight/congress'

Sunlight::Congress.api_key = "e179a6973728c4dd3fb1204283aaccb5"

def clean_zipcode(zipcode)
  zipcode.to_s.rjust(5,"0")[0..4]
end

puts "EventManager initialized."

contents = CSV.open 'event_attendees.csv', headers: true, header_converters: :symbol

contents.each do |row|
  name = row[:first_name]

  zipcode = clean_zipcode(row[:zipcode])

  legislators = Sunlight::Congress::Legislator.by_zipcode(zipcode)

  puts "#{name} #{zipcode} #{legislators}"
end
```

Running our application we find our output cluttered with information.

{% terminal %}
$ ruby lib/event_manager.rb
EventManager initialized.
Allison 20010 [#<Sunlight::Congress::Legislator:0x007fde87d974f8 ...
SArah 20009 [#<Sunlight::Congress::Legislator:0x007fde87d9db50 ...
...
{% endterminal %}

The **legislators** that we are displaying is an array. In turn, the array is
sending the `to_s` message to each of the objects within the array, each
legislator. The output that we are seeing is the *raw* legislator object.

We really want to capture the first name and last name of each legislator.

### Collecting the Names of the Legislators

Instead of outputting each raw legislator we want to print only their first
name and last name. We will need to complete the following steps:

* Iterate over the entire collection of legislators for the particular zip code.
* For each legislator we want to create a new string which is composed of their first name and last name.
* Add the name to a new collection of names.

```
legislator_names = []
legislators.each do |legislator|
  legislator_name = "#{legislator.first_name} #{legislator.last_name}"
  legislator_names.push legislator_name
end
```

The above operation of collecting values is a common operation. Common enough
that Array provides a method
[Array#collect](http://rubydoc.info/stdlib/core/Array#collect-instance_method).
Collect takes the array of objects of inputs and generates a new array as the
output. The last operation we perform in the block is added to our new
collection. This is exactly what we performed above only stated more simply as:

```
legislator_names = legislators.collect do |legislator|
  "#{legislator.first_name} #{legislator.last_name}"
end
```

### Cleanly Displaying Legislators

If we were to replace `legislators` with `legislator_names` in our output we
would be presented with a *slightly* better output.

{% terminal %}
$ ruby lib/event_manager.rb
EventManager initialized.
Allison 20010 ["Eleanor Norton"]
SArah 20009 ["Eleanor Norton"]
Sarah 33703 ["Marco Rubio", "Bill Nelson", "C. Young"]
...
{% endterminal %}

The problem now is that we are still sending the `to_s` message to our new
array of legislator names and by default an array does not know how you want to
display the contents.

We need to explicitly convert our array of legislator names to a string. This
way we are sure it will output correctly. This could be tedious work except
Array again comes to the rescue with the
[Array#join](http://rubydoc.info/stdlib/core/Array#join-instance_method) method.

[Array#join](http://rubydoc.info/stdlib/core/Array#join-instance_method) allows
the specification of a separator string. We want to create a comma-separated
list of legislator names with `legislator_names.join(", ")`

```ruby
contents.each do |row|
  name = row[:first_name]

  zipcode = clean_zipcode(row[:zipcode])

  legislators = Sunlight::Congress::Legislator.by_zipcode(zipcode)

  legislator_names = legislators.collect do |legislator|
    "#{legislator.first_name} #{legislator.last_name}"
  end

  legislators_string = legislator_names.join(", ")

  puts "#{name} #{zipcode} #{legislators_string}"
end
```

Running our application this time should give us a much more pleasant looking
output:

{% terminal %}
$ ruby lib/event_manager.rb
EventManager initialized.
Allison 20010 Eleanor Norton
SArah 20009 Eleanor Norton
Sarah 33703 Marco Rubio, Bill Nelson, C. Young
...
{% endterminal %}

### Moving Displaying Legislators to a Method

Similar to before, with this step complete, we want to look at our
implementation and ask ourselves:

* Does the code clearly express what it is trying to accomplish?

This code is fairly clear in it's understanding. It is simply expressing it's
intent near so many other things. It is also expressing itself differently from
how zip codes are handled. The dissimilarity breeds confusion when returning to
the code.

We want to extract our legislator names into a new method named
`legislators_by_zipcode` which accepts a single zip code as a parameter
and returns a comma-separated string of legislator names.

```ruby
require 'csv'
require 'sunlight/congress'

Sunlight::Congress.api_key = "e179a6973728c4dd3fb1204283aaccb5"

def clean_zipcode(zipcode)
  zipcode.to_s.rjust(5,"0")[0..4]
end

def legislators_by_zipcode(zipcode)
  legislators = Sunlight::Congress::Legislator.by_zipcode(zipcode)

  legislator_names = legislators.collect do |legislator|
    "#{legislator.first_name} #{legislator.last_name}"
  end

  legislator_names.join(", ")
end

puts "EventManager initialized."

contents = CSV.open 'event_attendees.csv', headers: true, header_converters: :symbol

contents.each do |row|
  name = row[:first_name]

  zipcode = clean_zipcode(row[:zipcode])

  legislators = legislators_by_zipcode(zipcode)

  puts "#{name} #{zipcode} #{legislators}"
end
```

An additional benefit of this implementation is that it also encapsulates how we
actually retrieve the names of the legislators. This is a benefit later if we
decide on an alternative to the sunlight gem or want to introduce a level of
caching to prevent look ups for similar zip codes.

## Iteration 4: Form Letters

We have our attendees and their respective representatives. We can now generate
a personalized call to action.

For each attendee we want to include a customized letter that thanks them for
attending the conference and provides a list of their representatives.
Something that looks like:

```html
<html>
<head>
  <meta charset='utf-8'>
  <title>Thank You!</title>
</head>
<body>
  <h1>Thanks FIRST_NAME!</h1>
  <p>Thanks for coming to our conference.  We couldn't have done it without you!</p>

  <p>
    Political activism is at the heart of any democracy and your voice needs to be heard.
    Please consider reaching out to your following representatives:
  </p>

  <table>
    <tr><th>Legislators</th></tr>
    <tr><td>LEGISLATORS</td></tr>
  </table>
</body>
</html>
```

### Storing our template to a file

We could define this template as a large string within our current application.

```ruby
form_letter = %{
  <html>
  <head>
    <title>Thank You!</title>
  </head>
  <body>
    <h1>Thanks FIRST_NAME!</h1>
    <p>Thanks for coming to our conference.  We couldn't have done it without you!</p>

    <p>
      Political activism is at the heart of any democracy and your voice needs to be heard.
      Please consider reaching out to your following representatives:
    </p>

    <table>
      <tr><th>Legislators</th></tr>
      <tr><td>LEGISLATORS</td></tr>
    </table>
  </body>
  </html>
}

```

Ruby has quite a few ways that we can define strings. This format `%{ String
Contents }` is one choice when defining a string that spans multiple lines.

However, placing this large blob of text, this template, within our application
will make it much more difficult to understand the template and the application
code and thus make it more difficult to change the template and the application
code.

Instead of including the template within our application, we will instead load
the template using the same File tools we used at the beginning of the exercise.

* Create a file named 'form_letter.html' in the root of your project directory.
* Copy the html template defined above into that file and save it.

Within our application we will load our template:

```ruby
template_letter = File.read "form_letter.html"
```

It is important to define the `form_letter.html` file in the root of project
directory and not in the lib directory. This is because when the application
runs it assumes the place that you started the application is where all file
references will be located. Later, we move the file to a new location and are
more explicit on defining the location of the template.


### Replacing with `gsub` and `gsub!`

For each of our attendees we want to replace the `FIRST_NAME` and `LEGISLATORS` with their respective values.

* We need to find all instances of `FIRST_NAME` and replace them with the individual's first name.
* We need to find all instances of `LEGISLATORS` and replace them with the individual's representatives.

Our template is a String of text which has two methods for replacing text:
[String#gsub](http://rubydoc.info/stdlib/core/String#gsub-instance_method) and
[String#gsub!](http://rubydoc.info/stdlib/core/String#gsub%21-instance_method).

These two methods are almost identical save for one important difference. The
method `gsub` returns a **new copy** of the original string with the values
replaced. Where `gsub!` will replace the values in the original string.

We have our template letter which we want to use for every attendee. It is
important that we create a new copy of this letter for each attendee. If we
change the original template, they'd all have the same name! By making a copy
and then changing the copy, we're sure everyone's name is unique.

```ruby
template_letter = File.read "form_letter.html"

contents.each do |row|
  name = row[:first_name]

  zipcode = clean_zipcode(row[:zipcode])

  legislators = legislators_by_zipcode(zipcode).join(", ")

  personal_letter = template_letter.gsub('FIRST_NAME',name)
  personal_letter.gsub!('LEGISLATORS',legislators)

  puts personal_letter
end
```

We replace the first name in the template letter and return a new copy (Thanks
[String#gsub](http://rubydoc.info/stdlib/core/String#gsub-instance_method)). We
save the new letter to a personal version of the letter `personal_letter`. We
then replace all the legislators with our legislators information in
`personal_letter` (Thanks
[String#gsub!](http://rubydoc.info/stdlib/core/String#gsub%21-instance_method)).

Methods like `gsub` and `gsub!` can often be confusing and when to use one over
the other may not be immediately clear. The above template manipulation could
have been written with just `gsub`:

```ruby
personal_letter = template_letter.gsub('FIRST_NAME',name)
personal_letter = personal_letter.gsub('LEGISLATORS',legislators)
```

### Our Template System has Problems

It is a treacherous road we start to walk defining our own templating language.
Our current system has some flaws:

* Using FIRST_NAME and LEGISLATORS to find and replace might cause us problems if later somehow this text appears in
  any of our template.

Though not likely, imagine if a person's name contained the word 'LEGISLATORS'.
When we perform the second replacement operation that part of the person's name
would also be replaced. This is unlikely in our simple template, but as our
template grows we may invite such disasters.

* We cannot represent multiple items very easily if they are surrounded by HTML.

Currently we copied our legislators string into a single table column. We would
have a hard time inserting our legislators as individual rows in the table
without having to build parts of the HTML table ourself. This could spell
disaster later if we decide to change the template to no longer use a table.

So, again, instead of building our own custom solution any further we are going to
seek a solution.

### Ruby's ERB

Ruby defines a template language named [ERB](http://rubydoc.info/stdlib/erb/frames).

> ERB provides an easy to use but powerful templating system for Ruby. Using ERB, actual Ruby code can be added to
> any plain text document for the purposes of generating document information details and/or flow control.

Defining an ERB template is extremely similar to our existing template. The
difference is that we use an escape sequence tags which allow us to insert any
variables, methods or ruby code we want to execute.

ERB defines several different escape sequence tags that we can use, the most
common are:

```
<%= ruby code will execute and show output %>
<% ruby code will execute but not show output %>
```

We can define our ERB escape tags within any string. The ruby defined within
the contents of the ERB tags will not be evaluated until we ask the template to
give us the results.

```ruby
require 'erb'

meaning_of_life = 42

question = "The Answer to the Ultimate Question of Life, the Universe, and Everything is <%= meaning_of_life %>"
template = ERB.new question

results = template.result(binding)
puts results
```

The code above loads the ERB library. Creates a new ERB template with the
`question` string. The question string contains ERB tags that will show the
results of the variable `meaning_of_life`. We send the `result` message to the
template with `binding` and

* What is `binding`?

The method
[binding](http://rubydoc.info/stdlib/core/Kernel#binding-instance_method)
returns a special object. This object is an instance of
[Binding](http://rubydoc.info/stdlib/core/Binding). A instance of binding knows
all about the current state of variables and methods within the given scope. In
this case, `binding` here knows about the variable `meaning_of_life`.

Having to explicitly specify a binding when we ask for the results of the
template gives us the flexibility to ask for the results of a template given a
different binding.

### Defining an ERB Template

To use ERB we need to update our current template **form_letter.html**.

* Save a new template as **form_letter.erb**

The convention is to save ERB template files with the extension **erb**. This
is not a requirement. It is a benefit to yourself and other users when they
return to the application.

* Update our existing keywords with the ERB escape sequences

```erb
<html>
<head>
  <meta charset='utf-8'>
  <title>Thank You!</title>
</head>
<body>
  <h1>Thanks <%= name %></h1>
  <p>Thanks for coming to our conference.  We couldn't have done it without you!</p>

  <p>
    Political activism is at the heart of any democracy and your voice needs to be heard.
    Please consider reaching out to your following representatives:
  </p>

  <table>
  <tr><th>Name</th><th>Website</th></tr>
    <% legislators.each do |legislator| %>
      <tr>
        <td><%= "#{legislator.first_name} #{legislator.last_name}" %></td>
        <td><%= "#{legislator.website}" %></td>
      </tr>
    <% end %>
  </table>
</body>
</html>
```

The first use of the ERB tags is familar to our previous example. The second
use, when we display the legislators, is different. We are using the ERB tag
that does not output the results `<% %>` to define the beginning of the block
`<% legislators.each do |legislator| %>` and later the end of the block `<% end
%>`. Inside those tags are the original tags which output the results. In
this case, we are ouputting the first name, last name and website of each
legislator.

This is a departure from what we originally implemented. Before we had to build
the names of all the representatives. We intend now to give the template direct
access to the array of legislators. We will let the template ask and display
what it wants from each legislator.

### Using ERB

We now need to update our application to:

* Require the ERB library
* Create the ERB template from the contents of the template file
* Simplify our `legislators_by_zipcode` to return the the original array of legislators

```ruby
require 'csv'
require 'sunlight/congress'
require 'erb'

Sunlight::Congress.api_key = "e179a6973728c4dd3fb1204283aaccb5"

def clean_zipcode(zipcode)
  zipcode.to_s.rjust(5,"0")[0..4]
end

def legislators_by_zipcode(zipcode)
  Sunlight::Congress::Legislator.by_zipcode(zipcode)
end

puts "EventManager initialized."

contents = CSV.open 'event_attendees.csv', headers: true, header_converters: :symbol

template_letter = File.read "form_letter.erb"
erb_template = ERB.new template_letter

contents.each do |row|
  name = row[:first_name]

  zipcode = clean_zipcode(row[:zipcode])

  legislators = legislators_by_zipcode(zipcode)

  form_letter = erb_template.result(binding)
  puts form_letter
end
```

* Require the ERB library

First we need tell Ruby that we want it to load the ERB library. This is done
through the `require` method which accepts a parameter of the functionality to
load.

* Create the ERB template from the contents of the template file

Creating our template from our new template file requires us to load the file
contents as a string and provide them as a parameter when creating the new ERB
template.

```ruby
template_letter = File.read "form_letter.erb"
erb_template = ERB.new template_letter
```

* Simplify our `legislators_by_zipcode` to return the the original array of legislators

The most surprising change of using ERB is that we have actually reduced the
size and complexity of the `legislators_by_zipcode` method to simply:

```ruby
def legislators_by_zipcode(zipcode)
  Sunlight::Congress::Legislator.by_zipcode(zipcode)
end
```

Looking at the final state of `legislators_by_zipcode`, it may be tempting to
simply remove it. If it's only calling one other method, why bother leaving it
in our code? Sometimes, it's nice to wrap unfamilliar APIs with one that's more
nice for our given situation. Let's leave it in for now.

### Outputting form letters to a file

Outputting each form letter to the screen was useful for ensuring our output
looked correct. It is time to save each form letter to a file.

Each file should be uniquely named. Fortunately, each of our attendees has a
unique id, the first column, or row number.

* Assign an ID for the attendee
* Create an output folder
* Save each form letter to a file based on the id of the attendee


```ruby
contents.each do |row|
  id = row[0]
  name = row[:first_name]

  zipcode = clean_zipcode(row[:zipcode])

  legislators = legislators_by_zipcode(zipcode)

  form_letter = erb_template.result(binding)

  Dir.mkdir("output") unless Dir.exists? "output"

  filename = "output/thanks_#{id}.html"

  File.open(filename,'w') do |file|
    file.puts form_letter
  end

end
```

* Assign an ID for the attendee

The first column does not have a name, like the other columns, so we fall back
to using the index value.

* Create an output folder

We make a directory named "output" if a directory named "output" does not
already exist.

```ruby
Dir.mkdir("output") unless Dir.exists? "output"
```

* Save each form letter to file based on the id of the attendee

[File#open](http://rubydoc.info/stdlib/core/File#open-class_method) allows us
to open a file for reading and writing. The first parameter is the name of the
file. The second parameter is a flag that states how we want to open the file.
The 'w' states we want to open the file for writing. If the file already exists
it will be destroyed.

Afterwards we actually send the entire form letter content to the file
object. The `file` object responds to the message `puts`. The
[file#puts](http://rubydoc.info/stdlib/core/IO#puts-instance_method) is similar to 
the [Kernel#puts](http://rubydoc.info/stdlib/core/Kernel#puts-instance_method)
that we have been using up to this point.

### Moving Form Letter Generation to a Method

Again, for the sake of writing clean and clear code we want to move the
operation of saving the form letter to it's own method:

```ruby
require 'csv'
require 'sunlight/congress'
require 'erb'

Sunlight::Congress.api_key = "e179a6973728c4dd3fb1204283aaccb5"

def clean_zipcode(zipcode)
  zipcode.to_s.rjust(5,"0")[0..4]
end

def legislators_by_zipcode(zipcode)
  Sunlight::Congress::Legislator.by_zipcode(zipcode)
end

def save_thank_you_letters(id,form_letter)
  Dir.mkdir("output") unless Dir.exists?("output")

  filename = "output/thanks_#{id}.html"

  File.open(filename,'w') do |file|
    file.puts form_letter
  end
end

puts "EventManager initialized."

contents = CSV.open 'event_attendees.csv', headers: true, header_converters: :symbol

template_letter = File.read "form_letter.erb"
erb_template = ERB.new template_letter

contents.each do |row|
  id = row[0]
  name = row[:first_name]
  zipcode = clean_zipcode(row[:zipcode])
  legislators = legislators_by_zipcode(zipcode)

  form_letter = erb_template.result(binding)

  save_thank_you_letters(id,form_letter)
end
```

The `save_thank_you_letter` requires the id of the attendee and the form letter
output.

{% exercise %}

## Iteration: Clean Phone Numbers

Similar to the zip codes the phone numbers suffer from multiple formats and
inconsistencies. If we wanted to allow individuals to sign up for mobile alerts
with the phone numbers we would need to make sure all of the numbers are valid
and well-formed.

* If the phone number is less than 10 digits assume that it is a bad number
* If the phone number is 10 digits assume that it is good
* If the phone number is 11 digits and the first number is 1, trim the 1 and use the first 10 digits
* If the phone number is 11 digits and the first number is not 1, then it is a bad number
* If the phone number is more than 11 digits assume that it is a bad number

## Iteration: Time Targeting

The boss is already thinking about the next conference: "Next year I want to
make better use of our Google and Facebook advertising. Find out which hours of
the day the most people registered so we can run more ads during those hours."
Interesting!

Using the registration date and time we want to find out what are the peak
registration hours.

* Ruby has a [Date](http://rubydoc.info/stdlib/date/frames) library which contains classes for
  [Date](http://rubydoc.info/stdlib/date/Date) and [DateTime](http://rubydoc.info/stdlib/date/DateTime).

* [DateTime#strptime](http://rubydoc.info/stdlib/date/DateTime#strptime-class_method) is a method that allows us to
  parse date-time strings and convert them into Ruby objects.

* [DateTime#strftime](http://rubydoc.info/stdlib/date/DateTime#strftime-instance_method) is a good reference on the
  characters necessary to match the specified date-time format.

* Use [Date#hour](http://rubydoc.info/stdlib/date/Date#hour-instance_method) to find out the hour of the day.

## Iteration: Day of the Week Targeting

The big boss gets excited about the results from your hourly tabulations. It
looks like there are some hours that are clearly more important than others.
But now, tantalized, she wants to know "What days of the week did most people
register?"

* Use [Date#wday](http://rubydoc.info/stdlib/date/Date#wday-instance_method) to find out the day of the week.

{% endexercise %}
[repo_sunlight_congress]: https://github.com/steveklabnik/sunlight-congress
