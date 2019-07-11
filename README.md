# ctf-practice
This repository documents what I have learnt from exploring CTF challenges.

# Google CTF > Web > bnv
Link: https://bnv.web.ctfcompetition.com/  
There is not much to see in this enterprise-ready™ web application.  
Reference: https://www.youtube.com/watch?v=rcgq8LyNDaQ

* This is what greets us when we load the page initially:
![](/screenshots/google-web-bnv/initialPage.jpg)
* There are 3 available cities to choose from: Zurich, Bangalore and Paris.
* Opening up the page source, something at the bottom draws our attention - `/static/post.js`.
![](/screenshots/google-web-bnv/initialPageSource.jpg)
* This is what we find within `/static/post.js`:
![](/screenshots/google-web-bnv/postJs.jpg)
* Using BurpSuite to intercept the request (which is fired after pressing the `Submit` button), with the city Zurich selected:
![](/screenshots/google-web-bnv/burpIntercept.jpg)
* The only body content sent is `{"message":"<NUMBER>"}`. The number is dependent on the city name only, and does not include any other factors (such as time), i.e. submitting the same city name multiple times results in the same body content being sent.
![](/screenshots/google-web-bnv/burpInterceptList.jpg)
* If we were to modify the number in `message` to a random one (such as `123` as seen below, or `abc`), we get the response `{"ValueSearch": "No result found"}`:
![](/screenshots/google-web-bnv/burpInterceptModifiedRequest.jpg)
* Removing the body content altogether gives us the response `{"ValueSearch": "No JSON object could be decoded"}`:
![](/screenshots/google-web-bnv/burpInterceptModifiedRequestEmpty.jpg)
**Background**: The information below are taken from an [article](https://blog.netspi.com/playing-content-type-xxe-json-endpoints/) on NetSPI, titled `Playing with Content-Type – XXE on JSON Endpoints`.
* Most common data formats for web services are XML and JSON.
* While a web service may be programmed to use just either XML or JSON, the server may accept data formats that the developers did not anticipate.
* Thus, JSON endpoints may be vulnerable to **XML External Entity attacks (XXE)**, an attack that exploits weakly configured XML parser settings on the server.
**Back to the challenge**:
* When we modify only the `Content-type` to `application/xml` instead of `application/json`, we get the response that "Start tag expected, '<' not found, line 1, column 1". This tells us that the **server accepts the XML content type as well!**
![](/screenshots/google-web-bnv/burpInterceptModifiedRequestXML.jpg)
* Next, we modify our body content to one that is formatted in XML, in addition to having the content-type set to XML, and it tells us that "Validation failed: no DTD found !". I followed the article's advice to add the `<root>` element as well, but it gives the same response.
![](/screenshots/google-web-bnv/burpInterceptModifiedRequestXMLFormattedRequest.jpg)
* DTD stands for Document Type Declaration. Next, we attempt to brute-force for a local DTD file on the target host, with reference to [this article](https://mohemiv.com/all/exploiting-xxe-with-local-dtd-files/). The response that we get is "failed to load external entity". (Note that we did not change any of the `file://`.
![](/screenshots/google-web-bnv/burpInterceptModifiedRequestXMLDTD.jpg)
* We make another attempt to change the first entity file to `file:///usr/share/yelp/dtd/docbookx.dtd`, which was seen in the site. While it was still an unsuccessful attempt, we get a different error message: "No declaration for element message". This perhaps meant that the external entity could now be loaded, but there is another XML formatting error that we have to take a look at.
![](/screenshots/google-web-bnv/burpInterceptModifiedRequestXMLDTD2.jpg)
* The main change which we made to solve this error was to remove the "element message" altogether. Next, we modified the second ENTITY line. Besides these 2 changes, redundant code was removed to make it neater. And then we have our `/etc/passwd` file displayed!
![](/screenshots/google-web-bnv/burpInterceptModifiedRequestXMLDTDSuccess.jpg)
* Instead of displaying the `/etc/passwd` file, we want to get the flag, so we change it to `/flag` instead:
![](/screenshots/google-web-bnv/burpInterceptModifiedRequestXMLDTDFlag.jpg)
* Note: We would normally expect the flag to be in `/root/flag` when chasing down VulnHub machines, but it isn't the case here, because from the `/etc/passwd` file, we notice that there is no `root` user!
* Our flag is `CTF{0x1033_75008_1004x0}`.

# Google CTF > Reversing > Malvertising
Link: https://malvertising.web.ctfcompetition.com/  
Unravel the layers of malvertising to uncover the Flag  
Reference:
1. https://balsn.tw/ctf_writeup/20190622-googlectfquals/#malvertising
2. https://www.hackiit.cf/write-up-google-ctf-malvertising/
3. https://github.com/EmpireCTF/empirectf/tree/master/writeups/2019-06-22-Google-CTF-Quals#140-reversing--malvertising

* Clicking into the link loads a screenshot of a YouTube site, with the top of the page standing out - "Your advertisement here".
![](/screenshots/google-reversing-maladvertising/initialLoad.jpg)
* Clicking on anywhere within the HTML element of the `iframe` results in the page being redirected to google.com.
* Opening up the page source, we see `ads/ad.html` that is the source of the iframe:
![](/screenshots/google-reversing-maladvertising/initialLoadPageSource.jpg)
* Clicking on it directly will not show us the code - we have to go to the Developer Tools console to check it out:
![](/screenshots/google-reversing-maladvertising/adHTML.jpg)
* We see the script that results in the loading of google.com upon clicking.
* There is a bunch of iframe links at the bottom, which do not seem to serve any purpose.
* Lastly, I found `/ads/src/metrics.js`, which I did not quite know how to make sense of anything in it even though I had pretty-printed it using Chrome's tool.
* So I found out that the file contents are not only minimised, but also completely obfuscated. At the end of the file, there is a chunk of code that is noteworthy:
![](/screenshots/google-reversing-maladvertising/metricsJSEnd.jpg)
* Converting some of the hexadecimal content on line 469 to ASCII leads us to the discovery of `android`.
* We have to insert `android` into the `User-Agent` parameter before the next file is revealed to us.
* At this point in time I am pretty much lost so I will be doing up some reading before making another attempt here.
* To-be-continued...