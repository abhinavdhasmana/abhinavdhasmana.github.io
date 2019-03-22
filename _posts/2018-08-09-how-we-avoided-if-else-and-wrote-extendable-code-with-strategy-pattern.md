---
layout: post
title:  "How we avoided if/else and wrote extendable code with Strategy Pattern"
tags: [Node.js, Design pattern, JavaScript]
author: Abhinav Dhasmana
---
We were working for a telecom client where we were rolling out `offers` for our customers. They had tons of business logic based on which the user will be given an `offer`. In addition to that, they would create more categories of `offer` as time passes by. We saw the code that looked like this

{% gist 8c462a9244a3f8a6b3656b7439575e19 %}

Every-time a new `offer` type was introduced, or existing rules needed modification, we needed to change `User` class.

Calling this class looked something like this

{% gist 5f7ecb4c26b7b4aa1e51b7283404ebf0 %}

We have following issues with the code:

*   Violates “O” in [SOLID principle](https://en.wikipedia.org/wiki/SOLID): According to SOLID principle

> “software entities … should be open for extension, but closed for modification.”

However, whenever we have to generate new offer category, we would have to change a `User` class.

*   Multiple `if/else` makes the code difficult to test and maintain
*   `offer` and `User` are tightly coupled

### Strategy Pattern to the rescue

Strategy pattern allows us to vary the algorithm independently from clients that use it. In our case, we want to modify the algorithm for `offer` independently from the `User`.

Let’s create a new class called `UserOffer`

{% gist 3d7ad9bf9bbf86a46593ae64c0f9c300 %}

Important things to note:

*   At `line 3`, it’s a function reference and not an actual call.
*   Starting `line 11`, we can create new offers

Our `User` class now becomes far simpler

{%gist 720f30b83de9b61e0170d99cf7c5c38f %}

In the above code, we have introduced `setOffer` which is similar to `setType`. `getOffer` now just calls the `offer.getOffer` which would return the relevant offer.

*   This approach gives us the flexibility to modify/create new offers without changing the `User` class.
*   We have avoided `if/else` coding style
*   Our code is easier to extend and test.