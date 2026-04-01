## Decode JWT in the terminal

Do you still use online tools like jwt.io to decode JWT tokens? Ever wonder how they are decoded?
Would you like to have a tool right in your terminal to do it?

Let's decode the JWT

A JWT token has three parts. It's in the following format:

`header.payload.signature`

All three parts are part of the token, but in this article we will only decode the header and payload.
The signature is used to verify the token with the secret. We will not be covering that this time.

In this article, we will decode a JWT token with Node.js and also write a small command line script so we can decode it from the terminal using `jwt <token>`.

Let's get started.

We will do this in steps so you can build along and test it all the way.

- We will start with a simple function `decodeJwt(token)` to decode the token
- We will add support to pass the token while running the script `node jwt <token>`
- We will add it as a global script which can be used from anywhere `jwt <token>`

### Step 1: Create the `decodeJwt` function

Create a `scripts` directory in a location of your choice.

```bash
mkdir scripts
```

Add the `jwt` file.

```bash
touch jwt
```

A JWT token will be in the `header.payload.signature` format. Let's split it and get the values.

```javascript
function decodeJwt(jwtToken) {
  const [header, payload, signature] = jwtToken.split(".");
}
```

Let's write a function to decode a base64url-encoded string.

```javascript
function base64UrlDecode(str) {
  // Replace URL safe chars
  str = str.replace(/-/g, "+").replace(/_/g, "/");

  // Add padding if needed
  while (str.length % 4) {
    str += "=";
  }

  return Buffer.from(str, "base64").toString("utf8");
}
```

Now let's decode the payload.

```javascript
function decodeJwt(jwtToken) {
  const [header, payload, signature] = jwtToken.split(".");
  const decodedPayload = JSON.parse(base64UrlDecode(payload));
}
```

Let's test the script.

Code so far:

```javascript
function base64UrlDecode(str) {
  // Replace URL safe chars
  str = str.replace(/-/g, "+").replace(/_/g, "/");

  // Add padding if needed
  while (str.length % 4) {
    str += "=";
  }

  return Buffer.from(str, "base64").toString("utf8");
}

function decodeJwt(jwtToken) {
  const [header, payload, signature] = jwtToken.split(".");
  const decodedPayload = JSON.parse(base64UrlDecode(payload));
  console.log("Payload::");
  console.log(decodedPayload);
}

decodeJwt(
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyMTIzIn0.OdG00YOF5aRPGJ2QgRh90hu9AO3Xye8bonaEtPaVodU",
);
```

Let's run this script with Node.js.

```bash
node jwt
```

```bash
Payload::
{ sub: 'user123' }
```

### Step 2: Let's make this run like a CLI utility.

Usage:

```bash
node jwt <jwtToken>
```

Remove the function call and add these to the end of the file.

```javascript
...
const jwtToken = process.argv[2];

if (!jwtToken) {
  console.error("token not found");
  console.error("Usage: jwt <token>");
  process.exit(0);
}

decodeJwt(jwtToken);
```

Now we can run this script by passing a JWT token as input.

```bash
node jwt <token>
```

### Step 3: Let's make this script run from anywhere in the terminal.

The goal is to run `jwt <token>` from any folder and have it work.

Let's do it.

Let’s add the following line to the top of our `jwt` file. This tells the operating system to run the script using Node.js. It will find `node` in the current `PATH` and use it to run the script.

```javascript
#!/usr/bin/env node
```

Now we have to make this file executable.

What is an executable? How do we check it?

Run `ls -l` in the `scripts` folder and you will get something like this:

```bash
-rw-r--r--@ 1 johndoe  staff   682  1 Apr 22:39 jwt
```

`-rw-r--r--` shows the file permissions.
We have only read and write access as a user. We have to make it executable.

To do this, run `chmod 700 jwt` and run `ls -l` again. You should have `rwx` now. You will be able to run this file.

```bash
-rwx------@ 1 johndoe  staff   682  1 Apr 22:39 jwt
```

Now you can just run `./jwt <token>` to decode.

Do I still have to be in the `scripts` folder to run it?

Yes, for now, but we can change this.
We need to add this `scripts` folder to our `PATH` variable so the terminal knows where to look for it.

How do I add this?

If you use the zsh terminal, there will be a `.zshrc` file in your home folder.

Add your folder path to `PATH`.

If your `scripts` folder is on your Desktop:

```bash
echo 'export PATH=$PATH:$HOME/Desktop/scripts' >> ~/.zshrc
```

Now source your `.zshrc` using `source ~/.zshrc`.

Run `which jwt`. It should show your file path.

```bash
which jwt
/Users/johndoe/Desktop/scripts/jwt
```

Now you can run `jwt <token>` from any folder in your terminal. Your JWT token will be decoded and displayed.

Final command:

```bash
jwt eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyMTIzIn0.OdG00YOF5aRPGJ2QgRh90hu9AO3Xye8bonaEtPaVodU
Payload::
{ sub: 'user123' }
```

You can find the code and article details in this repo [jwt repo](https://github.com/gousejani/scripts/tree/main/jwt-dir)
