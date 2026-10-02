# Excessive Trust in Client-Side Controls

<img width="960" height="892" alt="image" src="https://github.com/user-attachments/assets/24a272a0-f8db-4b24-a377-18414ca0f1cc" />


### Objective

The goal was to buy the **Lightweight l33t leather jacket**, even though my account didn't have enough store credit.

### My Approach

At first, I spent quite a while looking at the `Buy now` POST request in Burp and trying to figure out what could be manipulated. Nothing unusual really stood out to me there.

After looking through the previous requests for a while, I noticed another **POST request** that was sent when the product was added to the cart.

<img width="941" height="697" alt="Screenshot 2026-10-02 231806" src="https://github.com/user-attachments/assets/ab03a360-de69-4b8e-b8a9-1d20b47e923b" />


<img width="597" height="445" alt="Screenshot 2026-10-02 231938" src="https://github.com/user-attachments/assets/deae36b5-4091-4e7b-8e6c-9efeb88b03f0" />



That request contained parameters such as the `productId` and `price`, along with their values.

That caught my attention because the price was being sent by the client.

I turned on interception and sent the request again. Then I changed the value of the `price` parameter to a much lower amount.

<img width="953" height="868" alt="Screenshot 2026-10-02 232315" src="https://github.com/user-attachments/assets/1dca34c3-b8ad-4aa1-9c30-5b3931ef2bdb" />



After sending the modified request, I checked the cart and saw that the product had been added with the lower price.

<img width="940" height="690" alt="Screenshot 2026-10-02 232421" src="https://github.com/user-attachments/assets/3bd3bee4-f103-4fdc-af4d-438e82160610" />



From there, I could place the order using my available store credit and complete the lab.

### What Was Wrong?

The application was **trusting the price sent by the client**.

The website's interface did not normally allow me to choose the product price, but the server accepted the `price` value from the HTTP request without properly validating it.

So the flow was effectively:

```text
Client sends product + price
        ↓
Server accepts the supplied price
        ↓
Attacker modifies price
        ↓
Cart uses the modified price
```

### What Should Have Happened?

The server should not trust the price supplied by the client.

Instead, it should use the `productId` to retrieve the actual product price from its own trusted database/server-side data:

```text
Client sends productId
        ↓
Server looks up product
        ↓
Server gets the actual price
        ↓
Server calculates the cart total
```

Even if someone changes `price` in Burp, the server should ignore or reject that value.

### Key Takeaway

This lab showed me why **client-side restrictions are not security controls**.

When testing business logic, I should pay attention to important values such as `price`, `quantity`, `discount`, and `amount`, and check whether the server actually validates them instead of trusting whatever the client sends.
