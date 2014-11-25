Title: Inheritance for better exception handling
Date: 2014-11-25 10:35
Category: Python Programming technique
Tags: python, exception handling
Author: Varghese Chacko
Email: vctheguru@gmail.com
about_author: Just another Python/Perl Hacker

In most programming languages that supports exception handling, the base class for all exceptions are called ```Exception```. Let me explain a scenario I recently used. This article uses Python sample codes.

We have two famous e-commerce sites, Amazon and eBay. Assume that we are building a site from which users can buy items from both Amazon and eBay. So the use-case is

<ol>
    <li>The user comes to our site to buy an iPhone 5</li>
    <li>The user types in the search term iPhone 5 in the search box and hit search</li>
    <li>He/she gets a list of iPhones from both Amazon and eBay</li>
    <li>He/she click "Buy" button on an iPone listed and checkout the iPhone</li>
</ol>

Here we are going to focus on step #2, the search. Our search program uses API provided by eBay to search eBay and API by Amazon to search Amazon. If iPhones are there on both sites, well and good.

Suppose the iPhone is not available for sale on eBay, It will raise/throw an exception ```NotFoundOnEbay```. A typical definition of the class would be

```py
class NotFoundOnEbay(Exception):
    # some code to handle eBay exception
    pass
```

When iPone is not available on Amazon, we would like to raise ```NotFoundOnAmazon```. A typical definition of the class would be

```py
class NotFoundOnAmazon(Exception):
    # some code to handle Amazon exception
    pass
```

Assume that we have modules that we made namely ```eBay``` which rises ```NotFoundOnEbay``` and ```Amazon``` which rises ```NotFoundOnAmazon```.

We may write a class like this to search our site.

```py
import eBay
import Amazon

class SearchOnWebsites(object):

    def search(self, keywords):
        ebay_results = eBay(keywords).search()
        amazon_results = Amazon(keywords).search()
```

then in our main program, we may have

```py
try:
    SearchOnWebsites().search(keywords)
    
except Exception as e:
    process_not_found_exception(e)
```

Here ```process_not_found_exception``` knows how to handle the exceptions raised by Not Found errors. Its considered to be leaning towards sloppy as the except block will catch all exceptions and ```process_not_found_exception``` may cause exceptions if it receives some exception that it cant handle. A solution would be

```py
try:
    SearchOnWebsites().search(keywords)
    
except NotFoundOnAmazon:
    process_not_found_exception(e)
    
except NotFoundOnEbay:
    process_not_found_exception(e)
    
except Exception:
    process_exception(e)
```

OR

```py
try:
    SearchOnWebsites().search(keywords)
    
except (NotFoundOnAmazon, NotFoundOnEbay) as e:
    process_not_found_exception(e)
    
except Exception:
    process_exception(e)
```

Here we assume that ```process_exception``` handles all non-specific exceptions.

The except tree/list will grow whenever we add another e-commerce site and if we have fifteen of them, how will it look?

Here is a fix by defining a base class for all exceptions we catch. 


```py
# Base class for all not for found with vendor exception

class ItemNotAvailableWithVendors(Exception):
    pass

# Item not available with Amazon

class NotFoundOnAmazon(ItemNotAvailableWithVendors ):
    # some code to handle Amazon exception
    pass
    

# Item not available with eBay

class NotFoundOnEbay(ItemNotAvailableWithVendors ):
    # some code to handle eBayexception
    pass
```

then we change our code to

```
try:
    SearchOnWebsites().search(keywords)
    
except ItemNotAvailableWithVendors as e:
    process_not_found_exception(e)
    
except Exception:
    process_exception(e)

```

The fact is that we can raise an exception (child class) and can catch it with its parent, but the exception we got will be the instance of child class we actually raised and hence handle them properly. 

If it raised <code>NotFoundOnAmazon</code> we get the instance of <code>NotFoundOnAmazon</code> and if it raised <code>NotFoundOnEbay</code> we get instance of  <code>NotFoundOnEbay</code> and so on and will be caught as <code>ItemNotAvailableWithVendors</code>. Experts in other languages may try this scenario for their language and put as comment.

Note: This article was originally posted at <a href="http://www.onlinetoolbag.com/blog/using-inheritance-for-better-exception-handling">www.onlinetoolbag.com</a> by author.


