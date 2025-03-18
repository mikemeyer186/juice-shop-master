# OWASP Juice Shop challenge: Payback Time

## About the challenge

|            |                                    |
| ---------- | ---------------------------------- |
| Titel      | Payback Time                       |
| Category   | Inproper Input Validation          |
| Task       | Place an order that makes you rich |
| Difficulty | ⭐️⭐️⭐️                          |

### Description

This challenge is about manipulating the checkout process in a way that results in receiving a refund from the Juice Shop. The first step is to analyze how the order process works and identify which HTTP requests are responsible for adding items to the shopping cart. After that, it is crucial to understand how the data is validated during this process.

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
    "quantity":<quantity of product>      # quantitiy can also be negative, but not greater than 5
}
```

<br>

### 2. Manipulate the POST request

You have to manipulate the POST request to add a new `ProductId` to you basket. The same ProductId must not already be present in the shopping cart. For some products there is a maximum oder quantity (e.g. 5). But this validation is only for positiv numbers. You can send a negativ number within the payload of the POST request (e.g. -1000).

```
POST /api/BasketItems/                    # adds an item to the basket with specific payload

Payload of POST request:

{
    "ProductId":<id of product>,          # id of product from product catalog
    "BasketId":<number of own basket>,    # number of basket of own user
    "quantity":<quantity of product>      # quantitiy can also be negative
}
```

<br>

### 3. Checkout

After adding a `ProductId` with a high negativ number (e.g. -1000) to you basket with a manipulated POST request, you can easily start the checkout process of the cart in the frontend of the Juice Shop. In the payment options at the checkout process you can decide how you want to pay the order. The total sum of the order is now a negative number (e.g. -22,568$). Even if the balance of the wallet is 0.00$ the option to pay with the wallet is choosable. It seems that the only validation here is to compare if the wallet balance is greater than the total sum of the order. Due to the negativ total sum, it is the case and the order can be paid with the wallet. You can place the order and and pay with the wallet.

After the checkout of the cart, you can see that the negative total price of the order is now as a positve amount in the wallet. It seems that just a simple mathematical formula is responsible for that behavior.

```
Wallet Balance - Total Sum of Order + Bonus Points = 0.00$ - (-22,568$) + (-1994$) = 20,574$
```

Now, the wallet can be used as a payment option for the next orders.

<br>

## Explaination

### Vulnerability

The vulnerability here lies in the validation of the data sent to the server as a payload in the POST request that adds items to the shopping cart. In the Juice Shop frontend, users cannot select a quantity less than 1 for a product in the cart. The maximum quantity depends on the predefined limit for that specific product (e.g., stock availability, maximum of 1 or 5 items, etc.). By manipulating the POST request, it is possible to add a negative quantity of a product to the shopping cart. The checkout process can still be initiated, even if the total sum in the cart is negative, which represents another vulnerability. When selecting a payment option, the system performs a simple comparison between the available amount in the wallet and the order total (Wallet > Order Total). Since this condition is true in the case of a negative order total, the wallet can be selected as the payment option, and the checkout process can be completed.

As a result, the amount that the Juice Shop “owes” the user is credited to their wallet (minus the bonus points).

### Conseqences

In a real-world scenario, this vulnerability could cause massive financial damage to the shop, potentially leading to bankruptcy. Once the wallet is fraudulently credited with funds, it can be used for further real purchases. Some shops also offer the option to withdraw wallet funds to a bank account or credit card. In that case, it would be possible for real money to be transferred to cybercriminals, further escalating the financial impact.

### How to avoid inproper validations?

This vulnerability can be prevented by properly validating the shopping cart data. It must not be possible to add negative quantities of products to the cart. While this is already enforced in the frontend, server-side validation of the payload is also necessary. Any manipulated POST requests should be rejected by the server with an appropriate error message. Additionally, the checkout process validation should be improved. Completing an order with a negative total amount should either be entirely prevented or be subjected to manual review by a shop employee.
