Welcome to the backend of the of the Fencer sales emporium. That isn't the full
name but hey, when dealing with APIs you have to take some hyperbole when you.
Things just get a little bit to specific, if you specifically know what I mean.
If you don't, why are you looking at API's, that's a bit of a non specific
place to start.

Here are the links to our repos and deployed sites. Yes, this repo is included.

front end deployed : https://heroku-complex.github.io/team-project-front-end/ (edited)

backend deployed : https://mudabish.herokuapp.com/ (edited)

front end repo: https://github.com/Heroku-Complex/team-project-front-end

back end repo : https://github.com/Heroku-Complex/team-project-back-end


Let's start at the top.

## API

Use this as the basis for your own API documentation. Add a new third-level
heading for your custom entities, and follow the pattern provided for the
built-in user authentication documentation.

Scripts are included in [`scripts`](scripts) to test built-in actions. Add your
own scripts to test your custom API.

### Authentication

| Verb   | URI Pattern            | Controller#Action |
|--------|------------------------|-------------------|
| POST   | `/sign-up`             | `users#signup`    |
| POST   | `/sign-in`             | `users#signin`    |
| PATCH  | `/change-password/:id` | `users#changepw`  |
| DELETE | `/sign-out/:id`        | `users#signout`   |

#### POST /sign-up

Request:

```sh
curl --include --request POST http://localhost:4741/sign-up \
  --header "Content-Type: application/json" \
  --data '{
    "credentials": {
      "email": "an@example.email",
      "password": "an example password",
      "password_confirmation": "an example password"
    }
  }'
```

```sh
scripts/sign-up.sh
```

Response:

```md
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8

{
  "user": {
    "id": 1,
    "email": "an@example.email"
  }
}
```

Note that this database is dealing with hashes, so the id would be a non- specific hash instead of an expected, sql id. NoSQL databases will alwys use
to keep records.

#### POST /sign-in

Request:

```sh
curl --include --request POST http://localhost:4741/sign-in \
  --header "Content-Type: application/json" \
  --data '{
    "credentials": {
      "email": "an@example.email",
      "password": "an example password"
    }
  }'
```

```sh
scripts/sign-in.sh
```

Response:

```md
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "user": {
    "id": 1,
    "email": "an@example.email",
    "token": "33ad6372f795694b333ec5f329ebeaaa"
  }
}
```

#### PATCH /change-password/:id

Request:

```sh
curl --include --request PATCH http://localhost:4741/change-password/$ID \
  --header "Authorization: Token token=$TOKEN" \
  --header "Content-Type: application/json" \
  --data '{
    "passwords": {
      "old": "an example password",
      "new": "super sekrit"
    }
  }'
```

```sh
ID=1 TOKEN=33ad6372f795694b333ec5f329ebeaaa scripts/change-password.sh
```

Response:

```md
HTTP/1.1 204 No Content
```

#### DELETE /sign-out/:id

Request:

```sh
curl --include --request DELETE http://localhost:4741/sign-out/$ID \
  --header "Authorization: Token token=$TOKEN"
```

```sh
ID=1 TOKEN=33ad6372f795694b333ec5f329ebeaaa scripts/sign-out.sh
```

Response:

```md
HTTP/1.1 204 No Content
```

### Users

| Verb | URI Pattern | Controller#Action |
|------|-------------|-------------------|
| GET  | `/users`    | `users#index`     |
| GET  | `/users/1`  | `users#show`      |

#### GET /users

Request:

```sh
curl --include --request GET http://localhost:4741/users \
  --header "Authorization: Token token=$TOKEN"
```

```sh
TOKEN=33ad6372f795694b333ec5f329ebeaaa scripts/users.sh
```

Response:

```md
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "users": [
    {
      "id": 2,
      "email": "another@example.email"
    },
    {
      "id": 1,
      "email": "an@example.email"
    }
  ]
}
```

#### GET /users/:id

Request:

```sh
curl --include --request GET http://localhost:4741/users/$ID \
  --header "Authorization: Token token=$TOKEN"
```

```sh
ID=2 TOKEN=33ad6372f795694b333ec5f329ebeaaa scripts/user.sh
```

Response:

```md
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "user": {
    "id": 2,
    "email": "another@example.email"
  }
}
```

#### ORDERS /orders

We save all of our orders in the database with the following schema.

order {
  "date_placed": "Date",
  "salesProof": {
    {
    "id": "hashed id",
    "amount": "123124",
    "currency": "usd",
    "status": "success"
    },
  "isOpen": "true"
  "products": [["product_id", 1],["product_id", 1]],
  "_owner": "id",
  "timestamps": "timestamps"
  }
}

In this, the date is the current date of the order being placed. After that we have the optional salesProof object, which is taken from a token given to us from the Stripe API. After that we are looking ad the isOpen. isOPen is a Boolean that can have only one true at a time per user, as the open cart is the current cart in use. Closed carts are the only carts that will have the salesProof attached to it, as it is proof that the credit card has gone through.

Products is referenced from our products, and is nested within an array within an array. Each product is attached to a quantity, just incase you want more than one butter robot.

After that we have our user id, which is, of course, a reference to whomever purchased the cart.

#### Products /products

The product schema is as follows:

products {
  name: "sample name"
  category: "droids",
  price: 1231232
  description: "samples are expensive"
  img_url: "sample url"
  rating: 5
  _owner: "id"
}

This is pretty self explained.

#### Stripe

Stripe is a third party api that handles and processes all credit card information. On the front end, the user inputs the credit card information, which is then sent directly to the Stripe API to produce a token. This token is all the sensitive credit information, symbollically. There is no sensitive data being handled by us at anytime.

On the front, this token is then tacked on an amount, in cents only (for US dollars. If you are using a different currency, see Stripe docs for conversions). This token is then sent to us, processed (basically rebounded off our api) and hits the Stripe API through our own. The Client is then sent back data, essentially a copy of the token, on a successful payment. We then dissect that returned success and put it into a custom object to be stored in orders so anyone that had a dispute can see who the Stripe Id'ed person is, what the order was, how much, and solve any issue from there. The data should not be parsed by the developer, and should instead be built by the Stripe API. For questions on how to use Stripe, see the Stripe documentation.

####

Enjoy and start buying to our hearts desire....I mean your heart. You.

## [License](LICENSE)

1.  All content is licensed under a CC­BY­NC­SA 4.0 license.
1.  All software code is licensed under GNU GPLv3. For commercial use or
    alternative licensing, please contact legal@ga.co.
# fencer-back-end
