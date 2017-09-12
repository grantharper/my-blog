---
layout: post-bootstrap
title:  "Web App Security"
date:   2017-09-12
categories: software
---

With security breaches happening almost constantly, I thought it would be a fun project to put together a purposefully vulnerable application and then demonstrate how it could be defended against some common security vulnerabilities. You can download the application to run on your local machine from [my github][security-demo].

In the process of creating this sample application, I became more aware of just how insecure many websites likely are. And the scary thing is that vulnerable websites don't just put themselves at risk. They also put visitors in harm's way as well.

The security demo web application currently illustrates SQL injection, cross-site scripting (xss), insecure direct object reference, and cross-site request forgery (csrf). Once you have the app running, below is an interesting flow that illustrates just how badly things can go wrong. For details on how to perform these actions, take a look at the [README][README].

* Log in as an employee to view the general employee functionality
* Log in to a user profile and attack with xss vulnerability
* Log back in to the employee portal and see the hack happen based on stored xss
* Check the hack-output.txt to see the bank account info that was transmitted
* Now that the hacker knows what is in each account, use SQL injection to move money from another account to the hacker's account
* Lure employee to use csrf to close the account that was compromised using the sample forum page below

Below is a sample forum page that can conduct a csrf attack on the security demo app
<div>
<div class="well">
<h1>Welcome to the awesomest forum ever!</h1>
<p>Click below to discover the video trending all over the internet right now!</p>
<form action="http://localhost:8080/employee/customer/1/account/1/close" method="post">
    <button class="btn btn-primary">CSRF to Insecure</button>
</form>
<br/>
<p>To win a million dollars, click below!</p>
<form action="http://localhost:8081/employee/customer/1/account/1/close" method="post">
    <button class="btn btn-primary">CSRF to Secure</button>
</form>
<br/>
<p>Click to see the most surprising fact about eating fruit daily!</p>
<form action="http://grantharper.org" method="post">
    <button class="btn btn-primary">CORS Test</button>
</form>
</div>
</div>

If you're interested in this sort of thing, I would highly recommend the [NATAS][natas] web security demo website. Below are the passwords for future reference and reminders of the bad practices that allow for vulnerabilities to be introduced.

The table below identifies how to crack a few of the initial levels.

**Username** |  **Password Discovery Method**
natas0 | Given to you
natas1 | In the page source
natas2 | In the page source, but you have to view it through the developer console
natas3 | In the files directory of the site under users.txt
natas4 | Google uses robots.txt which excludes the s3cr3t directory where the users.txt can be found
natas5 | Get basic authorization setup and then spoof the referer header to be the correct url
natas6 | Notice that the cookie has a parameter `loggedin=0` and change this to `loggedin=1` in the header

[security-demo]: https://github.com/grantharper/security-demo
[natas]: http://overthewire.org/wargames/natas/
[README]: https://github.com/grantharper/security-demo/blob/master/README.md
