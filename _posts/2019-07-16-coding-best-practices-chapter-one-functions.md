---
layout: post
title: 'Coding Best Practices, Chapter One: Functions'
author: levi_velazquez
date: '2019-07-16 23:21:30'
---

My purpose IS NOT to say what is the best way to write code, some programmers have their ways and they are just fine. I just want to share with others what I consider code best practices and how to improve several aspects as readiness, structure, components, meaning, debugging, refactoring, etc.

# Chapter 1: Functions

## Do One Thing.

I wanted to start from the most fundamental rule and according to me, the best practice: __The One Thing Rule__. It will make our life easier, try to create your functions to Do One Thing.

It could be very hard to write functions that do just one thing, but it should be our ultimate goal in our daily process. Why? Let's look a this

```python
def create_user(email, password):
    try:
        user = User.objects.get(email=email)
    catch UserDoesNotExist:
        user = User.objects.create(
            email=email,
            password=password
        )
        auth_valid = authenticator(
            user=user,
            password=password
        )
        if auth_valid:
            user.last_login = datetime.now()
            user.save()
            msg_content = '<h2>Welcome {email}</h2>'.format(
                email=email
            ) 
            message = MIMEText(
                msg_content,
                'html'
            )
            message.update({
                'From': 'Sender Name <sender@server>',
                'To': email,
                'Cc': 'Receiver2 Name <receiver2@server>',
                'Subject': 'Account created'
            })
            msg_full = message.as_string()
            server = smtplib.SMTP('smtp.gmail.com:587')
            server.starttls()
            server.login(
                'sender@server.com',
                'senderpassword'
            )
            server.sendmail(
                'sender@server.com',
                [email],
                msg_full
            )
            server.quit()
            return user
```  

__What this function does ?__

* If the user doesn't exist, it creates a new user.
* It creates an HTML Template.
* It initializes a SMTP connector.
* It sends an email.

__Why it is wrong ?__
1. Is doing more than One Thing.
2. The unit test for that function is going to be huge and messy, why? E.g, the testing logic
should consider the authentication and the welcome email process at the same time, when those process should be tested separately.
3. It's easier to introduce errors, let's say we want to change the welcome email, we'll need to touch our function, so, making changes in process
A for fixing process B doesn't make sense.
4. If we try to do more than one thing, it would increase our function line numbers, losing readiness.
5. More things to do, it's harder to debug.

Said that, how we can try to fix or reduce the issues listed above? Let's refactor our code.

```python
def signup(email, password):
    new_user = create_user(
        email,
        password
    )
    authenticate(user)
    send_welcome_email(email)

def create_user(email, password):
    try:
        user = User.objects.create(
            email=email,
            password=password
        )
        return user
    except UserAlreadyExists:
        raise Exception('User exists')

def authenticate(user, password):
    auth_valid = authenticator(
        user=user,
        password=password
    )
    if auth_valid:
        user.last_login = datetime.now()
        user.save()

def send_welcome_email(email):
    email_msg = create_welcome_email_msg(email)
    send_email_to(
        email,
        email_msg
    )

def create_welcome_email_msg(email):
    msg_content = '<h2>Welcome {email}</h2>'.format(
        email=email
    ) 
    message = MIMEText(
        msg_content,
        'html'
    )
    message.update({
        'From': 'Sender Name <sender@server>',
        'To': email,
        'Cc': 'Receiver2 Name <receiver2@server>',
        'Subject': 'Account created'
    })
    msg_full = message.as_string()
    return msg_full

def send_email_to(email, email_msg):
    server = smtplib.SMTP('smtp.gmail.com:587')
    server.starttls()
    server.login(
        'sender@server.com',
        'senderpassword'
    )
    server.sendmail(
        'sender@server.com',
        [email],
        msg_full
    )
    server.quit()
```

__What we did ?__

Much better, right?

- We split up our `create_user` function into smaller functions, allowing us to separate the logic and adding a new different level of abstractions like `signup function`. 
- We applied the same principle (One Thing Rule) to the rest of the functions as well.
So, readiness was improved, you can easily do unit testing and you can now change your welcome email without touching anything about user creation process.

__Pro-tip__:
So, how we can know when our function is not doing just one thing? the simplest way is _if you can extract some logic from your function and putting it in a new one using a different logic name_, most likely your function was doing more than one thing. 

## Level of abstraction
In order to ensure our functions are doing one thing, we should keep the same level of abstraction across all function.

In our past example (Code Preview 1), we could spot different levels of abstraction, we are authenticating the user using a
high level function `authenticator(user=user,password=password)` but at the same time, we're creating  a HTML template from a string `msg_content = '<h2>Welcome {email}</h2>'.format(email=email)`. This kind of function definition should be avoided. 

Basically, we should be consistent at the moment we create our function. It also helps the reader to understand if our function is a high-level concept or it is a detailed process.

## Sections within functions

Another way to spot if you function is doing more than one thing,  if you can split it up by different sections grouping bunch of lines

``` python
def foo():
    # initialization
    # ...... several lines here
    # logic 1
    # ....... several lines here
    # Prepare return value
    # ....... several lines here 
```
In this example, we are doing more than one thing at the same level, in order to avoid this, you can extract every section in other functions and then use them together, of course, it depends on the level of abstraction your function has.

## Identation level

I encountered myself trying to understand functions with more than 2 levels of indentation. If you did as well, we can be agree
on the maximum level should be 2 as the worst case.

If you function have a lot of `if/else` blocks within other `if/else` blocks, for sure, your function is not doing One Thing. Let's see an example.

```python
def buy_concert_ticket(user, ticket):

    ticket_price = ticket.price
    user_age = user.age

    if user.age >= 18:
        user_money = user.money
        if user_money >= ticket_price:
            seats = stadium.seats
            seat_available = None
            for seat in seats:
                if seat.is_available():
                    seat_available = seat
                    break

            if seat_available: 
                seat.owner = user
                user.money -= ticket.price
                print("Congratulations, you have a seat")
            else:
                print("There is not available seat")

        else:
            print("Sorry you don't have enough balance")
    else:
        print("You are not allowed to buy a ticket")
```
So, these functions allow a user to buy a ticket, but before to complete the purchase, it verifies some conditions. 
This function doesn't meet our previous rules:
- It has more than 2 levels of indentation/blocks.
- It does more than one thing.
- It has several sections.
- It difficult to read due to nested if/else blocks.

How to fix this?
1.- We should try to decrease the level of indentation by removing nested if/else blocks. 

```python
def buy_concert_ticket(user, ticket):

    ticket_price = ticket.price
    user_age = user.age
    user_money = user.money

    if not user.age >= 18:
        print("You are not allowed to buy a ticket")
        return 

    if not user_momey >= ticket_price:
        print("Sorry you don't have enough balance")
        return

    for seat in steats:
        if seat.is_available():
            seat.owner = user
            user.money -= ticket.price
            print("Congratulations, you have a seat")
            return

    print("There is no available seat")
    return
```

You see, we were able to drop all the `if/else blocks`, so our max indentation level now is 2. Now, the function
has more readiness.

__Pro-Tip__: If you try to make simpler functions using the explained rules, you will never need to use an `else` again for function logic purposes.   

2.- Let's identifies the things our function does and extract them to separated functions. The refactored code should look like this.

```python
def buy_concert_ticket(user, ticket):

    if not user_has_legal_age(user):
        print("You are not allowed to buy a ticket")
        return

    if not user_has_enough_balance(user, ticket):
        print("Sorry you don't have enough balance")
        return
    
    available_seat = get_available_seat()

    if not available_seat:
        print("There is not available seat")
        return

    buy_seat(user, available_seat)

    return

def user_has_legal_age(user):
    user_age = user.age

    if not user.age >= 18:
        return False

    return True

def user_has_enough_balance(user, ticket):
    user_money = user.money
    ticket_price = ticket.price

    if user_momey >= ticket_price:
        return True

    return False

def get_available_seat():
    seats = stadium.seats
    for seat in seats:
        if seat.is_available():
            return seat

def buy_seat(user, seat):
    seat.owner = user
    user.money -= ticket.price
    print("Congratulations, you have a seat")
    return

```
We were able to extract four new functions from our existing function, it means we were trying to do too much.

Now, our main function `buy_concert_ticket` can be read using "natural language". Of course, I used accurate names(we're going to talk about this later on) but having a small number of lines, just one level of extra indentation, helps a lot to achieve this result.

3.- Do you think we can't make our function smaller? or reduces sections? We do, let's see it.

```python
def buy_concert_ticket(user, ticket):

    if not user_can_buy_a_ticket(user, ticket):
        return

    buy_available_seat(user)

    return

def user_can_buy_a_ticket(user, ticket):

    if not user_has_legal_age(user, ticket):
        print("You are not allowed to buy a ticket")
        return False

    if not user_has_enough_balance(user, ticket):
        print("Sorry you don't have enough balance")
        return False

    return True

def user_has_legal_age(user):
    user_age = user.age

    if not user.age >= 18:
        return False

    return True

def user_has_enough_balance(user, ticket):
    user_money = user.money
    ticket_price = ticket.price

    if user_momey >= ticket_price:
        return True

    return False

def buy_available_seat(user):
    available_seat = get_available_seat()

    if not available_seat:
        print("There is not available seat")

    buy_seat(user, available_seat)
    return

def get_available_seat():
    seats = stadium.seats
    for seat in seats:
        if seat.is_available():
            return seat


def buy_seat(user, seat):
    seat.owner = user
    user.money -= ticket.price
    print("Congratulations, you have a seat")
    return
```

We could trim our main function a bit more, now we reduced it from 21 lines to just 4. We just applied the same principle, 
try to abstract our logic and identifying sections/blocks in order to create new functions. This will become in an iterative process. 

So, what we gained?
- Anyone trying to read our function would take just seconds to understand what we were trying to do.
- You can read the whole process just like a "top/down story".
- Now, implementing unit test would be easier because we treated every step of buying ticket process as a single function.
- If we want to add extra requirements in our process, identifying the function to change will be enough, for example: adding another restriction for a buyer, we just need to change our logic within `user_can_buy_ticket()` function without touching anything else. 

Note: I know, doing this is time-consuming, but trust me, even when you can't iterate so many times due to deadlines, etc. Try to do it at least once. Always the first iteration brings good results.

That's all. In the next chapter, I'm going to talk more about how function parameters should be defined in order to improve readiness, giving other tips as well.


