# OWASP Juice Shop challenge: Manipulate Basket

## About the challenge

|              |                                                                |
| ------------ | ---------------------------------------------------------------|
| Titel        | Manipulate Basket                                              | 
| Category     | Broken Access Control                                          |
| Task         | Put an additional product into another user’s shopping basket  |
| Difficulty   | ⭐️⭐️⭐️                                                        |

### Description

This challenge is about bypassing permissions in the Juice Shop to manipulate the contents of another user’s shopping cart. The first step is to understand how the GET and POST requests work that retrieve the contents of a shopping cart and add products to it. A good starting challenge for this is “View another user’s shopping basket”.

To successfully complete this challenge, a product must be added to an existing shopping cart of another user, provided that the product is not already in the cart. Simply changing the quantity of an existing product is not sufficient. Likewise, removing a product does not meet the requirements to complete the challenge.

<br>

## Approach

### 1. 
