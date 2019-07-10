# ctf-practice
This repository documents what I have learnt from exploring CTF challenges.

# Google CTF > Web > bnv
Link: https://bnv.web.ctfcompetition.com/  
There is not much to see in this enterprise-ready™ web application.  

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
* To-be-continued...