# OWASP Juice Shop challenge: Manipulate Basket

## About the challenge

|            |                                                               |
| ---------- | ------------------------------------------------------------- |
| Titel      | Manipulate Basket                                             |
| Category   | Broken Access Control                                         |
| Task       | Put an additional product into another user’s shopping basket |
| Difficulty | ⭐️⭐️⭐️                                                     |

### Description

This challenge is about bypassing permissions in the Juice Shop to manipulate the contents of another user’s shopping cart. The first step is to understand how the GET and POST requests work that retrieve the contents of a shopping cart and add products to it. A good starting challenge for this is “View another user’s shopping basket”.

To successfully complete this challenge, a product must be added to an existing shopping cart of another user, provided that the product is not already in the cart. Simply changing the quantity of an existing product is not sufficient. Likewise, removing a product does not meet the requirements to complete the challenge.

> [!NOTE]
> You have to use a software tool to intercept and manipulate the HTTP requests (e.g. Burp Suite) in your local environment.

<br>

## Approach

### 1. Identify which requests are responsible for modifying the shopping cart

While adding a new item to your own basket, you can intercept the HTTP traffic to see which requests are called (you must be logged in with your own user).

```
GET /rest/basket/<number of basket>       # returns the content of every basket
POST /api/BasketItems/                    # adds an item to the basket with a specific payload

Payload of POST request:

{
    "ProductId":<id of product>,          # id of product from product catalog
    "BasketId":<number of basket>,        # number of basket of specific user (serves as access control)
    "quantity":<quantity of product>      # quantitiy can also be negative
}
```

<br>

### 2. Manipulate the POST request

You have to manipulate the POST request to add a new `ProductId` to someone elses basket. The `BasketId` serves as the access control. Trying to change the number of `BasketId` will end in an error message from the server ("Invalid BasketId"). But adding an additional parameter in the payload with the `BasketId` of someone else, will lead to succes.

```
POST /api/BasketItems/                    # adds an item to the basket with specific payload

Payload of POST request:

{
    "ProductId":<id of product>,          # id of product from product catalog
    "BasketId":<number of own basket>,    # number of basket of own user (serves as access control)
    "BasketId":<number of other basket>,  # number of basket of other user (following after our own BasketId)
    "quantity":<quantity of product>      # quantitiy can also be negative, but not greater than 5
}
```

<br>

## Explaination

### Vulnerability

The vulnerability here lies in the authorization check of the POST request on the server side. The POST request should only be successfully executed if the BasketId is assigned to the correct user. This check works as long as only one BasketId is included in the payload. This can be observed because when changing the BasketId, the server returns the error message “Invalid BasketId”. However, if an additional, existing BasketId is included as an extra parameter in the payload, the authorization check passes successfully. As a result, the item is added to the other user’s shopping cart.

> [!NOTE]
> This attack is called HTTP Parameter Pollution Attack

Submitting multiple HTTP parameters with the same name can cause an application to interpret values in unexpected ways. By exploiting these effects, an attacker may be able to bypass input validation, trigger application errors, or manipulate internal variable values. If a developer is unaware of the issue, the presence of duplicate parameters can lead to anomalous behavior in the application, which could potentially be exploited by an attacker. Unexpected behaviors are a common source of vulnerabilities, which in this case, could lead to HTTP Parameter Pollution attacks.

### Conseqences

As a consequence, an attacker could manipulate other users’ shopping carts, potentially causing significant damage to the shop. The primary risks include financial damage (e.g. incorrect orders, complaints or loss of customers) and reputational damage resulting from a loss of customer trust.

### How to avoid HTTP Paramter Pollution Attacks?

Developers should be aware of how their backend handles multiple parameters with the same name in the payload of a POST request. It must be ensured that the access control cannot be bypassed. One possible solution is to have the server reject any HTTP request containing multiple parameters with the same name by returning an error message.
