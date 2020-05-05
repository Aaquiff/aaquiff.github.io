---
layout: post
title:  "Cross-Site Request Forgery"
date:   2018-10-10 00:50:05 +0530
categories: csrf
---

# What is is CSRF?

**Cross-Site Request Forgery(CSRF)** is a web browser based attack that tricks the user to execute operations on a 
different domain than that the current browsing domain. It is used to perform operations unknowingly to the user but could be used for theft of information as well. Imagine you are logged into Facebook and browsing a malicious website at the same time and the attacker knows the request format for a operation that could alter the users account and includes an ajax call in the malicious website that performs the state changing request to Facebook. The browser will send this request along with all cookies related to Facebook as well and hence will seem to be a legitimate request.

# Protection

To protect against CSRF, we need to prove that the request is coming from the form, that we have provided to the user. In order to do this validation we use a token that is embedded into the form page and subsequently sent with the post request to the server. Request(like AJAX) is  sent on form load to receive this token. A question arises here, couldn’t the malicious website do the same to get the token from the server? No! because of Same-Origin policy which defines that the request can only come from the same domain.

There are two methods of implementing protection against CSRF and are explained below.

## Synchronous Token Pattern

In this method, the server generates a token for each session and stores it in a key value like store. The next time the form loads, it requests the token from the server to which the server responds and the browser embeds the token in the form. Subsequently, when the form is posted, the token is sent along with the form data. The server then validates the received token against the token in the server that maps to the session id to check if it is from the valid form.

This can help avoid CSRF However when there are huge heap of connections open, the server needs to maintain these tokens and this would be problematic in terms of storage and performance. To avoid the need for the server to store tokens, an alternative pattern called Double Submit Cookie Pattern is used.

Sample of a Java Web Application protected with Synchronous Token Pattern can be found [here](https://github.com/Aaquiff/csrf-synchronizer-token-pattern-example).

## Double Submit Cookie Pattern

This, instead of storing the token in the server, creates a cookie for the token in the browser. Now the browser has the CSRF token embedded in the form as well as a cookie containing the same CSRF token. When the data is posted to the server, the CSRF token in the cookie is sent along with the token in the form data.The server, then validates that both token values are equal to confirm that the request is coming from the legitimate site.

Sample for a Java Web Application protected with Double Submit Cookie Pattern can be found [here](https://github.com/Aaquiff/csrf-double-submit-cookie-example).